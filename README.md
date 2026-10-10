# BranchRunner documentation

This repository contains the public product documentation for [BranchRunner](https://branchrunner.com).

The documentation is intentionally stored as portable Markdown. The BranchRunner website reads these files directly and publishes selected Git tags as versioned documentation.

## Documentation layout

All product documentation lives in `docs/product`. Each file represents one public documentation page and begins with front matter:

```yaml
---
title: Page title
description: A concise search and navigation description.
group: Navigation group
order: 10
---
```

The filename becomes the page slug. For example, `docs/product/quickstart.md` is published as `/docs/quickstart`.

## Published versions

The website treats the working branch as **Latest** and release tags as immutable documentation versions. Published refs are configured on the website with an ordered list:

```env
NEXT_PUBLIC_DOCS_REFS=latest=main,v1.4.0=v1.4.0,v1.3.2=v1.3.2
NEXT_PUBLIC_DOCS_EDIT_REF=main
NEXT_PUBLIC_REPOSITORY_DOCS_ROOT=docs/product
```

A release tag must contain the complete `docs/product` directory. Tagged pages are read-only on the website; contribution links always target the editable source branch.

## Contributing

See [CONTRIBUTE.md](CONTRIBUTE.md) for writing conventions, validation requirements, and the pull-request checklist.

## License

The documentation in this repository is licensed under the [Creative Commons Attribution 4.0 International License](LICENSE) (CC BY 4.0). You may share and adapt it, including commercially, provided you give appropriate credit to BranchRunner, link to the license, and indicate whether you made changes.

The BranchRunner software is licensed separately; see the licensing information in the product repository.
