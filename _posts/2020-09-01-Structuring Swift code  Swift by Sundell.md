---
layout: post
title:  "Structuring Swift code | Swift by Sundell"
date:   2020-09-01 07:37:30 +0800
categories: Swift Cocoa
---
# Structuring Swift code | Swift by Sundell

One thing that almost all apps and frameworks have in common is that they tend to grow in both size and complexity as time passes. What might have started as a simple idea worked on by a single developer can often quickly transform into a much larger endeavor involving multiple teams and people with different levels of experience.

As a project grows it becomes more and more important to maintain a solid and consistent structure, but at the same time it also becomes increasingly difficult to do so - and it's very common to end up with a large code base that is tricky to navigate and takes a long time for new developers to get into.

This week, let's take a look at some tips and tricks as to how we can improve the structure of our Swift projects in a few key ways.

Level up your iOS development skills with a subscription to 3,000+ video tutorials for just $99/year. With Black Friday book bundles starting at just $99.97, the best investment you can make in your development career is waiting for you at [raywenderlich.com](https://store.raywenderlich.com/).

## Code dumping grounds

Arguably the main objective of any project structure is to both provide an easy way for any developer to get an overview of what the project and its various parts do, and to be able to quickly find and work on any given component, function or type.

To be able to fulfill that objective, a good structure also needs to be constantly evolving along with the project. Even if we had the best plan and the most clean structure to begin with, without ongoing tweaks and maintenance it's easy for parts of our code base to grow out of control and sort of become *"code dumping grounds"*.

An easy way to identify code dumping grounds is to look for types, files or folders that contain a lot of different unrelated functionality. For example, a very common starting point when building iOS apps is to create a `BaseViewController` that contains common functionality for all view controllers. While a *base class* like that might seem like a really convenient solution - and at first it might just have a small number of features - it's very easy (and common!) for such a type to become the default place to put *any kind* of shared functionality, quickly turning it into a dumping ground for all sorts of code that has little to nothing in common.

Another common source of code dumping grounds are folders called something like `Library`, `Utilities` or `Helpers`. Again, such a folder might start out with just a handful of extensions and helper methods - but can quickly become a *catch-all* for any kind of code, and basically turn into a place to put code that we simply don't know where else to put.

## Breaking things 

The core problem with code dumping grounds is that we've created a folder, file or type with *too broad of a scope*. You can call almost anything a *"Utility"* or a *"Helper"*, and a name with the prefix *"Base"* doesn't really tell us anything about what something is *supposed to do*. Instead - if we try to use more clear, focused names and give the various parts of our code base a much more narrow scope - it usually becomes much easier to maintain a more well-defined structure.

For example, to break up a `BaseViewController` dumping ground into smaller, more clearly named parts, we can use *child view controllers* - and leverage composition to mix and match the various functionality that we need in our different view controllers. Check out *["Using child view controllers as plugins in Swift"](https://www.swiftbysundell.com/articles/using-child-view-controllers-as-plugins-in-swift)* for some concrete examples on how that can be done.

Along the same lines, if we have a huge file called `String+Utilities.swift` that contains a lot of different `String` extensions, we can split it up into multiple files that each contain a subset of those extensions. We might have one for splitting strings (`String+Splitting.swift`), one defining API constants (`String+APIConstants.swift`), and so on. That way we're constantly *"forced"* to consider where a certain extension belongs, and we usually end up with a structure that makes things a lot easier to find and work with.

Successfully breaking things up is all about balance. If we break things up *too much* then we're not really improving the structure of our project, as we'll just end up with a large number of files and types that become hard to make sense of as a whole. One way to achieve such a balance is by following the *"Rule of Threes"*:

> Every time we end up with three parts of a type, folder or file that could be grouped together - let's try to do so.

For example, let's say that we're building an address book app that has a `ContactViewController` for displaying one of the user's contacts. Its UI has three components - a header, a table view displaying all of the contact's info and a view containing various actions. Right now these are all set up within `ContactViewController` itself, like this:

```swift
class ContactViewController: UIViewController {
    private lazy var headerImageView = UIImage()
    private lazy var headerLabel = UILabel()
    private lazy var headerButton = UIButton()

    private lazy var infoTableView = UITableView()

    private lazy var actionsTitleLabel = UILabel()
    private lazy var actionsSubtitleLabel = UILabel()
    private lazy var actionsStackView = UIStackView()
}
```

Applying the *Rule of Threes* to the above class, we can see that it might benefit from being broken up a bit. We have three properties that share the `header` prefix, and the same is also true for the `actions` prefix. That tells us that we can probably extract those properties into their own dedicated types, like this for the header view:

```swift
class ContactHeaderView: UIView {
    let imageView = UIImageView()
    let label = UILabel()
    let button = UIButton()
}
```

If we also do the same for the properties relating to actions, we can really improve the structure of `ContactViewController` by having it simply manage those new, more high-level container views:

```swift
class ContactViewController: UIViewController {
    private lazy var headerView = ContactHeaderView()
    private lazy var infoTableView = UITableView()
    private lazy var actionsView = ContactActionsView()
}
```

It's now much easier to get an overview of what `ContactViewController` does, and it becomes much less likely that it'll turn into a code dumping ground, now that it has a more solid structure.

*Another way to achieve the same thing could be to turn `ContactViewController` into a [custom container view controller](https://www.swiftbysundell.com/articles/custom-container-view-controllers-in-swift)*.

Here's another example in which we have a large function for saving a document in a word processing app. Saving a document requires a lot of different steps, and right now all of those steps are performed inline in that same function. Here's how the various attributes of the document's title are processed:

```swift
func saveDocument() {
    guard let titleText = titleLabel.text else {
        return
    }

    guard !titleText.isEmpty else {
        return
    }

    let titleFontIndex = titleFontPicker.selectedSegmentIndex
    let titleFont = fonts[titleFontIndex]
    let titleAlignmentIndex = titleTextAligmentPicker.selectedSegmentIndex
    let titleAlignment = textAlignments[titleAlignmentIndex]

    ...
}
```

Same thing here, let's apply the Rule of Threes, and as we have more than three references to something dealing with the document's title - let's extract that functionality into its own function, like this:

```swift
func makeTitle() -> Article.Title? {
    guard let text = titleLabel.text else {
        return nil
    }

    guard !text.isEmpty else {
        return nil
    }

    let fontIndex = titleFontPicker.selectedSegmentIndex
    let font = fonts[fontIndex]
    let alignmentIndex = titleTextAligmentPicker.selectedSegmentIndex
    let alignment = textAlignments[alignmentIndex]

    return .init(text: text, font: font, alignment: alignment)
}
```

As you can see above, performing these extractions and refactors in order to improve the structure of our code can also really improve the readability of it. Instead of always having to reference *"title"* everywhere, we can now drop that prefix and end up with much cleaner code 👍.

Just like any rule in programming, the trick with the Rule of Threes is not learning it - but rather deciding when and when not to apply it. Like most rules (at least in programming 😅) - it's completely fine to break it, given the right circumstances.

## Features

Another useful technique when it comes to improving the structure of a component or an app as a whole is to break it down in terms of what *features* its composed of. Even though we might mostly think of features as the top-level parts of the UI - an app usually has a lot more features than what is exposed to the user.

For example, if our app contains a cluster of classes and functions dedicated to parsing URLs, those could be viewed as a *URL parsing feature* - or the networking layer of our app could be called its *networking feature*. Grouping the types that belongs to a given feature together can be a nice starting point for a structure that also usually scales very well as new features are added. All features kind of become their own little subsystem with their own internal structure.

For example, here's how one user-facing feature (Search) and one system-level feature (Networking) could be organized in our Xcode project:

- Features
  - Search
    - View Controllers
      - SearchResultsViewController.swift
      - SearchViewController.swift
    - Models
      - SearchNetworkResponse.swift
      - SearchResult.swift
    - Views
      - SearchBar.swift
      - SearchTableViewCell.swift
    - Logic
      - SearchLogicController.swift
      - SearchResultsLoader.swift
  - Networking
    - Models
      - Request.swift
      - Endpoint.swift
    - Logic
      - DataLoader.swift
      - RequestFactory.swift
    - Extensions
      - URL+Endpoint.swift
      - URLSession+RequestFactory.swift

The nice thing about using features to structure and organize our project, the way we do above, is that it provides a nice extra level of hierarchy - rather than having folders like *View Controllers*, and *Models* on the top level (since those would quickly risk becoming dumping grounds).

Level up your iOS development skills with a subscription to 3,000+ video tutorials for just $99/year. With Black Friday book bundles starting at just $99.97, the best investment you can make in your development career is waiting for you at [raywenderlich.com](https://store.raywenderlich.com/).

## Conclusion

Maintaining a solid project structure is mostly about defining some key principles (like the Rule of Threes or organizing things by feature) as to how you and your team want to organize your code - and then continuously taking the time to re-arrange things according to those principles. One way of doing that is by adhering to the "*Scout rule*" - whenever you touch a part of the code base, you'll try to leave it better than how you found it.

Like most things in programming, there are no silver bullets when it comes to organizing and structuring a project. Every project is different and designing (and updating) a structure that is specific for each project is usually the way to go - while still trying to stick to common naming conventions and making the hierarchies we set up as intuitive as possible.

What do you think? How do you usually structure your Swift projects? Do you use any of the principles from this post, or is it something you'll try out? Let me know - along with your questions, feedback and comments - [on Twitter @johnsundell](https://twitter.com/johnsundell).

Thanks for reading! 🚀