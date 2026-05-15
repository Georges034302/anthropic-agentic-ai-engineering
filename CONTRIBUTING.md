# Contributing

> **Agentic AI Engineering Focus**
>
> This project is dedicated to agentic AI engineering. All contributions should align with the goal of building, documenting, or improving agentic AI systems and workflows—not general software development or unrelated AI topics.

Thanks for your interest in improving **Anthropic Agentic AI Engineering**! Contributions of all sizes are welcome — fixing typos, clarifying explanations, adding examples, or proposing new sections.

## Ways to Contribute

- **Fix issues** — typos, broken links, factual errors, or outdated references.
- **Improve clarity** — rephrase confusing passages, add diagrams, or expand examples.
- **Extend content** — propose new sub-sections, exercises, or supplementary material.
- **Add resources** — relevant papers, articles, or tools for the references list.

## Workflow

1. **Fork** the repository and create a feature branch from `main`:
   ```bash
   git checkout -b docs/your-change-summary
   ```
2. **Make your changes** in the appropriate file:
   - Course content → [`modules/`](modules)
   - Course index / TOC → [`docs/index.md`](docs/index.md)
   - Course tree / structure → [`docs/course_overview.md`](docs/course_overview.md)
   - Glossary terms → [`docs/glossary.md`](docs/glossary.md)
   - Repository overview → [`README.md`](README.md)
3. **Verify** Markdown renders correctly and all links resolve.
4. **Commit** with a clear, conventional message:
   ```
   docs(module-3): clarify MCP server lifecycle
   ```
5. **Open a Pull Request** against `main` describing the change and its motivation.

## Style Guidelines

- Use **Markdown** with consistent heading levels (`#`, `##`, `###`).
- Keep section numbering aligned with the existing `X.Y` scheme inside each module.
- Prefer short, descriptive bullet points with an em dash (`—`) separating term and explanation.
- Use relative links between repository files (e.g. `../modules/Module_3_MCP_Model_Context_Protocol.md`).
- Wrap code, file names, and technical identifiers in backticks.
- Keep tone neutral, technical, and vendor-respectful.

## Adding a New Glossary Term

1. Open [`docs/glossary.md`](docs/glossary.md).
2. Insert the term **alphabetically** under the correct letter heading.
3. Provide a concise definition (1–3 sentences) and, when useful, link to the module where it is introduced.

## Reporting Issues

If you find a problem but do not have a fix ready, open a GitHub Issue describing:
- Where the problem is (file + section).
- What is wrong or unclear.
- A suggested improvement, if any.

## License of Contributions

By submitting a contribution, you agree that your work will be licensed under the same [MIT License](LICENSE) that covers the rest of this repository.
