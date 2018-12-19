# Networking

My networking classes are roughly grouped together by object types (e.g., `AccountService`, `EpisodesService`) in a `Service` class. Here's an example of an `API` class making a `Service` network call.

```swift
final class EpisodesAPI {

  class func searchEpisodes(with text: String, callback: @escaping (Result<[Episode], NetworkError>) -> ()) {
    EpisodesService.searchEpisodes(with: text, callback: callback)
  }
}
```

And here's how that looks back up in the `ViewController`:

```swift
final class SearchVC: UIViewController {

  func searchEpisodes(with text: String) {
    EpisodesAPI.searchEpisodes(with: text) { [weak self] result in
      switch result {
      case .success(let episodes):
        self?.load(episodeResults: episodes)
      case .error:
        self?.status = .error
      }
    }	
  }
}

```

## Service Class
The `Service` classes *route* the network calls. Here's an example:

```swift
final class EpisodesService {

  private static let podcastServer = Server<PodcastEndpoint>()

  class func searchEpisodes(with searchTerm: String, callback: @escaping (Result<[Episode], NetworkError>) -> ()) {
    let params: Parameters = ["term": searchTerm]
    podcastServer.call(.searchEpisodes, parameters: params, callback: callback)
  }
}
```

You'll see that `podcastServer` is the dude that actually makes the network call. Let's dig into it.

## Server Class
The `Server` class takes a generic `Endpoint` and makes the call. The `Server` class doesn't do much; it just provides the `call()` method. The `Endpoint` is doing most of the work. Really, the `Server` class is there just for aesthetic reasons. It feels natural to write `myServer.call(endpoint)` as opposed to `endpoint.call()`.

#### The Entire Server Class
```swift
final class Server<E: Endpoint> {
    
    func call<T: Decodable>(_ endpoint: E, parameters: Parameters? = nil, callback: @escaping (Result<T, NetworkError>) -> ()) {
        Alamofire.request(
            endpoint.url,
            method: endpoint.method,
            parameters: parameters,
            encoding: endpoint.encoding,
            headers: endpoint.headers
            ).responseData { response in
                
                response.printEverything()
                
                switch response.result {
                case .success(let data):
                    do {
                        if let successCallback = callback as? (Result<NoReply, NetworkResult>) -> () {
                            successCallback(.success(NoReply()))
                        } else {
                            let instance = try JSONDecoder().decode(T.self, from: data)
                            callback(.success(instance))
                        }
                    } catch let error {
                        print(error)
                        callback(.error(.jsonDecodingFailed))
                    }
                case .failure(let error):
                    print(error)
                    guard let afError = error as? AFError else {
                        callback(.error(.failed(error.localizedDescription)))
                        return
                    }
                    callback(.error(.failed(afError.localizedDescription)))
                }
        }
    }
}
```

## Endpoint
`Endpoint` types provide all the data (minus parameters) for servers to make their call. Let's look at the protocol:

```swift
protocol Endpoint {
    var baseUrl: String { get }
    var path: String { get }
    var url: String { get }
    var method: HTTPMethod { get }
    var headers: HTTPHeaders { get }
}
```

Just create an object (enum, struct, or class) that conforms to this and you're ready to go. I typically group my `Endpoint`s by object type inside an enum class. Here's an example:

```swift
enum GroupsEndpoint {
    case myGroups
    case createGroup
    case leaveGroup(id: GroupId)
}

extension GroupsEndpoint: Endpoint {
    
    var path: String {
        switch self {
        case .createGroup:
            return "/groups/create"
        case .myGroups:
            return "/groups"
        case .leave(let id):
            return "/groups/\(id)/leave"
        }
    }
    
    var headers: HTTPHeaders {
        return AccountAPI.authHeader ?? HTTPHeaders()
    }
    
    var method: HTTPMethod {
        switch self {
        case .myGroups:
            return .get
        case .createGroup:
            return .post
        case .leave:
            return .post
        }
    }
    
    var baseUrl: String { return Constants.podcastServerUrl }
}
```


### Other Notes
I leave my `[String:Any]`/`Parameters` free form. I find that tends to be easier than trying to create type-safe parameters for each endpoint.

I'm using Alamofire. I should probably just switch over to Apple's networking libaries. Maybe one day.






