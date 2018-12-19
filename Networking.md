# Networking

My networking classes are grouped by an object type (e.g., `AccountService`, `EpisodesService`). Here's an example of a network call inside a `ViewController`

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

## Service Classes
The `Service` class is where you go to make your network call for certain categories. 