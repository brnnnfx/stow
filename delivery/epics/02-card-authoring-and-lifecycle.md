# Epic 02: Card Authoring and Lifecycle

**Status:** Proposed  
**Outcome:** The user can create, edit, and remove note-backed cards on the active board while card notes remain ordinary Markdown files and Canvas layout is preserved.

## Scope

- Create a standalone Markdown note in `/stow/cards` and corresponding native Canvas file card.
- Edit card content through Stow and through Obsidian.
- Remove a card through an explicit Remove button; delete its Canvas node and Markdown note while retaining referenced media files.

Out of scope: media import policy, tag filtering, search, bulk operations, and automatic cleanup of unreferenced notes.

## Dependencies

Depends on Epic 00's note-backed card validation and Epic 01's active-board workflow.

## Units of Work

### 02.1 Create a card

**Outcome:** A new card is available on the active board and has independently editable Markdown content.

**Acceptance criteria:**

- **Given** an active board, **when** the user creates a card with a title and initial text, **then** Stow creates a standalone Markdown note in `/stow/cards` and a native Canvas file card that references it.
- **Given** a note path already exists, **when** the user creates a card with a conflicting title/path, **then** Stow does not overwrite the existing note and offers a non-destructive resolution.
- **When** the board is reopened in Obsidian, **then** the card remains a normal resizable Canvas file card and opens the correct note.

### 02.2 Edit card content

**Outcome:** Card content remains consistent whether edited in Stow or directly in Obsidian.

**Acceptance criteria:**

- **Given** a card note, **when** the user edits its supported text content in Stow, **then** the Markdown note is updated and remains readable in Obsidian.
- **Given** the note is edited in Obsidian, **when** Stow refreshes or reopens the board, **then** the latest saved note content is shown without requiring a duplicate card.
- **When** Stow updates note content, **then** the Canvas node's position, dimensions, identity, and unrelated nodes are preserved.

### 02.3 Remove a card from a board

**Outcome:** The user can explicitly remove a card and its note without deleting media files referenced by the card.

**Acceptance criteria:**

- **Given** a card on the active board, **when** the user clicks its Remove button, **then** its Canvas node and associated Markdown note in `/stow/cards` are deleted.
- **Given** a removed card referenced uploaded or linked media, **when** the user checks the vault, **then** those media files remain available.
- **When** a card is removed, **then** all other Canvas nodes and layout remain unchanged.

## Unit Tests

Add unit tests for `/stow/cards` note creation/content updates, safe path conflict handling, Canvas node creation/removal, media retention, and preservation of unrelated Canvas data and geometry. Include cases for missing notes and renamed references based on Epic 00 findings. Target 100% function coverage for in-scope, unit-testable card/Canvas logic; report statement/line and branch coverage separately and use Jest if compatible with the chosen setup.

## Manual Validation

1. Create a card and verify both the `.md` note and Canvas file card exist.
2. Open and edit the note directly in Obsidian; verify the card reflects the change after refresh/reopen.
3. Edit the card in Stow; verify the standalone Markdown remains valid and Canvas geometry is unchanged.
4. Click Remove and verify the Canvas node and card note are deleted while linked and uploaded media files remain.
5. Test a title/path conflict and verify no existing note is overwritten.

## Open Decisions

- Card note folder and filename rules.
- Which Markdown fields are part of the initial card editor versus edited only in Obsidian.
- Conflict behavior if a note is edited in both places before either view refreshes.
- Card-note filename rules and behavior if the filename already exists in `/stow/cards`.