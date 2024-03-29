<!-- markdownlint-disable MD025 -->
<!-- markdownlint-disable MD040 -->

# Team

## Code Review and Collaboration

Be open to criticism and look for ways to help others out.

Other people have the advantage of a different perspective.

---

If a developer implements a feature, they don't have proprietary rights to the maintenance or enhancement of that feature.
They should be open to others working on that feature and develop with that in mind.

# Code

## Principles

### Don't Repeat Yourself

If you need to write the same code twice, abstract it into code that can be reused. This also applies to object inheritance. Even though there can be more lines of code at the start, it will allow for easier refactoring and development later on.

Common candidates for DRY:

- `if/else if` chains.
- `switch` statements.
- `enum` state filtering

```dart
// Don't
class Event {
    enum EventType {public, private}
    ...
}
```

```dart
// Do
class Event {
    ...
}

class PublicEvent extends Event {
    ...
}

class PrivateEvent extends Event {
    ...
}
```

### Code like you're winning the lottery tomorrow

If you win the lottery tomorrow and never come back to work, another developer should be able to seamlessly pick up where you left off because your code was well documented, intuitive, and easy to understand.

---

### Avoid Coupling

Don't have two separate classes that rely on each other for functionality directly. Sometimes coupling is unavoidable but it should be a **one-way** relationship.

If class `Foo` needs to be notified of changes to object `Bar`, consider using a `ValueNotifier` that makes announcements for specific values or implement `Listenable` on `Bar` itself for notifications of the entire object. That way, if another situation appears later where `Tuba` needs to also be notified of changes to `Bar`, it can simply listen to the already-existing method of notification.

### TODOs: Just Do It

There shouldn't be any reason why a PR or commit would include a `// TODO` unless there's some code that you the developer are unable to implement. In those circumstances make sure to properly document what exactly needs to be done and add a `throwUnimplementedError()` when applicable.

### Least Privilege

Avoid using public variables in favor of get/set methods. Only create getters/setters for what is necessary. If someone who isn't you uses a class you wrote without any other communication from you, it should be clear what can and **should** be done through comments and public methods.

### Don't Pull from the Void

Avoid using public top level functions or variables. Wrap them into a class as a static reference where most appropriate. This will also reduce and clarify import statements.

```dart
// Don't
mainUser = User(...);

void main() {
    runApp(MyApp());
}
...
```

```dart
// Do
class User {
    static mainUser = User(...);
    ...
}
```

## Style

### Class Structure

```
- variables
    - static
    - public
    - private
- constructors
    - default
    - named
    - factory
- methods
    - getters
    - setters
    - static
    - public
    - private
```

### Comments

> When in doubt, comment.

Use `//` for comments and `///` for descriptions (i.e. anything you want included in the documentation).
**All** comments should follow this formatting:

- A space between the slashes and the comment
- A capital first letter
- A period at the end of the sentence

---

Every public variable and method should have a description.

Variable description example:

```dart
/// Called in initState() when the tutorial passes checks to show on screen.
final Function onShow;
```

Method description example:

````dart
/// Bakes the given potato for 20 minutes or [customTime].
///
/// [potato] must not be null. If the potato isn't cooked, this function
/// will cook the potato for two minutes recursively until the potato is
/// cooked.
///
/// Throws [FullOvenException] if the oven is full.
///
/// Example:
/// ```dart
/// cook(new Potato()).then((bakedPotato) => enjoy(bakedPotato));
///```
Future<BakedPotato> cook(Potato potato {Duration customTime}) async {
    assert(potato != null);

    try {
        putInOven(potato);
    } catch on FullOvenException (e) {
        throw 'Oven full!!';
    }

    await Future.delayed(customTime ?? Duration(minutes: 20));

    if(!potato.cooked) return await cook(
        potato,
        customTime: Duration(minutes: 2),
    );

    return new BakedPotato(potato);
}
````

## Performance Optimization

Consider using `const` for unchanging values. They are optimized by the compiler to only ever create one
instance of the object without the coder storing a globally-accessible reference to an object.

- A `const` must have only final fields.
- They are immutable.
- All `const` objects are created in memory at the start of run-time.
- They are never garbage collected.

This results in performance improvements which can be significant over the course of a program.

---

Here is an example of the syntax required to take advantage of `const`.

```dart
class Point {
  static final Point ORIGIN = const Point(0, 0);
  final int x;
  final int y;
  const Point(this.x, this.y);
  Point.clone(Point other): x = other.x, y = other.y; //[2]
}

main() {
  // Assign compile-time constant to p0.
  Point p0 = Point.ORIGIN;
  // Create new point using const constructor.
  Point p1 = new Point(0, 0);
  // Create new point using non-const constructor.
  Point p2 = new Point.clone(p0);
  // Assign (the same) compile-time constant to p3.
  Point p3 = const Point(0, 0);
  print(identical(p0, p1)); // false
  print(identical(p0, p2)); // false
  print(identical(p0, p3)); // true!
}
```

Using Flutter objects, consider `EdgeInsets`. A common occurrence is:

```dart
Padding(
    padding: EdgeInsets.all(8),
    child: ...
)
```

Even though EdgeInsets is a class of immutable data and the constructor being used is const, it must be declared as `const` to be properly optimized.

```dart
Padding(
    padding: const EdgeInsets.all(8),
    child: ...
)
```

When in doubt, declare an item as const if you know it will not change in the life of the app and its values are known at compile time. Some examples of Flutter objects with const constructors are:

- `Text()`
- `Colors`
- `Borders`
- `DecoratedBox()`

## Libraries

Group items together that belong to the same collection. This reduces the number of import statements and assists with understanding the logical groupings of items.

There are two ways to create a library. Each has its own uses.

1. The [Library](#method-one-library-with-parts) method should be used for low level code such as an app's core API.
2. The [Export](#method-two-exports) method should be used for Flutter-level conceptual groupings such as a collection of Style widgets.

### Method: Library with Parts

With this method, each file can not exist on its own without the library.

- It relies on imports from the main library file.
- Private members are accessible to all files in the library.
- Separating parts of a would-be large file into individual pieces reduces conflict in source control.

```
|- lib
    |- styles
        |- styles.dart
        |- gradient_button.dart
        |- header.dart
```

At the top of styles.dart, add the line `library styles;`.

Below that, add all import statements required by the other files in the library.

Lastly, declare the other parts of the library.

```dart
library styles;

import 'dart:ui';

part 'gradient_button.dart';
part 'header.dart';
```

Then, in each file that will be part of that library (as defined by the `part` directive in components.dart)

```dart
part of styles;
```

Any other file in the project that wishes to use all of the contents of the library may now use a single import statement.

```dart
import 'package:<appName>/styles/styles.dart';
```

---

### Method: Exports

Create a library file similar to before. Then, declare the files that are part of the library with `export`.

- Each file is responsible for its own import statements including references to other files that will be part of the library.
- There are no import statements, part statements, or library statements.
- It is a (nearly) purely semantic grouping.
  - Using `show` and `hide` can force certain items to be effectively private even if they couldn't be made so for purposes of access within the library.

This method does not prevent the individual files from being imported individually as they do not have the `part of` directive.

```dart
export 'gradient_button.dart';
export 'header.dart';
```

# Flutter

State dependant code does **not** belong in the return statement of `build()` for stateful widgets. Define a local variable instead. Return statements shouldn't be 100+ lines long!!

```dart
// Don't
...
@override
Widget build() {
    return Visibility(
        visible: _progress == Progress.loaded && canShow == true,
        child: ...
    );
}
...
```

```dart
// Do
...
@override
Widget build() {
    bool visible = _progress == Progress.loaded && canShow == true;

    return Visibility(
        visible: visible,
        child: ...
    );
}
...
```

## Project Structure

```
|- lib
    |- style
        |- style.dart (Library file)
        |- button.dart (example)
        |- input_field.dart (example)
    |- api_1
        |- api_1.dart (Single-file Library)
    |- api_2
        |- api_2.dart (Multi-file Library)
        |- api_component_1.dart
        |- api_component_2.dart
    |- pages
        |- page_1
            |- page_1.dart (Library)
            |- components
                |- component_1.dart (Library part)
                |- component_2.dart (Library part)
    |- assets
        |- fonts
        |- images
```

### Style

This is where the individual elements of the app are collected into a library for a single import using an [Export Library](#method-exports). `style.dart` is the library that exports the visual elements. It also holds the `Style` class which contains constants for the app such as padding and border radii.

### Libraries

For any given library, the file intended for import (the library file) _must_ match the name of the folder it sits in except for Libraries based in Pages, since those are not for export.

Even if a library has only one file in it, it should still be placed in a folder.

Most API libraries for the app should be parts-based. Over the life of the app, it is likely that these files will grow quite large so using the parts setup will allow fewer merge conflicts.

### Pages

Each page should be a library with the components within its component folder as parts (see [Library with Parts](#method-one-library-with-parts)). The items within the components folder should have no use outside of the page they are used in.

If a component that was previously in a page-specific folder needs to be used in another location, move it to the global components library.

# GitHub

## Before Commits

1. Remove excess print statements.
2. Check/remove [TODOs](#todos-just-do-it).
3. Ensure all code has been commented according to [standards](#comments).
4. Organize imports. If using VS Code, you can press Ctrl + Shift + P or F1 to run 'Organize imports'.

## Commits

?> In order to keep the commit history clean and make collaboration more fluid, being thoughtful about when and what you commit can save a lot of time.

Here are some general rules of thumb:

- Keep titles short yet descriptive. _(e.g. "Fixed #134", or "Updated Apple Sign In Button")_
- Make use of the description to add details you couldn't fit in the title.
- Only check files directly pertaining to the commit.
- If what you're working on will take several commits to finish, make a new branch and store your changes there until you're ready to make a pull request.
- Commits should relate to only 1 issue or concept. More complicated issues or complex concepts should make use of a PR.

## Pull Requests

- When reviewing a PR, make sure the code is understandable, clean, and orderly.
- Don't merge unless the code is production ready.
- Every PR should have at least 1 reviewer.
- Prefer using the squash and merge option when merging branches.

# Apache Cordova

Don't.

Just don't.

# Starting from a Clean Install(Dev Environment setup)

1. Install Git.
   - https://git-scm.com/downloads
2. Install Flutter (follow all steps until "Windows setup").
   - https://flutter.dev/docs/get-started/install
3. If on Windows, install the Google USB driver for Android debugging.
   - https://developer.android.com/studio/run/win-usb
4. Install VS Code (follow all steps).
   - https://flutter.dev/docs/get-started/editor?tab=vscode
5. Install extensions for VS Code
   - Flutter (required)
   - Dart (required)
   - Code Spell Checker (useful for simple spelling mistakes)
   - Dart Data Class Generator (useful for automating boilerplate)
   - Prettier - Code formatter (multi-language code formatter)
   - Better Comments - Comment Decorator (Helps make TODOs and other comments stand out)
6. Download and save the [debug and release signing files](#) to somewhere you won't lose or move.
7. Edit your system environment variables to add the key `signing_keys` with the value of the directory that contains the files you downloaded.

# Full New Project Setup

1. Follow the instructions on the [Flutter Project Template](https://github.com/OverstepCo/flutter_project_template) for a new project.
2. If Firebase is not required, stop here.
3. Create a Firebase project on [Firebase](https://console.firebase.google.com). If you have multiple Google accounts and your Overstep email is not your default, you will need to make sure it shows the correct profile the top right.
4. Give the project a name.
5. Set the parent organization to `reddlegend.com`.
6. Do **_not_** enable Google Analytics.
7. Once it's done creating the project, you should find yourself on the overview page. Select iOS or Android to add to the app. You'll ultimately do both.
8. For iOS:
   1. Fill out the `iOS Bundle ID` field with the same bundle ID you set in the [Flutter Project Template](https://github.com/OverstepCo/flutter_project_template) steps.
   2. Save `GoogleService-Info.plist` to `/ios/Runner/` in the Flutter project files.
   3. Add that file to all targets in Xcode. MacOS is required for this step.
9. For Android:
   1. Fill out the `Android package name` field with the same package name you set in the [Flutter Project Template](https://github.com/OverstepCo/flutter_project_template) steps.
   2. Add the official™ debug key so that people can access Firebase on their devices (see [Clean Install](#starting-from-a-clean-install) for debug key). If you miss entering it here, you can add it later under Project Settings.
      1. If you don't know the exact values to enter for the debug keystore, go to where you saved the debug/release files in a terminal/command prompt. Enter the command `keytool -list -v -keystore debug-keystore.jks` and give the SHA1 value to Firebase.
10. Go to Project Settings using the gear wheel at the top of the side panel on Firebase. Edit the public-facing name and support email.
11. If Authentication is required, select it from the side panel. It should say something like "Get Started" to enable it. Enable the authentication methods relevant to the project.
12. If Firestore is required, select it from the side panel. It should say something like "Create database" to enable it. Start it in test mode.

# Helpful links

- [Firestore Best Practices](https://firebase.google.com/docs/firestore/best-practices)
- [Effective Dart](https://dart.dev/guides/language/effective-dart)
- [Overstep Developer Handbook](#team)
- [Inspiration](https://www.youtube.com/watch?v=dQw4w9WgXcQ)
