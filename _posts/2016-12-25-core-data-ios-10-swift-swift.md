---
layout: post
title:  "Core Data in iOS 10"
date:   2016-12-25 23:04:38 +0200
categories: coredata ios swift swift3
tags: [coredata, core, data, swift, swift3, ios, ios10, database, NSPersistentContainer, NSManagedContext, stack, xcode]
---

CoreData always was first option for storing and fetching data in iOS and macOS. It has many cool features that comes for free like visual model editor, auto-migrations, model versioning etc. Using Core Data contexts developer can fetch and save records in multithreaded environment. Core Data can efficiently load record only when developer actually need it to reduce memory consumption.

But with all these features comes complexity and a lot of boilerplate code. Which easily can lead to bugs and crashes. Many libraries were created to fix these issues. One is most popular is Magical Record. It inspired by Ruby Active Record. And greatly helped me on many projects.

It implements _root saving context_ design pattern

`Background root saving context <-> Default Main Context <-> Child Worker`

![image-title-here](/assets/posts/2016-12-25-core-data-ios-10-swift-swift/core-data-stack.jpg){: .img-responsive }

It is good when saving operations run on background thread. However this setup is not always produce great performance. Because of parent-child context chain main UI thread can be locked during heavy writes in background context.

# NSPersistentContainer

Since iOS 10 Apple decided to finally remove boilerplate and incapsulate CoreData setup into its own class.

Now there is no need of hundreds of lines of code to setup `NSPersistentStoreCoordinator`, `NSManagedObjectModel` and `NSManagedContext`.
With just one line you CoreData setup is ready.  

`let persistentContainer = NSPersistentContainer(name: "YourModelNameHere")`

then you just load underlaying store with

```
persistentContainer.loadPersistentStores(completionHandler: { (storeDescription, error) in
    //handle errors here
})
```

In case you want to directly access store you can access `persistentStoreCoordinator` property.

After initialization developer should use main `viewContext` to fetch and show data in UI thread.

To make background fetch or import data from server you either create `newBackgroundContext()` or just use `performBackgroundTask` method of `NSPersistentContainer`.

```
persistentContainer.performBackgroundTask { backgroundContext in
      let newUser = User(context: backgroundContext)
      newUser.name = "Bob"
      do {
        try backgroundContext.save()
      } catch {
        // handle error
      }
}
```

To sync changes back to main `viewContext` you should turn on `automaticallyMergesChangesFromParent`.

`persistentContainer.viewContext.automaticallyMergesChangesFromParent = true`

After you turn on context starts to observe any save notification both from its parentContext and persistentStoreCoordinator. Because `NSPersistentContainer` sets same persistentStoreCoordinator for each context new changes will be forwarded to `viewContext`.

Be aware that default merge policy type for `viewContext` is `NSMergePolicy.errorMergePolicyType`.
To automatically handle merge conflicts while saving viewContext I suggest to set merge policy to `mergeByPropertyStoreTrump`. It will prioritize store changes over in-memory while saving.
```
viewContext.mergePolicy = NSMergePolicy.mergeByPropertyStoreTrump
```

For background import context you may want to choose `NSMergePolicy.mergeByPropertyObjectTrump`.
