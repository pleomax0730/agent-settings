---
description: Perform modernize types, sorting import and formatting code in Python
---

<lint_format>
  - uv tool run ruff check --target-version py3xx --select E,F,I,UP --fix [FILES]...
  - uv tool run ruff format [OPTIONS] [FILES]...
</lint_format>
