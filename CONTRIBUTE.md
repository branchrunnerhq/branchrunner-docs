# Contributing to the BranchRunner documentation

Thank you for improving the BranchRunner documentation. Contributions should remain useful to both first-time evaluators and experienced operators.

## Make a change

1. Fork or clone this repository.
2. Create a focused branch from the current documentation branch.
3. Edit or add Markdown files under `docs/product`.
4. Preview the Markdown and check every internal and external link.
5. Open a pull request describing the reader problem and the proposed improvement.

## File requirements

Every documentation file must:

- use a lowercase, hyphenated filename;
- use the `.md` extension;
- contain `title`, `description`, `group`, and numeric `order` front matter;
- contain one documentation topic;
- omit a level-one heading because the website renders the page title;
- begin content sections at `##`;
- use fenced code blocks with a language identifier where possible;
- use relative links for files in this repository;
- avoid MDX, raw HTML, scripts, and framework-specific components.

Use this template:

```markdown
---
title: Example page
description: What the reader will learn or accomplish.
group: Getting started
order: 30
---

A short introduction that establishes the outcome.

## First task

Explain the task and provide a minimal example.
```

## Writing style

- Lead with the reader's outcome.
- Prefer concrete steps and copyable examples.
- Define product-specific terms on first use.
- State prerequisites and permissions explicitly.
- Do not invent availability, pricing, guarantees, security claims, or roadmap commitments.
- Use **BranchRunner** consistently.
- Keep commands, file paths, API fields, and configuration keys in inline code.

## Pull-request checklist

Before requesting review, confirm that:

- the front matter is complete and the navigation order is intentional;
- commands and examples are technically accurate;
- links resolve without requiring a private account unless clearly identified;
- screenshots or transient UI descriptions are avoided when text is sufficient;
- secrets, tokens, customer data, and private repository information are absent;
- related pages are updated when terminology or behavior changes;
- a version-sensitive change explains which BranchRunner release introduced it.

## Publishing a documentation version

Maintainers publish immutable documentation by tagging a commit that contains the complete docs tree:

```bash
git tag v1.4.0
git push origin v1.4.0
```

The website configuration must then include that tag in `NEXT_PUBLIC_DOCS_REFS`. Do not move or replace an existing release tag; create a new patch tag for corrections.

## License of contributions

By submitting a contribution, you agree that it is licensed under the same [CC BY 4.0 license](LICENSE) as the rest of this documentation.
