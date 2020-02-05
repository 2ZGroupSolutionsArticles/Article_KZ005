# Article_KZ005
Review of using libraries, frameworks and swift packages in iOS Swift-based application.


# Libraries, frameworks, swift packages… What’s the difference?

 
Every developer knows how important to support clean architecture in the project. Reasons for preference modular codebase to monolith app obvious. They are namespacing, access controls, using different programming languages in different modules. And, of course, code reuse.

In this article, we will talk about options to support modular codebase in iOS Swift-based project with:
-   libraries;
-   frameworks, including new XCFramework bundle type;
-   Swift packages.

Although the review is limited to iOS only, this doesn't mean that you can't use these approaches for other Apple's platforms.

## Basics

The concept of code organization and access control in Swift based on [Modules](https://swift.org/package-manager/). The module represented as a single unit of code distribution. Frameworks, libraries, swift packages and build targets treated in Xcode as a separate module. Each with its namespace and access controls. Module, usually, solves a particular problem. It can be reused in different situations.

[**Bundle**](https://developer.apple.com/documentation/foundation/bundle) is a file directory with subdirectories inside. On iOS, bundles serve to conveniently ship related files together in one package – for instance, images, nibs, or compiled code. The system treats it as one file and you can access bundle resources without knowing its internal structure. Dylib that cannot be linked, only opened in runtime with dlopen()

**Source file** is single Swift source code file within a module.

**Executable** - main binary for application.

[**Object file**](https://en.wikipedia.org/wiki/Object_file#targetText=An%20object%20file%20is%20a,work%20like%20a%20shared%20library.) - An object file is a file containing [object code](https://en.wikipedia.org/wiki/Object_code), meaning relocatable format [machine code](https://en.wikipedia.org/wiki/Machine_code) that is usually not directly executable. An object file may also work like a [shared library](https://en.wikipedia.org/wiki/Shared_library).

[**Object code**](https://en.wikipedia.org/wiki/Object_code) - In computing, object code or object module is the product of a compiler. In a general sense object code is a sequence of statements or instructions in a computer language, usually a machine code language (i.e., binary or an intermediate language such as register transfer language).

## Libraries

In computer science, [library](https://en.wikipedia.org/wiki/Library_(computing)) means collection of resources and code, compiled into one or more architecture. Working with iOS application you’ll faced static and dynamic libraries. Let’s take a look on what they are.

[**Static library**](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html) - (*.a) ([static archive library, static linked shared library](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html#//apple_ref/doc/uid/TP40001873-SW1)) - collection or archive of object files. Static linker collects app compiled source code with library code into a single executable file, which loaded into memory in its entirety at runtime.

![](https://github.com/SezorusArticles/Article_KZ005/blob/master/Assets/image_1.png)

Linking static library and memory usage

Since static library, in its general sense, is a sequence of statements or instructions in a machine code language, this is adding some limitations to create and distribute them:

- You’ll need to build a library for the same processor architecture as the client code. If you, for example, working on a library for the iOS application you will need to create a library for iOS Simulator and iOS devices.

- The library can’t include resource files: images, assets, nibs, strings file and other visual data. If you will include this resources to project them will be separate from .a file. Usually, as a solution to this problem, all relates external resources provided within another independent bundle.

You can create a Swift static library, this is supported from Xcode 9.0. Till Xcode 9 dynamic frameworks where required. Developers, who were using CocoaPods remember that it was required to add [use_frameworks](http://blog.cocoapods.org/CocoaPods-1.5.0/). It tells CocoaPods that you want to use Frameworks instead of Static Libraries since it wasn't supported for Swift. But fortunately Swift and Xcode are constantly improving and now we have support for Swift static libraries.

**Dynamic libraries** (*.dylib) ([dynamic shared library, shared object, dynamically linked library](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html#//apple_ref/doc/uid/TP40001873-SW1)) are not copied into executable file, like static libraries. Instead, they are dynamically linked to at load or runtime, when both binary files and the library are in memory. Dynamic libraries stored and versioned separately. As a result, the dynamic library may be loaded not the same which was originally referenced if the update is considered binary compatible with the original version.

![](https://github.com/SezorusArticles/Article_KZ005/blob/master/Assets/image_2.png)

Linking dynamic library and memory usage

System iOS and macOS libraries are dynamic. This means that your app will receive improvements from Apple's updates without new build submission. This also may lead to issues with interoperability. That’s why it is always a good idea to test the app on the new OS version before it becomes released.

There are some old discussions related to the creation and integration of own custom .dylib for iOS app: [topic1](https://stackoverflow.com/questions/48209855/xcode-9-no-option-to-create-dylib-project-ios), [topic2](https://stackoverflow.com/questions/4733847/can-you-build-dynamic-libraries-for-ios-and-load-them-at-runtime). In spite of this Apples documents [clearly says](https://developer.apple.com/library/archive/technotes/tn2435/_index.html#//apple_ref/doc/uid/DTS40017543-CH1-PROJ_CONFIG-APPS_WITH_DEPENDENCIES_BETWEEN_FRAMEWORKS):

> Dynamic libraries outside of a framework bundle, which typically have
> the file extension .dylib, are not supported on iOS, watchOS, or tvOS,
> except for the system Swift libraries provided by Xcode.

## Frameworks

> [A framework](https://developer.apple.com/library/archive/technotes/tn2435/_index.html#//apple_ref/doc/uid/DTS40017543-CH1-PROJ_CONFIG-APPS_WITH_DEPENDENCIES_BETWEEN_FRAMEWORKS) ( .framework): is a hierarchical directory that encapsulates a dynamic library, header files, and resources, such as storyboards, image files, and localized strings, into a single package. Apps using frameworks need to embed the framework in the app's bundle.

Frameworks intended for the same purposes as a [static and dynamic shared libraries](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html). But unlike libraries, frameworks:

-   can include resources like images, assets, documentations, strings files.
-   only one copy of framework read-only resource loaded to memory, that allows to decrease memory footprint and share framework between iOS app and extensions.
    

There is also another point of view on the difference between frameworks and libraries, which based on the architect and clean design perspective. In the great article “[Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html)” [Martin Fowler](https://martinfowler.com/) says that inversion of control is a key part of what makes a framework different from a library:

-   The library is essentially a set of functions that you can call, these days usually organized into classes. Each call does some work and returns control to the client.
    
-   A Framework embodies some abstract design, with more behavior built in. In order to use it you need to insert your behavior into various places in the framework either by subclassing or by plugging in your own classes. The framework's code then calls your code at these points. The main control of the program is inverted, moved away from you to the framework.
    

Frameworks supported from iOS 8.

Talking about frameworks also worth mentioning umbrella frameworks and universal frameworks (fat frameworks):

**umbrella framework:** is a framework bundle that contains other frameworks. Umbrella frameworks available for [macOS apps](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/FrameworkAnatomy.html#//apple_ref/doc/uid/20002253-97623-BAJJHAJC), but Apple [do not recommend](https://developer.apple.com/library/archive/technotes/tn2435/_index.html#//apple_ref/doc/uid/DTS40017543-CH1-PROJ_CONFIG-APPS_WITH_DEPENDENCIES_BETWEEN_FRAMEWORKS) using them. Umbrella frameworks [are not supported on iOS, watchOS, or tvOS](https://developer.apple.com/library/archive/technotes/tn2435/_index.html).

**universal framework (fat framework)**: multi-architecture binary that contains code native to multiple instructions sets and can run on multiple processor types. In short, it contains code compiled for all the platforms which you are going to support. For example, x86_64 (64-bit Simulator), arm64 arm64e armv7 armv7s for devices. As a result, such a framework will have a larger size than a one-architecture framework. There are lots of tutorials on how to make universal framework using [lipo](https://ss64.com/osx/lipo.html). This approach widely used for sharing a private binary. In this way, your framework consumer able work with your framework on a real device and simulator.

You can expect framework with `file` command in Terminal:

```
 file <PathToAppFramework>/<FrameworkName>.framework/<FrameworkName>
```
![](https://github.com/SezorusArticles/Article_KZ005/blob/master/Assets/image_3.png)

Example of universal framework

To inspect all the dynamically linked frameworks and libraries to a binary you can use [otool](https://www.manpagez.com/man/1/otool/):

    otool -L <PathToArchive>/Products/Applications/<AppName>.app/<AppBinary>

![](https://github.com/SezorusArticles/Article_KZ005/blob/master/Assets/image_4.png)

The output of `otool` Terminal command lists all of the dynamic frameworks and libraries that linked to binary

Apple says that the app on average contains 100 - 400 system dylibs. The loading of system frameworks is highly optimized. But loading custom embedded frameworks can be expensive. Apple’s engineers encourage you to use frameworks wisely and limit the amount of used framework because it impacts on the app launch time. If you are interested in deep details on how the framework works and how it impacts on the app launch time, check the WWDC session [Optimizing App Startup Time](https://developer.apple.com/videos/play/wwdc2016/406/).

This year Apple declare that Swift 5 provides binary compatibility for apps, that means that app built with one version of the Swift compiler will be able to talk to a library built with another version. Swift’s ABI is currently declared [stable for Swift 5](https://swift.org/blog/abi-stability-and-more/) on Apple platforms.

## XCFrameworks

XCFrameworks is a new supported way to distribute binary frameworks, available from Xcode 11. Actually a framework that now can containing code for multiple architectures and platforms. You will still be required to generate archives for different platforms and bundle them up together in single XCFrameworks. There is a great session from WWDC 2019: [Binary Frameworks in Swift](https://developer.apple.com/videos/play/wwdc2019/416/) that explain in detail how to create, integrate and distribute XCFrameworks.

There are few advantages of using XCFrameworks:
- XCFramework contain variants not only for device and Simulator, but for any of the platforms that Xcode supports: iOS, macOS, tvOS, watchOS;
- It supports Swift and C-based code;
- Can bundle up frameworks and static libraries.
    
## Swift Package

Swift is a cross-platform language and requires a cross-platform tool for building the Swift code. One of the goals [Swift Package Manager](https://swift.org/package-manager/#targetText=The%20Swift%20Package%20Manager%20is,in%20Swift%203.0%20and%20above.) (SwiftPM) is simplified distribute source code in the Swift ecosystem. SwiftPM is an open-source project, you can find information about it on [GitHub](https://github.com/apple/swift-package-manager) as well.

Swift Package contains source files and manifest file (Package.swift). Manifest describes the configuration for the Swift package. Swift Package is defined and used with [Swift Package Manager](https://swift.org/package-manager/), that included  [Swift 3.0 and above](https://swift.org/package-manager/). Because distribution in Source form, there is no need anymore to maintain binary compatibility for clients. So if you can ship the source code, then Swift Package is a great tool for you. With new changes in Xcode 11, you can easily create and distribute Swift Packages.

Swift package consists of 3 parts: dependencies, targets and products:

**Dependencies**: other swift packages, that you are using inside your package. In Package file each dependency specified by source location and version.

**Target**: As Apple’s document says, [Targets are the basic building blocks of a package](https://developer.apple.com/documentation/xcode/creating_a_swift_package_with_xcode). Is may be either a library or an executable as its product. Swift package can contain Swift, Objective-C/C++, or C/C++ code that should be separated in individual target, you can’t have Swift with Objective-C in one target.

**Products**: the output of your package, either a library or an executable produced by a package, and make them visible to other packages.

Below Pros and Cons of Swift Packages (at least at the time of writing this article).

Pros:
1.  Dependencies managed by Xcode;
2.  Versions managed by Xcode;
3.  No binary compatibility requires, package can be compiled for multiple platforms in the same build operation. There is no need anymore to create separate framework target for each platform;
4.  Distribution in Source form allows inspect the code and step into it while debugging.
   
Cons:
1.  Swift packages contain source code but don’t support assets and resources.
2.  Swift package can only depend on other Swift packages, binary dependencies aren’t supported.
3.  Distribution in Source form will not fit for framework providers, who don’t want to share source code.
    

---

   When you are making architectural choice between static libraries, frameworks or Swift packages for your iOS application, obviously, you should take into account the limitations of each specific project.

Linking too many static libraries into an app produces large app executable files, slow launch times and large memory footprints. Frameworks gives you much more flexibility than static libraries, they can contain resources. But each embedded framework added to project increases startup time as well.

If you can ship your source files, Swift packages may be the right choice for you. Xcode will take care of all the dependencies, versioning, platforms.


  #### Additional Sources

**Static libraries and frameworks:**
 
[Overview of Dynamic Libraries](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html#//apple_ref/doc/uid/TP40001873-SW1)

[Embedding Frameworks In An App](https://developer.apple.com/library/archive/technotes/tn2435/_index.html#//apple_ref/doc/uid/DTS40017543-CH1-PROJ_CONFIG-APPS_WITH_DEPENDENCIES_BETWEEN_FRAMEWORKS)

[What are Frameworks?](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html#//apple_ref/doc/uid/20002303-BBCEIJFI)

**XCFramework:**

[Binary Frameworks in Swift](https://developer.apple.com/videos/play/wwdc2019/416/)

**Swift package:**


[Swift Package](https://developer.apple.com/documentation/swift_packages/package)

[Getting to Know Swift Package Manager](https://developer.apple.com/videos/play/wwdc2018/411/)

[Package Manager](https://swift.org/package-manager/)

[Adopting Swift Packages in Xcode](https://developer.apple.com/videos/play/wwdc2019/408/)

[Creating Swift Packages](https://developer.apple.com/videos/play/wwdc2019/410/)

[Creating a Swift Package with Xcode](https://developer.apple.com/documentation/xcode/creating_a_swift_package_with_xcode)



Thank you for your time.



#### Author

Kseniia Zozulia

Email:  [kseniiazozulia@sezorus.com](mailto:kseniiazozulia@sezorus.com)

LinkedIn:  [Kseniia Zozulia](https://www.linkedin.com/in/629bb187)
