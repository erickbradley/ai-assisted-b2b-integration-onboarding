# Repository Guidelines

## Project Structure & Module Organization

This repository currently contains the project overview in `README.md` and is in the architecture/MVP design phase. Keep high-level design documents in `docs/`, application code in `src/`, automated tests in `tests/`, and diagrams or other documentation assets in `assets/`. Place AWS infrastructure-as-code in `infra/`, grouped by deployable component or environment. Update `README.md` whenever the supported workflow or repository layout changes.

## Build, Test, and Development Commands

There is no application build, local server, or automated test command yet. Before submitting documentation or configuration changes, run:

- `git diff --check` — detects trailing whitespace and malformed patch formatting.
- `git status --short` — confirms that only intended files are included.
- `git diff -- README.md AGENTS.md` — reviews contributor-facing documentation changes.

When a build or test tool is introduced, add its reproducible commands to both this guide and the README; prefer checked-in scripts over undocumented local commands.

## Coding Style & Naming Conventions

Use two-space indentation for Markdown lists and configuration files unless the language formatter requires otherwise. Keep Markdown headings in sentence case, paragraphs short, and examples runnable. Use lowercase, hyphenated names for documentation and assets (for example, `docs/schema-mapping-flow.md`). Follow language-standard formatters for future source code and commit their configuration with the first module that uses them. For Terraform, use `terraform fmt` and descriptive `snake_case` resource names.

## Testing Guidelines

No testing framework or coverage threshold is configured. New executable code should arrive with automated tests under `tests/`, mirroring the source layout. Name tests after observable behavior, such as `test_mapping_requires_human_approval`. Infrastructure changes should eventually be validated with `terraform fmt -check` and `terraform validate` once Terraform files and provider configuration exist.

## Commit & Pull Request Guidelines

Recent commits use short, imperative, sentence-case subjects (for example, `Revise README with new title and project status`). Keep each commit focused and explain non-obvious design decisions in the body. Pull requests should include a concise summary, validation performed, related issue links, and screenshots or rendered diagrams for visual changes. Call out AWS cost, security, data-handling, or compatibility implications explicitly.

## Security & Configuration Tips

Never commit credentials, state files, or environment-specific secrets. The `.gitignore` excludes Terraform state, `.tfvars` files, override files, and CLI configuration; preserve those protections. Provide sanitized example configuration such as `terraform.tfvars.example` when contributors need documented inputs.

## Agent-Specific Instructions

When running commands such as Git, shell, build, or test commands on the user's behalf, include a brief beginner-level explanation of what was done and why. Mention the repository instructions that guided the action, and provide bite-size learning tips rather than long lessons. Keep explanations clear enough for someone developing foundational command-line and software-development skills.
