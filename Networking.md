# Networking

My networking classes are roughly grouped together by object types (e.g., `AccountService`, `EpisodesService`). Here's an example of a network call inside a `ViewController`

```swift
class SearchVC: UIViewController {

  func searchEpisodes(with text: String) {
    EpisodesService.searchEpisodes(with: text) { [weak self] result in
      switch result {
      case .success(let results):
        self?.load(episodeResults: results)
      case .error:
        self?.status = .error
      }
    }		
  }
}
```

## The Service Classes
The `Service` classes make the network calls. Here's an example:

```swift
final class EpisodesService {
  private static let listenNotesService = Service<ListenNotesEndpoint>()

  class func searchEpisodes(with searchTerm: String, callback: @escaping (Result<EpisodeResults, NetworkError>) -> ()) {
    let params: Parameters = ["term": searchTerm]
    listenNotesService.call(.search, parameters: params, callback: callback)
  }
}
```