---
description: Generate Conventional Commit messages
---

## Purpose

Generate Conventional Commit messages.

## Behavior

* If staged changes exist, generate one conventional commit message describing those changes.
* If no staged changes exist, generate one or more conventional commit messages based on all changes discussed across the conversation.

## Rules

* Follow the Conventional Commits specification.
* Use correct commit types (`feat`, `fix`, `docs`, `refactor`, `chore`, `test`, etc.).
* Include scopes only when meaningful.
* Use imperative, concise descriptions.