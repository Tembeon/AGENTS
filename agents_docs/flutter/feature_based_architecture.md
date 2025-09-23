## Architecture Overview
This is a feature-first architecture aimed at fast and convenient development without adding layers "for the sake of layering" or complicating the initial setup. See the diagram below to understand the main ideas.

Goal: reduce the project to two top-level layers — `core` (reusable core) and `features` (features). We remove ceremonial layers added "for correctness", keeping only what helps write and maintain code quickly and truly speeds up development.

- Clear boundaries: features don’t know about each other directly; they interact only via the core or navigation (which also lives in the core).
- Minimal layering within a feature: `presentation` + `controllers` + `data` as needed.
- Shared services, utilities, and navigation live in `core`.

```
├── core                                    # shared functionality, the project core.
│   ├── app                                 # everything related to the "application"
│   │   ├── config                          # app configuration, e.g., Flavors, env constants, feature flags
│   │   ├── router                          # app navigation config, guards, routes
│   │   ├── theme                           # app theme configuration
│   │   ├── locale                          # app locale configuration
│   │   └── widget.dart                     # the application entry widget
│   ├── data                                # app data layer, encapsulates all data-fetching logic
│   │   ├── models                          # application data models
│   │   │   ├── local                       # models that exist only in the app and are used globally
│   │   │   └── remote                      # models representing remote contracts (API schemas)
│   │   ├── repository                      # repositories: contracts and implementations
│   │   └── sources                         # data sources
│   │       ├── local                       # local data source (DB/cache)
│   │       └── remote                      # remote data source (API)
│   ├── di                                  # global dependency injection for the app
│   ├── utils                               # utilities used across the app
│   │   ├── controllers                     # global controller setup (BLoC/state/observers)
│   │   ├── errors                          # shared errors and errors-contracts
│   │   ├── extensions                      # Dart extensions
│   │   └── network                         # HttpFactory, middlewares, tokens
│   └── widgets                             # shared widgets, e.g., Shimmer, ImageLoader
├── features                                # application features (services/user-interfact)
│   ├── example                             # sample feature name
│   │   ├── controllers                     # controllers used within this feature
│   │   ├── example_scope.dart              # the feature’s data scope, usually InheritedModel/Widget/Notifier
│   │   └── example_screen.dart             # entry point of the feature
├── main.dart                               # application entry point
```