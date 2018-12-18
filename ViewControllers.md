# View Controllers

## Solving Massive View Controllers
I solve the problem of Massive View Controllers by breaking my view controller down into smaller pieces with *child view controllers*. 

If you are not aggressively breaking down your view controllers into child view controllers, I strongly suggest you try doing so. This has been the most helpful architectural technique I've ever come across in iOS development.

A lot of people praise Flutter and React-Native for their "widget"-based approach to building layouts. In those platforms you create a little widget called `A`, then you can create another widget `B` composed of `A` widgets, and then create a widget `C` composed of `B` widgets and mayben other widgets, and so on and so on. iOS has the exact same functionality through child view controllers.


#### MVVM?
I do not like MVVM. MVVM does not solve the Massive View Controller problem. It just breaks up the view controller code into two different files: one for view logic and one for business logic. At the end of the day, your view controller itself is still massive.

Occasionally I will use a ViewModel for a child view controller that's widely re-used. That way I can use several types of business/presentation logic for the same view. But again, that rarely happens.



