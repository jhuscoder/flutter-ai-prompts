## Global Master Application Skeleton Rollout

### Execution Goal
Scaffolds the absolute root structure of the application. It creates the global core layers, configures dependencies, initializes environment parsers, and establishes the foundational structure so feature-level work can begin immediately.

### The Prompt
> Act as a Principal Platform Engineer. Your task is to initialize the complete, production-grade application skeleton directory tree based entirely on our `CLAUDE.md` standards. 
> 
> Execute the following tasks systematically:
> 1. **Core Package Orchestration**: Update or verify the project's `pubspec.yaml` to include essential enterprise libraries under their appropriate versions: `flutter_riverpod`, `riverpod_annotation`, `go_router`, `dio`, `flutter_secure_storage`, `shared_preferences`, `freezed_annotation`, and development dependencies `build_runner`, `riverpod_generator`, and `freezed`.
> 2. **Environment & Security Gates**: Create a boilerplate `.env.json` at the root with standard keys (`API_BASE_URL`, `ENCRYPTION_KEY`). Ensure your logic prints an explicit reminder to add `.env.json` to the project's `.gitignore` if it isn't already present. Create a `lib/core/config/app_environment.dart` file using `String.fromEnvironment` to safely parse these values.
> 3. **Global Subdirectory Generation**: Generate the master architecture folder layout precisely:
>    - `lib/core/theme/` (Placeholder for light/dark app themes)
>    - `lib/core/constants/` (Placeholder for unified design tokens like spacing/sizes)
>    - `lib/core/error/` (Placeholder for type-safe system failures)
>    - `lib/core/network/` (Placeholder for our intercepted HTTP client)
>    - `lib/core/storage/` (Placeholder for secure storage and cache services)
>    - `lib/core/navigation/` (Placeholder for our declarative go_router routing table)
>    - `lib/features/` (The empty feature root folder)
> 4. **Project Health Synchronization**: Run `flutter pub get` followed by a preliminary `flutter pub run build_runner build --delete-conflicting-outputs` to guarantee that code-generation paths are fully aligned and valid right from day zero.
> 
> Do not generate any UI pages or mock features yet. Only deploy the absolute structural skeleton, and report back with a clear tree diagram mapping the resulting structure.
