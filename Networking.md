# Networking

My networking classes are roughly grouped together by object types (e.g., `AccountService`, `EpisodesService`). Here's an example of a network call inside an `API` class.

```swift
final class EpisodesAPI {

  class func searchEpisodes(with text: String, callback: @escaping (Result<[Episode], NetworkError>) -> ()) {
    EpisodesService.searchEpisodes(with: text) { [weak self] result in
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

### Other Notes
I leave my `[String:Any]`/`Parameters` free form. I find that tends to be easier than trying to create type-safe parameters for each endpoint.

I'm using Alamofire. I should probably just switch over to Apple's networking libaries. Maybe one day.






