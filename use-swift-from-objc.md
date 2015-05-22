# Using Swift from Objective-C

There are few basic steps to use a **Swift** class from **Objective-C**. Most of the documentation around skip or misconvey the actual usage. The first thing to know is that, the first time you add any **Swift** or **Objective-C** file to your project, Xcode will prompt you to add a *bridge-header.h* file. This is completely irrelevant to use **Swift** from **Objective-C** because an invisible file named `YOUR_PROJECT_NAME-Swift.h` is automatically created for you when you build the project.

The last sentence is a very important point, in order to get access to this invisible file you have to add the `@objc` tag to the classes, methods, properties, etc. that you want to expose to **Objective-C** and then run the build command (i.e. ⌘ + b).
A class supposed to be available in **Objective-C** will look like this:

```
@objc class Person {
    
    init() {
        // Initialization
    }

}
```
After you run the build you can import the invisible file header into the `.m` files and use then the class, methods and properties as usual in *Objective-C*

```
#import "YOUR_PROJECT_NAME-Swift.h"
...

Person *person = [[Person alloc] init];

```
The ```YOUR_PROJECT_NAME-Swift.h``` file must be imported in all the files you are planning to expose the classes, methods, properties, etc. defined in **Swift** and tagged with the tag `@obj`.

