# Jarvis

A self-hosted voice assistant. [SPEC.md](SPEC.md) is the design.
[FEATURES.md](FEATURES.md) is the map of what is being built. The work itself
is tracked in GitHub issues on `magna-nz/jarvis`.

## How work is tracked

- **Parent issues** are components (`component` label, C1–C9) and features (`feature` label, F1–F11). Each has a "done when" line.
- **Sub-issues** (`task` label) are the sub-work under a parent.
- **Milestones** are slices. One slice is built at a time, in milestone order.
- `needs-puck`, `needs-speaker` and `blocked-question` mark work that cannot be finished yet.

Sub-work lives only in the issues. Do not copy checklists into `FEATURES.md`.

## "Work on the next item"

1. Find the current slice: the lowest-numbered milestone with open issues.
2. Find the next parent: the lowest-numbered open `component` or `feature` issue that has open sub-issues in that slice.
3. The item is all of that parent's open sub-issues in the current slice. Skip any labelled `needs-puck`, `needs-speaker` or `blocked-question`, and say which were skipped.
4. Read the parent, its sub-issues and the matching section of `SPEC.md` before planning.
5. State the plan and wait for approval before starting.

If asked for a single issue by number, do only that one.

Useful commands:

```bash
gh api repos/magna-nz/jarvis/milestones --jq '.[] | "\(.number) \(.title): \(.open_issues) open"'
gh issue list --milestone "Slice 1: Typed end to end" --label task --state open --limit 100
gh api repos/magna-nz/jarvis/issues/<parent>/sub_issues --jq '.[] | "\(.number) \(.state) \(.title)"'
```

## Finishing an item

- One branch and one pull request per item. Never commit to `main`.
- The pull request body lists `Closes #N` for every sub-issue it completes, one per line.
- Do not close a parent in the pull request. A parent is closed when all its sub-issues are closed **and** its "done when" line has been checked and is true.
- When a parent closes, tick its box in `FEATURES.md`.
- If the work changes the design, update `SPEC.md` in the same pull request.
- If new sub-work turns up, create a sub-issue under the right parent with the `task` label and the slice's milestone, rather than doing it unrecorded.

## Planning later slices

Slices 3 onwards have parent issues but no sub-issues. Break a slice into
sub-issues when it becomes the next one, not earlier.

## Build and test

Nothing is built yet. Add the commands here when slice 1 creates them.

## Docs site

`docs/` is served by GitHub Pages from `main`. It is hand-written HTML with no
build step. Diagrams are standalone SVG files with a light card background so
they read in both themes; sequence diagrams are Mermaid, inline.
