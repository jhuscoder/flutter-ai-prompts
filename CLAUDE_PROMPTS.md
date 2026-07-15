# CLAUDE_PROMPTS.md - Active Engineering Playbook

This document contains production-grade operational prompts optimized for terminal-based AI agents (like Claude Code) or IDE assistants. Use these prompts verbatim or adjust the bracketed placeholders `[FEATURE_NAME]` to drive development during the 2-day sprint.

---

## 1. Feature Scaffolding & Architecture Initialization

### Execution Goal
Enforces a perfect feature-first architecture layout before any functional code is introduced. Prevents folder clustering and file pollution.

### The Prompt
> Act as a Senior Flutter Engineer. Initialize a new feature module named `[FEATURE_NAME]` using our strict feature-first architecture layout specified in `CLAUDE.md`. Create all necessary subdirectories exactly: `data/datasources`, `data/dtos`, `data/repositories`, `domain/models`, `domain/repositories`, `view_model`, and `views/widgets`. Generate only the abstract repository interface inside `domain/repositories/` and the empty immutable domain model file inside `domain/models/`. Do not write any UI or concrete implementations yet. Reference `CLAUDE.md` to ensure absolute structural validation before reporting success.

---

## 2. Micro-Modular UI Widget Assembly

### Execution Goal
Enforces the "One Widget, One Action" micro-modularization standard. Stops the agent from writing unmaintainable, thousands-of-lines-long view screens.

### The Prompt
> Implement the user interface for `[SCREEN_NAME]` within the `[FEATURE_NAME]` module. You must strictly follow Section 2 of our `CLAUDE.md` rules regarding micro-modularization. The primary screen file (`[SCREEN_NAME]_screen.dart`) must act exclusively as a high-level layout compositional canvas. Every actionable button, dynamic card, or isolated visual section **MUST** be cleanly extracted into its own dedicated, professionally named Dart file inside the `views/widgets/` folder. All custom sub-widgets must expose declarative constructor callbacks (such as `VoidCallback onTap` or `ValueChanged<T> onChanged`) to pass actions cleanly up to the view layout. Do not write inline styling, nested anonymous layouts, or inline business logic.

---

## 3. Riverpod AsyncNotifier ViewModel Generation

### Execution Goal
Generates modern, safe, code-generated Riverpod ViewModels that bind logic state changes directly to the extracted UI components.

### The Prompt
> Create the state management layer for the `[FEATURE_NAME]` module using Riverpod code generation (`@riverpod`). 
> First, define an immutable state representation class using `freezed` or a structured custom copyable data layout inside `view_model/[FEATURE_NAME]_state.dart` that explicitly models initialization, loading, success, and error variables. 
> Second, build the concrete `AsyncNotifier` ViewModel controller class inside `view_model/[FEATURE_NAME]_view_model.dart`. Ensure all async methods capture platform-specific exceptions securely, routing mapped error messages to the state model rather than leaking raw faults to the UI. Run the local build runner command when finished to ensure code generation compiles without error.

---

## 4. Secure Storage & Local Cache Core Client

### Execution Goal
Safely partitions persistent device storage, utilizing biometrics/hardware vaults for sensitive keys and plain-text file options for generic state variables.

### The Prompt
> Develop or update our core local storage layer. You must build a unified, abstract infrastructure wrapper that cleanly splits platform duties based on security context as per `CLAUDE.md`: 
> 1. Use `flutter_secure_storage` to write, read, and clear sensitive values generated at runtime (such as authentication JWT tokens, access cookies, or cryptographic state keys). 
> 2. Use `shared_preferences` to preserve non-sensitive local preference properties (such as app onboarding completion flags, UI language indices, or local application theme indices). 
> Expose a clean, strongly typed repository API interface that other feature-level ViewModels can easily inject and read from safely.

---

## 5. Pre-Commit Enterprise Code Review Gate

### Execution Goal
Acts as a static analysis quality checker before code hits production branches. Catches hardcoded strings, sloppy sizing variables, and memory leak vectors.

### The Prompt
> Execute a strict pre-commit enterprise-grade code review across all recently created or modified files in this sprint. Specifically audit and report on:
> 1. Any hardcoded UI text strings or labels that must be abstracted into localization configurations (`AppLocalizations`).
> 2. Elements that violate our micro-modular UI rules (e.g., custom widgets missing isolated Dart files or button logic tightly coupled to inline framework providers).
> 3. Hardcoded sizing metrics, padding values, or absolute hex colors bypass-coding our primary context theme rules.
> 4. Missing resource disposal handlers, open event subscriptions, or unmanaged controllers capable of triggering local memory leaks.
> Present your critique points as an organized audit list, and wait for my direct confirmation before implementing corrections.

---

## 6. ViewModel Unit & State Transition Testing

### Execution Goal
Enforces automated verification behaviors on the state machine layer, assuring data integrity without requiring manual emulator click tests.

### The Prompt
> Build a robust, automated unit testing module for our `[FEATURE_NAME]` ViewModel class. Initialize the validation script using a Riverpod `ProviderContainer` to ensure absolute component isolation. Fully mock out all external domain and data repository dependencies using `mocktail` or standard fake classes. Test 100% of the core state permutations—verify that triggering an active business routine successfully cycles the ViewModel through its loading indicator sequence, securely intercepts failure responses from the database layer, and appropriately updates final state structures with expected data values on success.

---

## 7. Automated Project Health Reset

### Execution Goal
Resolves local development caching blockages, schema errors, or missing elements caused by overlapping code generation routines.

### The Prompt
> The local compilation engine is currently experiencing code-generation mismatch conflicts. Execute our standard recovery pipeline exactly as outlined in our project guidelines:
> 1. Wipe out temporary platform caching completely using: `flutter clean`
> 2. Re-verify package system structures and fetch missing libraries using: `flutter pub get`
> 3. Re-run our global build runner pipeline while forcing overlapping schema replacements using: `flutter pub run build_runner build --delete-conflicting-outputs`
> Report back with active terminal log confirmations as soon as the project tree compiles in a healthy state.
