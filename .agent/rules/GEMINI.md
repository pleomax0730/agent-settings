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
