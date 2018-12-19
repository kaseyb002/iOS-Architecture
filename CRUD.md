# Create, Read, Update, and Deleting Data
I create a separate `API` class for all data transformations. I try to model these `API` classes as if you were calling a backend API. I roughly group them by object types (e.g.,`EpisodesAPI`, `AccountAPI`).

`ViewControllers` use the `API` classes a one-stop-shop for all CRUD operations. For example, whether an object should be saved locally or to iCloud, read from the backend or the cache, the `API` will take care of all those details.

Let's look inside the that `saveEpisode()`.

```swift
final class EpisodesAPI {

  class func saveEpisode(id: EpisodeId, atPosition position: TimeInterval) {
    Cache.shared.saveEpisode(id: EpisodeId, atPosition: position)
    EpisodesService.saveEpisode(id: EpisodeId, atPosition: position)
  }
}
```

## Passive models, active `API`s

My data model objects are always passive. They don't do any CRUD operations. So for example, my `Episode` struct will never have a method like `savePosition()`. Instead, I would use `EpisodesAPI.saveEpisode(id: EpisodeId, atPosition: TimeInterval)` do that.

```swift
EpisodesAPI.saveEpisode(id: EpisodeId, atPosition: TimeInterval)
```

## Models
Here's an example of what my core model structs look like.

```swift
struct Episode {
  let id: EpisodeId
  let title: String
  let summary: String
  let downloadLink: URL
  let releasedDate: Date
  let showId: ShowId
}
```

I use extensions on my core model objects for stuff like convenience strings:

```swift
extension Episode {

  var releasedString: String {
    if releasedDate.yearInt == Date().yearInt {
      return released.string("MMM d")
    }
    return releasedDate.string("MMM d, YYYY")
  }
}
