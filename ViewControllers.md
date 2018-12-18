# View Controllers

## Solving Massive View Controllers
I solve the Massive View Controller problem by breaking my view controller down into smaller pieces with **child view controllers**. 

If you are not aggressively breaking down your view controllers into child view controllers, I strongly suggest you try doing so. This has been the most helpful architectural technique I've ever come across in iOS development.

A lot of people praise Flutter and React-Native for their "widget"-based approach to building layouts. In those platforms you create a little widget called `A`, then you can create another widget `B` composed of `A` widgets, and then create a widget `C` composed of `B` widgets and maybe other widgets, and so on. iOS has the exact same functionality with child view controllers.

### Example
I will walk through the `Now Playing` screen of a podcast app I'm working on.


### State Containers
There is one particularly helpful application of child view controllers that deserves special mention. That is handling empty/loading/error/found states. By creating a child view controller for each state (`LoadingVC`, `ErrorVC`, `EmptyVC`, found - `ListTableViewController`), you can just swap each one out as needed.


### Code
Here's the function I use to embed my child view controllers:

```swift
extension UIViewController {
    
    func embed(_ childVC: UIViewController,
               into container: UIView,
               fadeInDuration: TimeInterval? = nil) {
        // clean out container
        container.subviews.forEach { $0.removeFromSuperview() }
        
        // remove child VC's association with any other parent
        childVC.view.removeFromSuperview()
        childVC.removeFromParent()
        
        // add in the child vc
        addChild(childVC)
        
        childVC.view.frame = container.bounds
        childVC.view.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        childVC.view.translatesAutoresizingMaskIntoConstraints = true
        container.addSubview(childVC.view)
        
        childVC.didMove(toParent: self)
        
        // IB has default white background
        container.backgroundColor = .clear 
        
        // handy fade animation, if desired
        if let fadeInDuration = fadeInDuration {
            container.alpha = 0.0
            UIView.animate(withDuration: fadeInDuration) { container.alpha = 1.0 }
        }
    }
}
```

##### Using the `embed` func

```swift
class FullScreenExampleVC: UIViewController {

	@IBOutlet weak private var childView: UIView!

	private lazy var childVC = ChildExampleVC(data: "hello")

	override func viewDidLoad() {
	    super.viewDidLoad()
	    embed(childVC, into: childView)
	}
```

#### MVVM?
I do not like MVVM. MVVM does not solve the Massive View Controller problem. It just breaks up the view controller code into two different files: one for view logic and one for business logic. At the end of the day, the view controller itself is still massive.

Occasionally I will use a ViewModel for a child view controller (or table view cell) that's being widely re-used. That way I can use several types of business/presentation logic for the same view. But it's rare.



