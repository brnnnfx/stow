# Epic 00: Canvas and Markdown Interoperability

**Status:** Proposed technical spike  
**Outcome:** Confirm that native, resizable Obsidian Canvas file cards backed by standalone Markdown notes can support Stow's POC without corrupting board layout or making card content dependent on Stow.

## Scope

- Validate `.canvas` boards that reference Markdown card notes.
- Validate note edits and Canvas layout preservation across Obsidian and Stow workflows.
- Investigate how tag filtering could display a subset of native Canvas cards without changing the saved board.
- Record supported integration options, limitations, and a proceed/revise recommendation.

Out of scope: a polished plugin UI, production board management, broad card CRUD, mobile support, search, and theming.

## Dependencies

None. This epic should complete before committing to the plugin's Canvas integration approach.

## Units of Work

### 00.1 Validate note-backed Canvas cards

**Outcome:** A regular Canvas board displays a standalone Markdown note as a native, resizable file card.

**Acceptance criteria:**

- **Given** a `.canvas` file referencing a Markdown note, **when** the board is opened in Obsidian, **then** the note appears as a native Canvas file card and can be opened and edited in Obsidian.
- **Given** the note contains text, links, tags, and image embeds, **when** it is viewed through Canvas and opened directly, **then** the content remains usable without Stow, images render visibly in the card and note view, and animated GIFs play.

### 00.2 Validate safe content updates

**Outcome:** Card-note changes do not damage board geometry or unrelated Canvas content.

**Acceptance criteria:**

- **Given** a board with multiple nodes and a note-backed card, **when** the card note's content changes, **then** the card's position, dimensions, identity, and unrelated nodes remain unchanged.
- **Given** the note is edited directly in Obsidian, **when** the Canvas board is refreshed or reopened, **then** the card reflects the saved note content without requiring Stow to rewrite the board.
- **Given** a note is renamed or missing, **when** the board is reopened, **then** the observed behavior is documented and no other note or node is silently deleted or replaced.

### 00.3 Validate tag-filter feasibility

**Outcome:** Determine whether the requested tag filter can be applied to the active native Canvas board without destructive changes to the saved board.

**Acceptance criteria:**

- **Given** a board with cards carrying different tag combinations, **when** a filter with one or more tags is applied in the experiment, **then** only cards containing all selected tags remain visible and nonmatching cards are hidden; clearing the filter restores all cards.
- **When** the filter is applied and cleared, **then** the saved `.canvas` layout and card membership are unchanged unless the user explicitly edits the board.
- The spike records whether native Canvas behavior, a supported plugin extension point, or another approach is needed; it does not assume internal Canvas APIs are stable.

### 00.4 Record findings and decision

**Outcome:** Later epics can rely on an explicit interoperability decision rather than an unverified assumption.

**Acceptance criteria:**

- Findings include the tested Obsidian version, steps, observed behavior, layout-preservation results, and known limitations.
- The recommendation says whether to proceed with the note-backed Canvas model, revise it, or investigate a constrained alternative.
- Any behavior that could risk data loss is listed as a blocker until resolved.

## Unit Tests

This is primarily a manual spike. If reusable Canvas or Markdown transformation code is produced, add unit tests for parsing and serializing representative Canvas fixtures, resolving file-card references, and preserving node geometry and unrelated data during updates. Prefer Jest if it fits the eventual TypeScript setup; target 100% function coverage for in-scope, unit-testable logic and report statement/line and branch coverage separately. Do not create tests solely to inflate coverage for discarded spike code.

## Manual Validation

1. Create a Canvas board with at least two file cards and one unrelated node; resize and reposition them.
2. Open a referenced note directly, edit its text, tag, and image embed, then reopen the board and verify the card still points to the note.
3. Change the note through the spike/plugin path and confirm all Canvas positions, dimensions, and unrelated nodes are preserved.
4. Try a tag filter and clear it; inspect that the underlying board file was not altered by filtering.
5. Rename and temporarily remove a referenced note, reopen the board, and record behavior without deleting other files.

## Open Decisions

- Record the latest Obsidian desktop version and latest macOS, Windows, and Ubuntu releases used for the spike; older Obsidian versions are out of scope.
- What supported integration surface can Stow use to open and update Canvas files?
- Can tag filtering meet the POC goal while retaining the native Canvas view and saved layout?