# Repository Guide

## Project purpose

This is a long-term engineering-development repository covering backend engineering, distributed systems, infrastructure, DSA, system design, MLOps, interview preparation, and side projects. It stores durable technical work: notes, implementations, exercises, designs, assessments, experiments, progress, references, and decisions. It is not a transcript archive.

## Technical preferences

- Go is the primary language; use it for DSA and backend engineering.
- Use Python primarily for ML and MLOps.
- Prefer practical implementation alongside theory, with technically rigorous explanations and implementations.
- Favor understanding over blindly accepting generated solutions.

## Learning philosophy

- Preserve the opportunity to attempt exercises before providing a full solution.
- Increase difficulty incrementally.
- Explain tradeoffs when implementation choices matter.
- Favor idiomatic Go and simple, readable implementations before clever optimizations.
- Include tests for meaningful implementations.
- Consider complexity and failure modes.
- Avoid unnecessary dependencies.

## Repository behavior

- Inspect before modifying; preserve existing work and avoid unrelated changes.
- Keep changes scoped, and update documentation when behavior or structure changes.
- Run relevant tests after implementation.
- Avoid large amounts of boilerplate and files without a clear purpose.
- Prefer standard-library functionality when appropriate.
- Record important architectural and engineering decisions.

## Git behavior

Use clear conventional-style commit messages, for example `feat(dsa): add binary search exercises`, `feat(go): implement worker pool`, `docs(system-design): add URL shortener design`, and `chore: initialize engineering workspace`.

Never commit secrets, API keys, credentials, tokens, `.env` files, private certificates, or machine-specific configuration. Do not automatically push to GitHub unless explicitly instructed.
