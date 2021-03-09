<!-- markdownlint-disable MD025 -->
<!-- markdownlint-disable MD040 -->
# Team

## Code Review and Collaboration

be open to criticism and look for ways to help others out.

other people have the advantage of a different perspective.
___
If a developer implements a feature, they don't have proprietary rights to the maintenance or enhancement of that feature.
they should be open to others working on that feature and develop with that in mind.

# Code

## Principles

### Code using the DRY (don't repeat yourself) method

If you need to write the same code twice, abstract the algorithm into code that can be reused. This also applies to object inheritance. Even though there can be more lines of code at the start, it will allow for easier refactoring and development later on.

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
___

### TODOs: Just do it

There shouldn't be any reason why a PR or commit would include a `// TODO` unless there's some code that you the developer are unable to implement. In those circumstances make sure to properly document what exactly needs to be done and add a `throwUnimplementedError()` when applicable.

# Flutter

Avoid using top level functions or variables unless they are defined as static.

Avoid using public variables in favor of get/set methods.

All local function variables should be defined as private.

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
    bool _visible = _progress == Progress.loaded && canShow == true;

    return Visibility(
        visible: _visible,
        child: ...
    );
}
...
```

## Project Structure

```
|- lib
    |- components
        |- component_1.dart
        |- component_2.dart
    |- api_1.dart
    |- api_2.dart
    |- pages
        |- page_1
            |- page_1.dart
            |- components
                |- component_1.dart
                |- component_2.dart
    |- assets
        |- images
        |- fonts
```

## Comments

> When in doubt, comment.

Use `//` for comments and `///` for descriptions (i.e. anything you want included in the documentation).
**All** comments should follow this formatting:

- A space between the slashes and the comment
- A capital first letter
- A period at the end of the sentence

___
Every public variable and method should have a description.

Variable description example:

```dart
/// Called in initState() when the tutorial passes checks to show on screen.
final Function onShow;
```

Method description example:

```dart
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
```

___

## Class Structure

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

# GitHub

## Commits

?> In order to keep the commit history clean and make collaboration more fluid, being thoughtful about when and what you commit can save a lot of time.

Here are some general rules of thumb:

- Keep titles short yet descriptive. *(e.g. "Fixed #134", or "Updated Apple Sign In Button")*
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

# Helpful links

- [Firestore Best Practices](https://firebase.google.com/docs/firestore/best-practices)
- [Effective Dart](https://dart.dev/guides/language/effective-dart)
- [Overstep Developer Handbook](#team)
- [Inspiration](https://www.youtube.com/watch?v=dQw4w9WgXcQ)
