# Calorie Tracker
Multi module app
<img src="https://user-images.githubusercontent.com/65896669/219024786-c99208cf-3984-433d-97fc-5792fd371bc8.gif" width="200" align="right" hspace="10">


* Tech-stack
    * [100% Kotlin](https://kotlinlang.org/) + [Coroutines](https://kotlinlang.org/docs/reference/coroutines-overview.html) - perform background operations
    * [Retrofit](https://square.github.io/retrofit/) - networking
    * [Jetpack](https://developer.android.com/jetpack)
        * [Navigation](https://developer.android.com/topic/libraries/architecture/navigation/) - in-app navigation
        * [Kotlin Flows](https://developer.android.com/kotlin/flow) - notify views about database change
        * [Lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle) - perform an action when lifecycle state changes
        * [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) - store and manage UI-related data in a lifecycle conscious way
        * [Room](https://developer.android.com/jetpack/androidx/releases/room) - store offline cache
    * [Hilt](https://dagger.dev/hilt/) - dependency injection
    * [Coil](https://github.com/coil-kt/coil) - image loading library
* Modern Architecture
    * Clean Architecture (at feature module level)
    * Single activity architecture using [NavComponent](https://developer.android.com/jetpack/compose/navigation)
    * MVVM (presentation layer)
    * [Android Architecture components](https://developer.android.com/topic/libraries/architecture) ([ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel), [Kotlin Flows](https://developer.android.com/kotlin/flow), [Navigation](https://developer.android.com/jetpack/androidx/releases/navigation))
    * [Android KTX](https://developer.android.com/kotlin/ktx) - Jetpack Kotlin extensions

* Testing
    * [Unit Tests](https://en.wikipedia.org/wiki/Unit_testing) ([JUnit 5](https://junit.org/junit5/) via
    [android-junit5](https://github.com/mannodermaus/android-junit5))
    * [Mockk](https://mockk.io/) - mocking framework
* UI
    * [Material design](https://material.io/design)
    * [Jetpack Compose](https://developer.android.com/jetpack/compose?gclid=Cj0KCQiAorKfBhC0ARIsAHDzslu18iedo1CyRG8nhYJqrrmMQfzxL-nVQa9Ilbea70GvnPZTHTRNOssaAorMEALw_wcB&gclsrc=aw.ds)
* Gradle
    * [Gradle Kotlin DSL](https://docs.gradle.org/current/userguide/kotlin_dsl.html)
    * [Gradle KTS](https://developer.android.com/studio/build/migrate-to-kts) Kotlin script
    * Custom tasks
    * Plugins ([SafeArgs](https://developer.android.com/guide/navigation/navigation-pass-data#Safe-args),
    [android-junit5](https://github.com/mannodermaus/android-junit5))

## Architecture

Feature related code is placed inside one of the feature modules.
We can think about each feature as the reusable component, equivalent of [microservice](https://en.wikipedia.org/wiki/Microservices) or private library.

The modularized code-base approach provides few benefits:
- better [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns). Each module has a clear API., Feature related classes live in different modules and can't be referenced without explicit module dependency.
- features can be developed in parallel eg. by different teams
- each feature can be developed in isolation, independently from other features
- faster compile time

### Module types and module dependencies

We have different kinds of modules in the application:

- `app` module - this is the main module. It contains code that wires multiple modules together and fundamental application configuration.
- `buildSrc` module - this module contains all version values so they can be reused/changed easily throught all the .build files
- `core UI` module - contains all shared data related to UI (this allows us to not import compose dependencies in __data__ and __domain__ modules)
- `core` module - contains all shared data apart from UI related
- `feature` modules - the most common type of module containing all code related to a given feature (containing __data__ , __domain__ and __presentation__ submodules)

### Feature module structure

`Clean architecture` is the "core architecture" of the application, so each `feature module` contains own set of Clean architecture layers:

Each feature module contains non-layer components and 3 layers with distinct set of responsibilities.

<img src="https://user-images.githubusercontent.com/65896669/219033599-0b998412-3ae7-4916-9643-39a9156095a3.jpeg" width="600" align="center" hspace="10">

<img src="https://user-images.githubusercontent.com/65896669/219026796-a4f0a618-89a1-4b68-b21b-87b5e6caeb22.png" width="200" align="center" hspace="10">

### Presentation layer

This layer is closest to what the user sees on the screen. The `presentation` layer is designed with `MVVM` - Jetpack `ViewModel` used to preserve data across activity restart, `actions` modify the `common state` of the view and then new state is edited to a view via `Kotlin Flows` to be rendered).

Components:
- **View (Fragment)** - presents data on the screen and pass user interactions to View Model. Views are hard to test, so they should be as simple as possible.
- **ViewModel** - dispatches (through `Kotlin Flows`) state changes to the view and deals with user interactions.
- **ViewState** - common state for a single view

### Domain layer

This is the core layer of the application. Notice that the `domain` layer is independent of any other layers. This allows to make domain models and business logic independent from other layers.
In other words, changes in other layers will have no effect on `domain` layer eg. changing database (`data` layer) or screen UI (`presentation` layer) ideally will not result in any code change withing `domain` layer.

Components:
- **UseCase** - contains business logic
- **DomainModel** - defines the core structure of the data that will be used within the application. This is the source of truth for application data.
- **Repository interface** - required to keep the `domain` layer independent from the `data layer` ([Dependency inversion](https://en.wikipedia.org/wiki/Dependency_inversion_principle)).

### Data layer

Manages application data and exposes these data sources as repositories to the `domain` layer. Typical responsibilities of this layer would be to retrieve data from the internet and optionally cache this data locally.

Components:
- **Repository** is exposing data to the `domain` layer. Depending on application structure and quality of the external APIs repository can also merge, filter, and transform the data. The intention of
these operations is to create high-quality data source for the `domain` layer, not to perform any business logic (`domain` layer `use case` responsibility).

- **Mapper** - maps `data model` to `domain model` (to keep `domain` layer independent from the `data` layer).
- **RetrofitService** - defines a set of API endpoints.
- **DataModel** - defines the structure of the data retrieved from the network and contains annotations, so Retrofit (Moshi) understands how to parse this network data (XML, JSON, Binary...) this data into objects.

## Dependency management

This project utilizes multiple mechanics to easily share the same versions of dependencies such as base-module to be interited from and BuildSrc module containing all version values to be reused.

## Design decisions

Read related articles to have a better understanding of underlying design decisions and various trade-offs.

* [Multiple ways of defining Clean Architecture layers](https://proandroiddev.com/multiple-ways-of-defining-clean-architecture-layers-bbb70afa5d4a)

### Android Studio

1. `Android Studio` -> `File` -> `New` -> `From Version control` -> `Git`
2. Enter `https://github.com/igorwojda/android-showcase.git` into URL field an press `Clone` button

### Command-line + Android Studio

1. Run `git clone https://github.com/igorwojda/android-showcase.git` command to clone project
2. Open `Android Studio` and select `File | Open...` from the menu. Select cloned directory and press `Open` button


### Cheat sheet

- [Core App Quality Checklist](https://developer.android.com/quality) - learn about building the high-quality app
- [Android Ecosystem Cheat Sheet](https://github.com/igorwojda/android-ecosystem-cheat-sheet) - board containing 200+ most important tools
- [Kotlin Coroutines - Use Cases on Android](https://github.com/LukasLechnerDev/Kotlin-Coroutine-Use-Cases-on-Android) - most popular coroutine usages

### Android projects
- to be updated...

## Author
[![Follow me](https://img.shields.io/twitter/follow/YosifKalchev?style=social)](https://twitter.com/yosifkalchev)
