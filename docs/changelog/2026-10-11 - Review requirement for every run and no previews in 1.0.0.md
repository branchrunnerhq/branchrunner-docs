# Review requirement for every run and no previews in 1.0.0

The product pages now say that every 1.0.0 run needs a reviewer from a second
provider and a human technical review, and that previews are not available in
1.0.0 (owner decisions D10 and D11, 2026-10-11).

## Added

- **Review requirements for every run.** A new section in validation and
  approvals explains that 1.0.0 reviews every change as Critical, so every run
  needs a reviewer agent from a different provider and model family than every
  Implement step and a human technical review before publication. The product's
  workflow editor and run preflight link to this section.

## Changed

- **Agent providers, workflows, tasks and runs.** Each page states the
  requirement where review or run start is described and links to the new
  section. Agent providers no longer implies that medium- and low-risk work is
  reviewed without a second provider in 1.0.0.
- **Release status.** The known limitation "High-risk review needs another
  provider" is replaced by "Every run needs two providers and a human
  reviewer", and a new limitation states that previews are not available in
  1.0.0.
- **Data retention, deletion and custody.** The removal actions no longer
  mention preview identities or preview links behind an optional edge; a note
  states that a 1.0.0 installation has none to revoke.
