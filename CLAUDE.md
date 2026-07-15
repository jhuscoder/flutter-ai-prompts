# CLAUDE.md - Feature-First MVVM Engineering Standards

This document establishes the architecture patterns, performance rules, and security guidelines for this project. The AI Agent **MUST** read and enforce this Feature-First MVVM structure for every file generation, refactor, and terminal execution.

## 1. Feature-First MVVM Architecture

The codebase is organized by business feature rather than by layer type. Each feature must be an entirely self-contained module containing its own Model, View, and ViewModel boundaries. 

### Directory Structural Blueprint
When creating a new feature (e.g., `authentication`), enforce the following folder tree strictly:
```text
lib/features/authentication/
├── data/
│   ├── datasources/         # Remote API clients, local storage hooks
│   ├── dtos/                # Network data serialization mapping
│   └── repositories/        # Repository implementations
├── domain/
│   ├── models/              # Pure immutable models/entities
│   └── repositories/        # Abstract contracts/interfaces
├── view_model/
│   ├── auth_state.dart      # Immutable structural representation of the UI state
│   └── auth_view_model.dart # AsyncNotifier/Notifier handling UI actions
└── views/
    ├── auth_screen.dart     # Primary layout canvas (ConsumerWidget)
    └── widgets/             # Lightweight, single-responsibility sub-widgets
```

### Architectural Component Rules
*   **The View**: Must be a `ConsumerWidget` or `ConsumerStatefulWidget`. It reads state using `ref.watch()` and binds UI actions directly to the ViewModel methods. 
*   **The ViewModel**: Powered exclusively by **Riverpod** (`Notifier` or `AsyncNotifier`). It manages the feature's UI state and communicates with Domain Repositories. It must never reference UI elements or `BuildContext` directly.
*   **The Model**: Business entities must be immutable. Data Transfer Objects (DTOs) parse payloads from data sources and map into domain models before reaching the ViewModel.

## 2. Micro-Modularization & Widget Cleanliness

To guarantee extreme code readability and easy long-term maintenance, strict modularization guidelines apply to all visual layers:

### Single-Responsibility Widgets
*   **One Widget, One Action**: All custom UI widgets, layouts, and actionable buttons must serve one dedicated purpose only. Do not mix multiple business flows or mixed UI responsibilities into a single class.
*   **No Giant View Files**: The primary feature view (e.g., `auth_screen.dart`) must act only as a high-level compositional layout canvas. It should stitch child micro-components together rather than hosting inline layout implementations.
*   **Mandatory File Extraction**: Avoid keeping multiple widget classes inside a single document. If a layout section or interactive button grows beyond a basic atomic definition, it **MUST** be cleanly extracted into its own dedicated Dart file within the `views/widgets/` subdirectory.
*   **Declarative Callbacks**: Actionable buttons and micro-widgets must expose clear, declarative constructor callback functions (e.g., `VoidCallback onTap`). They should pass click behaviors up to the parent layer instead of coupling directly with unrelated dependency injections.

## 3. State Management & Storage Frameworks

### State Management (Riverpod)
*   **Engine Preference**: Use Riverpod code generation (`@riverpod`) for defining all providers, ViewModels, and dependency injections.
*   **State Immutability**: All states must rely on immutable data classes.
*   **Subscription Scopes**: Views must use precision watch selectors (`ref.watch(provider.select(...))`) to prevent whole-screen redraws when only small data segments mutate.

### Secure Storage (flutter_secure_storage)
*   **Target Scope**: Sensitive data generated at runtime (JWT tokens, encryption keys, session cookies, user credentials).
*   **Rule**: Absolute zero-tolerance for saving sensitive credentials in plain-text. Use hardware-backed secure storage.

### Local Cache Storage (shared_preferences)
*   **Target Scope**: Non-sensitive, persistent application states only (e.g., app onboarding completion status, theme preferences, local settings flags).
*   **Rule**: Keep keys structured and scoped per feature. Do not use for user identity files or API tokens.

## 4. Secret Management via Env JSON

### Secure Variable Ingestion
*   **Zero Hardcoding**: Do not embed raw keys, credentials, backend URLs, or secrets in the codebase.
*   **Storage Source**: All sensitive compilation keys must reside in a root-level `.env.json` file.
*   **Version Control Gate**: The `.env.json` file **MUST** be explicitly listed in the `.gitignore` file to ensure secrets are never leaked to public repositories.

### Compilation Parsing Protocol
*   **Dart Extraction**: Extract the secrets into your application using compile-time constants through the `String.fromEnvironment` syntax.
*   **Implementation Example**:
    ```dart
    class AppEnvironment {
      static const String apiBaseUrl = String.fromEnvironment('API_BASE_URL');
      static const String stripePublishableKey = String.fromEnvironment('STRIPE_PUBLISHABLE_KEY');
    }
    ```

## 5. Error Handling & Data Resilience

### Type-Safe Failures
*   **No Raw Exceptions**: ViewModels must never catch unmapped, raw platform exceptions (`Exception`, `SocketException`).
*   **Result Pattern**: Wrap all asynchronous operations from data repositories in a functional Result type (e.g., using packages like `fpdart` or custom `Result<Success, Failure>` classes).
*   **UI Boundaries**: ViewModels must convert data/domain failures into clean, user-friendly semantic states (e.g., `AuthenticationState.error(String message)`) before passing them to the View.

## 6. UI & Design System Alignment

### Theme Integration Rules
*   **No Hardcoded Styling**: Do not use raw hex colors (`Color(0xFF...)`) or arbitrary double values for text styles inside localized feature views.
*   **Context Extensions**: Always query the primary theme context tree (`Theme.of(context).colorScheme.primary` or `Theme.of(context).textTheme.bodyLarge`).
*   **Atomic Sizing Layouts**: Use a unified, central spacing design class (e.g., `AppSizes.paddingMedium`) to maintain layout spacing consistencies.

### Localization & Core Hardcoding Bans
*   **String Separation**: Zero tolerance for raw, hardcoded user-facing strings in the View file (`Text("Welcome Back")`).
*   **Localization Mapping**: Access localized string assets strictly through the application's localization framework configuration (e.g., `AppLocalizations.of(context)!.welcomeBack`).

## 7. High-Performance Optimization

### Memory Management
*   **Auto-Dispose**: Use `.autoDispose` or the default auto-disposing behavior of Riverpod code generation to ensure ViewModels clear memory when the corresponding View unmounts.
*   **Widget Instantiation**: Mark UI layers with `const` constructors where possible to skip redundant element tree layout cycles.

## 8. Automated CI/CD & Testing Requirements

### MVVM Testing Strategies
*   **ViewModel Tests (Unit)**: Test 100% of state transitions by overriding Riverpod providers with mocked abstract repositories (`ProviderContainer`).
*   **View Tests (Widget)**: Inject mock providers into target Views to ensure appropriate components render correctly based on specific state variants.

### Code Generation & Conflict Resolution
*   **State Conflicts**: If compilation errors or file state conflicts occur due to outdated generated elements, execute a recursive project sync gate:
    ```bash
    # Clean project build files completely
    flutter clean

    # Pull project network dependencies
    flutter pub get

    # Re-run build runner forcing output synchronization
    flutter pub run build_runner build --delete-conflicting-outputs
    ```

### Local Build Execution Gates
Run these checks before finalizing any code edits:
```bash
# Generate Riverpod files, Freezed models, etc.
flutter pub run build_runner build --delete-conflicting-outputs

# Pass environment variables to local emulator execution
flutter run --dart-define-from-file=.env.json

# Build the release bundle with environment variables and obfuscation
flutter build apk --release --dart-define-from-file=.env.json --obfuscate --split-debug-info=/<directory>

# Code format check
flutter format --set-exit-if-changed .

# Linter analysis
flutter analyze

# Execute test suite
flutter test
```
