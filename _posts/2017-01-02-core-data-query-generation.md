---
layout: post
title:  "NSQueryGenerationToken"
date:   2017-01-02 12:04:38 +0200
categories: ios coredata  
tags: [coredata, core, data, swift, swift3, ios, ios10, database, sqlite3, sqlite, NSPersistentContainer, NSQueryGenerationToken, xcode]
description:  This article describes new Core Data API and also demystifies what actually happens with sqlite3 database while using query generation.
---

iOS 10 does not only brings `NSPersistentContainer` to Core Data. But also very useful `NSQueryGenerationToken`.
New API allows to fetch different snapshots of data in time. Developer can pin `NSManagedObjectContext` to specific point in time and fetch unchanged data until context become unpinned or pined to different point.

By default `NSManagedObjectContext` is unpinned. To pin context you need to set context generation token.

`context.setQueryGenerationFrom(NSQueryGenerationToken.current)`

Context will pin itself to the generation of the database when it **first** loads data.

After this all subsequent reads from the store using this context will return old data regardless any changes made in other contexts.
This works only for **fetched data** so when you decide to save context it will become **unpinned** again.
Also you can unpin context by setting generation token to `nil`. It is important to note that context does not refresh objects automatically.

With Query Generations you can make consistent database snapshot. This help when you data is frequently or concurrently updated. Query Generations save from fault exceptions like "Core Data could not fulfill a fault".

## Query Generations and SQLite 3 Snapshots

Query Generation is available thanks to sqlite 3 WAL journaling mode and snapshots.

With WAL("Write-Ahead Log") each change to the SQLite Database is written to a separate -wal file. And at at some point in time all changes are transferred into the original database. This helps SQLite work fast and concurrent.

SQLite snapshots records the state of a WAL mode database for some specific point in history.

Before first read Core Data gets snapshot of current state with `sqlite3_snapshot_get` and on each fetch it adopts this snapshot to get historical version of data using `sqlite3_snapshot_open`.

![Core Data Opens Snapshot](/assets/posts/2017-01-02-core-data-query-generation/shapshot_open.png){: .img-responsive }

It was unexpected for me to found that snapshotting is actually **experimental** [SQLite 3 API](https://www.sqlite.org/c3ref/snapshot.html).
