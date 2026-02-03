---
trigger: always_on
description: dummy rule description
---

# Project Configuration

## Run Script Strategy
(Use `Start-Process` to run in background, capture PID, and log output)

### 1. General Python Script
- **Use case:** Data processing, utility scripts.
- **Command:** `uv run python`
- **PowerShell Command Pattern:**
  ```powershell
  $env:PYTHONUTF8=1; $p = Start-Process -FilePath "uv" -ArgumentList "run python -u <script_name.py>" -PassThru -RedirectStandardOutput "logs/<script_name>.log" -RedirectStandardError "logs/<script_name>.log" -WindowStyle Hidden; Write-Host "Script started with PID: $($p.Id)"
  ```

### 2. FastAPI Server
- **Use case:** Starting the API server during development.
- **Command:** `uv run fastapi run`
- **PowerShell Command Pattern:**
  ```powershell
  # Note: Replace "app/main.py" with your actual entry point
  $env:PYTHONUTF8=1; $p = Start-Process -FilePath "uv" -ArgumentList "run fastapi run app/main.py" -PassThru -RedirectStandardOutput "logs/api.log" -RedirectStandardError "logs/api.log" -WindowStyle Hidden; Write-Host "FastAPI Server started with PID: $($p.Id)"
  ```

## Stop Script Strategy
(Force stop and log the termination, including all child processes)

- **PowerShell Command Pattern:**
  ```powershell
  taskkill /F /T /PID <PID>; Add-Content -Path "logs/<log_name>.log" -Value "[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] [INFO] Process <PID> and its child processes terminated forceably."
  ```

# Python Architectural Principles (Modern Minimalist)

## 1. Vertical Slice Architecture (VSA)
**Description**: Organize code by business features rather than technical layers (MVC). Each feature should be a self-contained unit of work.
*   **High Cohesion**: Keep related code together. Changes to a feature should be localized within its directory.
*   **Low Coupling between Slices**: Features should be independent. Avoid hard dependencies between different feature slices to minimize side effects.
*   **Scale Complexity Independently**: Simple features should remain simple. Introduce internal layers ONLY when a feature's internal complexity warrants it.

## 2. Dependency Inversion & Testability
**Description**: Isolate core business logic from unstable infrastructure and external dependencies.
*   **Low Coupling to Infrastructure**: Business logic should not depend on concrete implementations (e.g., Playwright, PostgreSQL). This allows switching tools without rewriting logic.
*   **Prefer Pure Data Structures**: Business logic should operate on simple, framework-agnostic data structures. **Pydantic** is highly recommended for defining type-safe contracts and validation at feature boundaries.
*   **Define External Contracts**: Use `typing.Protocol` (Static Duck Typing) to define abstract interfaces. It is preferred over `abc.ABC` because it doesn't require explicit inheritance, further decoupling the implementation from the slice.
*   **Mandatory Constructor Injection (DI)**: Classes must receive dependencies via `__init__`. Avoid direct imports/instantiation of technical components inside the logic layer. (e.g., FastAPI's `Depends` can be used as a DI mechanism in web contexts.)
*   **I/O-Free Unit Testing**: Business logic should be testable in milliseconds by injecting Mock/Fake implementations.

## 3. Practical Design Discipline (SRP & YAGNI)
**Description**: Focus on current maintainability and testability. Avoid over-engineering for hypothetical future needs.
*   **Single Responsibility Principle (SRP)**: One class or module should have one business reason to change. Separate distinct business processes (e.g., Capture vs. Merge).
*   **YAGNI (You Ain't Gonna Need It)**:
    *   **DON'T** create abstractions for "hypothetical database swaps".
    *   **DO** create abstractions to make "current unit tests fast and reliable".
    *   **DO** create abstractions when you have 2+ concrete implementations now.
*   **Composition over Inheritance**: Build complex functionality by composing simple, focused components instead of deep inheritance hierarchies.
