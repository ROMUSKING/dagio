# Contributing to DAGIO

First off, thank you for considering contributing to DAGIO! It's people like you that make open source such a great community. We welcome any and all contributions, from bug fixes to documentation improvements to new features.

## Code of Conduct

This project and everyone participating in it is governed by the [DAGIO Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior.

## How Can I Contribute?

### Following the Blueprint

Our project has a guiding vision and a phased development plan outlined in our **[Action Plan Blueprint](docs/blueprint.md)**. We kindly ask that contributions align with this blueprint to ensure a cohesive and focused development effort.

If you have an idea for a major feature or change that isn't in the blueprint, please **open an issue first** to discuss it with the maintainers. This helps ensure your hard work aligns with the project's direction.

### Finding Something to Work On

The easiest way to get started is to look for issues tagged with `good first issue` or `help wanted` in our GitHub Issues. These are tasks that have been identified as good entry points into the codebase.

If you find an issue you'd like to work on, please leave a comment to let others know you're on it.

## Your First Code Contribution

Ready to contribute? Here's how to set up your local environment and submit your changes.

1.  **Fork the repository** on GitHub.
2.  **Clone your fork** locally: `git clone https://github.com/your-username/dagio.git`
3.  **Set up your environment** by following the instructions in `docs/setup.md`.
4.  **Create a new branch** for your changes: `git checkout -b feature/your-feature-name` or `fix/the-bug-you-are-fixing`.
5.  **Make your changes.** Write clean, readable code that follows the project's architectural patterns.
6.  **Add tests** for your changes to ensure they work as expected and don't break existing functionality.
7.  **Format and lint your code.** We enforce standard Rust formatting and linting rules.
    ```bash
    cargo fmt
    cargo clippy -- -D warnings
    ```
8.  **Commit your changes** with a clear and descriptive commit message.
9.  **Push your branch** to your fork: `git push origin feature/your-feature-name`
10. **Open a Pull Request** to the `main` branch of the `romusking/dagio` repository.

## Pull Request Process

1.  Ensure your PR has a **clear title and description**. Explain the "what" and "why" of your changes. If it resolves an existing issue, link to it (e.g., `Fixes #123`).
2.  The core team will review your PR. We may ask for changes or improvements.
3.  Once your PR is approved and all automated checks (CI) have passed, it will be merged.

Thank you again for your contribution!