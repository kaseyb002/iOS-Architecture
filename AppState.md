# App State

Keeping the app state up-to-date is one of the hardest problems for me. I am not very satisfied with my current solution.

My techniques for updating state fall into two categories: (1) state updates where only **one** person needs to be notified, and (2) state updates where **many** people may need to be notified.

For the first case, notifying about state updates through simple callback closures seems to get the job done. And usually, if only one person needs to be notified, then the notifier and notifiee tend to be close to each other anyway.

For the second case, I'm using a little library called [Signal](https://github.com/artman/Signals). It's basically a type-safe `NotificationCenter`. It's a great little library, but I don't like the general idea of sending out a notification to keep the rest of your app in-sync.