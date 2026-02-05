---
description: "SwiftData conventions for CloudKit-enabled models"
globs: ["**/*.swift"]
alwaysApply: false
---

# SwiftData Conventions

If SwiftData is configured to use CloudKit:

- Never use `@Attribute(.unique)`.
- Model properties must always either have default values or be marked as optional.
- All relationships must be marked optional.
