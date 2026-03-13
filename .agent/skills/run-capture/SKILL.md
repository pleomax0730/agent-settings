---
name: run-capture
description: |
  When running shell or Python commands where the output might get stuck or
  is not captured reliably via run_command, redirect stdout+stderr to a
  timestamped file in /tmp, then read that file to get the result.
  Use this skill whenever standard run_command output is unreliable 
  (e.g. long-running uv run scripts, background processes, etc.)
---

# Run & Capture Skill

## When to use this skill

Use this skill instead of a plain `run_command` when:

- The command involves `uv run` with heavy package loading
- The command is likely to go to background and produce no visible output
- You need to reliably read stdout + stderr as text
- The previous `run_command` attempts got stuck or returned no output

## Step-by-step instructions

### Step 1 – Choose a unique tmp filename

Pick a descriptive tmp file name with a timestamp or incremental ID:

```
/tmp/cmd_<short_description>_<YYYYMMDD_HHMMSS>.txt
```

Example: `/tmp/cmd_verify_3_2_20260304_163000.txt`

### Step 2 – Run the command with output redirect

Use `run_command` with the output redirected to the tmp file.
Always append `2>&1` so that both stdout and stderr are captured:

```bash
<your-command> > /tmp/cmd_<name>.txt 2>&1; echo "EXIT:$?" >> /tmp/cmd_<name>.txt
```

For long-running Python commands, disable stdout buffering so progress is written
to the file immediately. Prefer one of these forms:

```bash
PYTHONUNBUFFERED=1 python -u <script>.py > /tmp/cmd_<name>.txt 2>&1; echo "EXIT:$?" >> /tmp/cmd_<name>.txt
```

```bash
PYTHONUNBUFFERED=1 uv run <script>.py > /tmp/cmd_<name>.txt 2>&1; echo "EXIT:$?" >> /tmp/cmd_<name>.txt
```

Example:

```bash
uv run scripts/verify_3_2.py > /tmp/cmd_verify_3_2.txt 2>&1; echo "EXIT:$?" >> /tmp/cmd_verify_3_2.txt
```

**Important settings for `run_command`:**

- `SafeToAutoRun`: set to `true` for read-only verification scripts
- `WaitMsBeforeAsync`: set to `5000` (5 seconds is enough since the command exits quickly after redirecting)

### Step 3 – Read the tmp file

Use `view_file` to read the captured output:

```
view_file /tmp/cmd_<name>.txt
```

This gives you the complete stdout + stderr synchronously, regardless of
how long the underlying command took to complete.

### Step 4 – Clean up (optional)

After reading, you may optionally delete the file:

```bash
rm /tmp/cmd_<name>.txt
```

But this is not strictly necessary since `/tmp` is cleared on reboot.

## Full example

```bash
# Step 2: run and capture
uv run scripts/verify_gpt52_1_1.py > /tmp/cmd_verify_gpt52_1_1.txt 2>&1; echo "EXIT:$?" >> /tmp/cmd_verify_gpt52_1_1.txt
```

Then:

```
view_file /tmp/cmd_verify_gpt52_1_1.txt
```

## Notes

- Do NOT use `WaitMsBeforeAsync` larger than 10000ms. The redirect itself is instant;
  the file will contain the full output once the command finishes.
- Always add `echo "EXIT:$?" >> <file>` at the end so you can see the exit code.
- For Python scripts that print progress during execution, always use `python -u`
  or `PYTHONUNBUFFERED=1`; otherwise the tmp file may stay empty until the process exits.
- `/tmp` files persist across the session but are cleared on reboot. That is fine.
