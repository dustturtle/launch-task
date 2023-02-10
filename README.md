# LaunchTask

Language: English | [中文](README-ZH.md)

This is a lib for managing App launch tasks.
A large App often needs to perform a large number of tasks during the launch phase. If there are many app developers, it is easy to cause the code of AppDelegate to be modified frequently and become more and more bloated. It is also easy to cause the launch time of the App to be longer, and it will eventually be killed by Watchdog.
This lib encapsulates the launch code into subclasses of each Task, and configures the execution order and dependencies of Tasks through Workflow. At the same time, by blocking the didFinishLaunch method, tasks are executed concurrently to speed up the launch time.

## Example

Assume that when the App launchs, the initialization steps in the figure below need to be performed. Then it can be managed through two Workflows.
<img src="https://github.com/jayden320/launch-task/blob/master/example.jpg">
didFinishLaunch phase
```
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    var launchWorkflow = TaskWorkflow(name: "Launch")
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        launchWorkflow.setBlockingTasks([
            TaskA(),
            TaskB(),
            TaskC(sons: [TaskD()]),
            TaskE(queue: .concurrentQueue),
            TaskF(queue: .concurrentQueue),
        ])
        launchWorkflow.addTask(TaskG())
        launchWorkflow.addTask(TaskH())
        launchWorkflow.start()
        return true
    }
}

```
viewDidAppear phase
```
class ViewController: UIViewController {
    var firstScreenWorkflow = TaskWorkflow(name: "FirstScreen")
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .lightGray
        firstScreenWorkflow.addTask(TaskI())
        firstScreenWorkflow.addTask(TaskJ(queue: .concurrentQueue))
        firstScreenWorkflow.addTask(TaskK())
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        if firstScreenWorkflow.state == .pending {
            firstScreenWorkflow.start()
        }
    }
}

```
The above example is to execute the start method of workflow directly in viewDidAppear. If you want to perform a specific task after the first frame is rendered, you can also execute the start method in the CFRunLoopActivity.beforeTimers callback.

## Install
```
pod 'LaunchTask'
```

## Author

Jayden Liu

## License

LaunchTask is available under the MIT license. See the LICENSE file for more info.
