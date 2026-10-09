# Stow POC Architecture

**Status:** Draft for technical review  
**Source of truth:** [Feature specification](./feature-spec.md)

## Purpose

Describe the system context, major responsibilities, data flow, and key decisions for the Stow Obsidian desktop plugin POC. The architecture centers on native Canvas boards and standalone Markdown card notes stored in the current vault. It is an architecture overview, not an API design, database schema, or implementation plan.

## System context

Stow runs as a plugin within Obsidian desktop and is used by an individual vault user. Obsidian provides the plugin runtime, workspace, Canvas experience, and vault file operations. Stow provides the workflows for discovering and switching boards, managing note-backed cards and media references, editing tags, and filtering the active board.

Stow-managed content is stored as normal files in the current vault:

- Canvas boards directly in `/stow`.
- Standalone Markdown card notes in `/stow/cards`.
- Media imported through Stow in `/stow/files`.

Existing vault media is referenced in place. Obsidian's existing storage, synchronization, and sharing behavior applies; Stow does not provide separate controls for these.

```text
User
  |
Obsidian desktop
  |-- Stow plugin workflows and controls
  |-- Native Canvas board and file-card experience
  |-- Vault files
      |-- /stow/*.canvas
      |-- /stow/cards/*.md
      |-- /stow/files/*
```

## Major components

### Obsidian host

- Hosts and activates the Stow plugin in the desktop application.
- Provides the workspace and native Canvas behavior, including pan, scroll, and resizing of file cards.
- Owns the vault's file operations and existing storage, synchronization, and sharing behavior.

### Stow plugin workflows

- Provides entry points through an Obsidian command and sidebar.
- Discovers and presents boards under `/stow`, and maintains one active Stow board at a time.
- Coordinates board and card operations with the vault and Canvas.
- Presents errors and unresolved references without silently substituting, overwriting, or deleting unrelated files.

### Board and Canvas integration

- Represents each board as a native Obsidian Canvas file stored directly in `/stow`.
- Adds and removes native Canvas file-card nodes that reference standalone Markdown notes.
- Preserves each node's identity and saved layout when card content changes; board layout changes are saved as Canvas changes.
- Applies tag filtering as a view operation, leaving saved Canvas membership and layout unchanged.

### Card content and tags

- Represents each card as a standalone Markdown note in `/stow/cards`, readable and editable in Obsidian without Stow.
- Supports the specified text and links, image and animated GIF references, and freeform tags.
- Keeps tag changes local to the relevant card note; provides access to existing vault tags for reuse.

### Media handling

- References existing vault media in place.
- Stores imported supported media as separate files in `/stow/files` and references those files from card notes.
- Removes a media reference without deleting its underlying asset; retains referenced or uploaded media when a card is removed.

## Component responsibilities and boundaries

| Concern | Responsibility | Boundary |
| --- | --- | --- |
| Runtime and workspace | Obsidian | Hosts the plugin and owns the desktop workspace. |
| Canvas rendering and saved layout | Obsidian Canvas, coordinated by Stow | Boards remain native `.canvas` files; Stow filtering must not change saved membership or layout. |
| Board and card workflows | Stow plugin | Coordinates user actions and vault changes; does not replace the host's storage or synchronization behavior. |
| Durable board state | Canvas files in the vault | Board membership and layout are represented by native Canvas data. |
| Durable card content and tags | Markdown notes in the vault | Card text, links, media references, and tags remain readable and editable in Obsidian. |
| Media bytes | Vault files | Existing assets are referenced in place; imported assets live in `/stow/files`; removal of references does not remove assets. |
| Tag filtering | Stow's active-board view | All selected tags must match; filtering changes visibility only. |

## Data flow

1. **Discover and open boards:** Stow finds Canvas files under `/stow`, presents them by name, and opens the selected board as the active board using Obsidian's Canvas experience.
2. **Create a board:** Stow creates a Canvas file under `/stow` without overwriting an existing file or folder. Board data remains in the vault.
3. **Create or edit a card:** Stow creates or updates a standalone Markdown note in `/stow/cards` and associates it with a native Canvas file-card node. Edits made directly in Obsidian are reflected when Stow refreshes or reopens the board. Content edits do not change unrelated nodes or layout.
4. **Add media:** Stow validates that selected media is supported and accessible. Existing vault assets are linked in place; supported files selected from the file system are copied to `/stow/files` and referenced from the card note. A filename conflict is surfaced without overwriting either file.
5. **Edit tags and filter:** Stow saves a card's tag changes in that card's note. It filters the active board using an all-selected-tags match. Clearing or changing boards does not persist filtering as a Canvas membership or layout change.
6. **Remove content:** An explicit card Remove action deletes that card's Canvas node and Markdown note while retaining its media. Removing a media reference changes the note only; the referenced file remains.
7. **Refresh references:** On opening or refreshing a board, Stow identifies missing or renamed note and media references. It does not silently substitute files or modify unrelated content.

## Cross-cutting requirements

- **Data integrity:** No silent overwrites or unintended deletion of existing notes, media, unrelated Canvas nodes, or layout. Conflicts and unsupported or inaccessible files are reported. Card removal is explicit and identifies the card being removed.
- **Obsidian interoperability:** Boards remain native Canvas files; card notes remain readable and editable Markdown files; supported images and animated GIFs remain usable in Obsidian when Stow is disabled.
- **Vault ownership:** Stow-managed files remain subject to the current vault's storage, synchronization, and sharing behavior. Stow adds no separate settings for these.
- **Performance observations:** For the specified 100-card fixture, record whether a board becomes usable within 3 seconds and whether applying or clearing a tag filter completes within 500 milliseconds. These are exploratory targets, not release gates; record the machine, OS, Obsidian version, fixture, and timing events.
- **Test quality:** In-scope, unit-testable production logic targets 100% function coverage. Also report statement/line and branch coverage, documenting the coverage tool and exclusions.
- **Desktop compatibility:** Validate all agreed smoke-test cases on the latest macOS, Windows, and Ubuntu releases and the latest Obsidian desktop version available at validation time. Record the versions used.

## Key architecture decisions

| ID | Decision | Rationale and traceability |
| --- | --- | --- |
| D-01 | Implement Stow as an Obsidian desktop plugin, using Obsidian's workspace and Canvas experience. | Matches the product context, access model, and native board/card behavior in FR-01–FR-04; A-01, A-03, and A-08. |
| D-02 | Use normal vault files as the durable source of board, card, and media content; do not introduce a separate application database or storage service. | Matches FR-03, FR-06, and FR-10; AC-03.3, AC-06.6, AC-10.1–10.4; A-02 and A-04. |
| D-03 | Store boards, card notes, and imported media under `/stow`, `/stow/cards`, and `/stow/files`, respectively. | Matches FR-01, FR-03, and FR-06; AC-01.6, AC-03.1, and AC-06.3; A-02. |
| D-04 | Keep filtering separate from persisted Canvas board membership and layout. | Matches FR-08 and AC-08.2–08.4; preserves the saved layout guarantees in FR-02. |
| D-05 | Never silently overwrite files; retain media when removing cards or references. | Matches FR-05, FR-06, and FR-09; NFR-03 and A-05–A-06. |

## Assumptions and decisions for human review

- **H-01 — Card filename collision:** The feature spec leaves unresolved what to do when a generated timestamp-based note filename already exists (OQ-01; A-13). Confirm the non-destructive user resolution.
- **H-02 — External media import:** The feature spec draft assumes copying external media into `/stow/files` while leaving the original intact (OQ-02; AC-06.3). Confirm this assumption.
- **H-03 — Obsidian compatibility baseline:** Confirm the Obsidian desktop version available for validation and record the version used, consistent with NFR-04 and A-11.

## Traceability

- Board discovery, creation, and switching: FR-01; AC-01.1–01.6.
- Native Canvas navigation, resizing, and layout: FR-02; AC-02.1–02.3.
- Note-backed card creation and editing: FR-03–FR-04; AC-03.1–03.4 and AC-04.1–04.4.
- Explicit removal and media handling: FR-05–FR-06; AC-05.1–05.3 and AC-06.1–06.8.
- Freeform tags and all-selected-tags filtering: FR-07–FR-08; AC-07.1–07.4 and AC-08.1–08.6.
- Missing/renamed files and safe handling: FR-09; AC-09.1–09.3.
- Vault storage and sharing behavior: FR-10; AC-10.1–10.4.
- Performance, coverage, integrity, and desktop compatibility: NFR-01–NFR-04.
