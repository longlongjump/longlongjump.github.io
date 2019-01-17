---
layout: post
title:  "What’s  new in Alamofire 5"
date:   2019-01-07 18:00:01 +0200
categories: ios
tags: [Alamofire, swift, Networking, UrlSession, ios, swipe, Xcode]
---

Alamofire 5.0 beta was released on December but there are still not many examples of new features.

So I decided to investigate all updates myself and check how they align with my existing code.


## ParameterEncoder with support of Encodable params

Now with support of `ParameterEncoder` you can start using `Encodable` structures and objects instead of passing tons of parameters or using `[String: Any]` dictionary.

```
struct UserParams: Encodable {
    var name: String?
    var middleName: String?
    var lastName: String?
    var age: Int?
}

struct SearchParams: PagedParams {
    var keyword: String?
    var offset = 0
    var limit = 30
}



enum Router: URLRequestConvertible {
    case createUser(parameters: UserParams)
    case searchUsers(parameters: SearchParams)


    static let baseURLString = "https://example.com"

    var method: HTTPMethod {
        switch self {
        case .createUser:
            return .post
        case .searchUsers:
            return .get
        }
    }

    var path: String {
        switch self {
        case .createUser, .searchUsers:
            return "/users"
        }
    }

    func asURLRequest() throws -> URLRequest {
        let url = try Router.baseURLString.asURL()

        var urlRequest = URLRequest(url: url.appendingPathComponent(path))
        urlRequest.httpMethod = method.rawValue

        switch self {
        case .createUser(let parameters):
            urlRequest = try URLEncodedFormParameterEncoder.default.encode(parameters, into: urlRequest)
        case .searchUsers(let parameters):
            urlRequest = try URLEncodedFormParameterEncoder.default.encode(parameters, into: urlRequest)
        }

        return urlRequest
    }
}
```


```
var user = UserParams()
user.name = "eugene"

Alamofire.request(Router.createUser(parameters: user))

// no more [String: Any] dictionaries
Alamofire.request(Router.searchUsers(parameters: SearchParams(keyword: "eugene", offset: 0, limit: 30))
```

## Decodable responses

Not much coding needed to actually implement it yourself but It is nice this is finally part of Alamofire.

```
final class User: Decodable {
  var name: String?
}

let req = Alamofire.request(Router.searchUsers(parameters: SearchParams(keyword: "eugene", offset: 0, limit: 30))
req.responseDecodable { (res: DataResponse<[User]>) in
    if let error = res.error {
        debugPrint(error.localizedDescription)
    } else if let users = res.value {
        debugPrint(users.first?.name)
    }
}
```



## AlamofireNotifications and EventMonitor

In case you have custom logger of network activity now it bit easier to log server responses.


```
NotificationCenter.default.addObserver(
            self,
            selector: #selector(NetworkActivityLogger.networkRequestDidComplete(notification:)),
            name: Alamofire.Request.didComplete,
            object: nil
        )
```

```
@objc private func networkRequestDidComplete(notification: Notification) {
       guard
              let request = notification.request,
              let urlRequest = request.request
              let httpMethod = urlRequest.httpMethod,
              let requestURL = urlRequest.url
           else {
               return
       }

      print("\(String(response.statusCode)) '\(requestURL.absoluteString)'")

       if let data = (request as? DataRequest)?.data  {
         logData(data: data)
      }       
}
```

You can also adopt `EventMonitor` protocol and and implement your custom `Session` monitoring yourself.

```
final class SessionMonitor: EventMonitor {
    func request(_ request: Alamofire.Request, didCompleteTask task: URLSessionTask, with error: Error?) {
        debugPrint(request.request?.url?.absoluteString ?? "")
    }
}

let session = Session(eventMonitors: [SessionMonitor()])
```


## RequestAdapter and RequestRetrier

`RequestAdapter` became more flexible. You can now update request asynchronously or even discard request if something is wrong.

`RequestRetrier` is also useful it can retry failed request if server is not available at the moment or API access token needs to be refreshed. Asynchronously as well.
