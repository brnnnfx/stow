# Stow POC Architecture

**Status:** Draft for technical review  
**Source of truth:** [Feature specification](./feature-spec.md)

## Purpose

This document describes a simple local architecture for the Stow POC using a React frontend, a Node.js backend, SQLite, and files in a local application folder. It covers the board and card workflows, media references, tags and filtering, and simulated PDF/DOCX reading. It is an architecture overview, not an API specification, database schema, or implementation plan.

## System context

The POC is a single-user application running on the user's computer. The user works with boards and note-backed cards through the React UI. A local Node.js process coordinates application behavior and is the only component that reads and writes SQLite or the application's files.

The application has no authentication, roles, remote services, or shared server. Its durable state is kept locally:

- SQLite stores application metadata and board/card relationships.
- The application's data folder stores board/card content and media files.
- PDF and DOCX reading is simulated; the application does not parse document contents.

The feature specification describes an Obsidian desktop plugin, native Canvas files, and vault-managed storage. The requested runtime and storage constraints instead describe a standalone local application. Consequently, this architecture preserves the core board/card workflows but does not assume an Obsidian runtime or promise native Canvas/vault interoperability. This is a material scope decision that requires review (see [Assumptions and decisions for human review](#assumptions-and-decisions-for-human-review)).

## Major components

```text
User
  |
React frontend
  | local application boundary
Node.js backend
  |                     |
SQLite metadata      Local application folder
                      (board/card files and media)
```

### React frontend

- Presents the board list, one active board, card editing, media selection, tag controls, and tag filters.
- Renders the board and its cards, including navigation and resizing.
- Shows unresolved file references, unsupported media, name conflicts, and other operation errors.
- Keeps transient UI state, such as the current filter selection, in the frontend; it does not persist filtering as a board change.
- Presents a simulated PDF/DOCX reading experience without claiming to extract or understand real document contents.

### Node.js backend

- Owns application behavior and is the boundary between the UI, SQLite, and local files.
- Loads and saves boards and cards, applies tag matching, and coordinates media references.
- Validates operations that could overwrite or delete data. It reports conflicts instead of silently replacing files.
- Simulates document-reading results; it does not invoke a real PDF/DOCX parser or external intelligence service.
- Does not expose or depend on a separately deployed remote service.

### SQLite persistence

- Stores structured application metadata, including board/card identity, relationships, tags, and saved board layout.
- Is accessed only by the Node.js backend.
- Does not store binary media or replace the user-readable card content files.
- Is used for local persistence only; no synchronization or multi-user coordination is provided.

### Local file storage

- Stores user-readable board/card content and media beneath an application-managed data folder.
- Uses separate locations for boards, card notes, and uploaded media, following the feature spec's `/stow`, `/stow/cards`, and `/stow/files` organization where applicable.
- Keeps media separate from note content. Existing local media may be referenced in place; imported media is copied into the application's media location and referenced from the card.
- Retains media when a card or media reference is removed. Removing a card removes its associated card content and board membership only after an explicit user action.
- Does not manage backups, synchronization, or sharing.

## Component responsibilities and boundaries

| Concern | Responsible component | Boundary |
| --- | --- | --- |
| User workflows and presentation | React frontend | Does not access the database or write application files directly. |
| Validation and operation coordination | Node.js backend | Owns decisions about persistence and destructive changes; returns visible errors rather than silently falling back. |
| Structured metadata and saved layout | SQLite via backend | Not a public interface and not a detailed schema contract. |
| Board/card content and media bytes | Local file storage via backend | Files remain distinct from metadata; referenced media is not deleted as a side effect of removing a card. |
| Tag filtering | Frontend presentation backed by current board/card data | All selected tags must match; filtering changes visibility only, not saved membership or layout. |
| PDF/DOCX reading | Simulated reader behavior in the application | No real format parsing, extraction, indexing, or document intelligence. |

## Data flow

1. **Start and select a board:** The frontend requests the available boards from the backend. The backend reads board metadata and associated content, then returns the selected board's current state. Only one board is active in the UI at a time.
2. **Create or edit a card:** The frontend submits the user's changes to the backend. The backend validates the target and any generated filenames, saves the card content and metadata, and updates the board/card relationship. Existing files are not silently overwritten.
3. **Add media:** The backend checks that the selected file is supported and accessible. Existing local media is referenced in place. An imported file is copied into the application's media folder; a same-name conflict is reported and requires the user to rename the new file. The card content stores a reference, not the binary data.
4. **Edit tags and filter:** Tag changes are saved for the selected card. Filtering applies an all-selected-tags match to the active board's current card data. Clearing the filter restores visibility without writing a board change.
5. **Remove a card or media reference:** The user explicitly identifies and confirms the removal action. The backend removes the card's board membership and associated note as applicable, but retains media files. Removing a media reference changes the card content only.
6. **Simulated document reading:** The user opens a PDF or DOCX item in the UI. The application presents simulated reading behavior and does not claim to read or extract the actual document.
7. **Refresh after external file changes:** When the application reloads or refreshes local state, missing or renamed references are surfaced to the user. The backend does not substitute another file or alter unrelated content.

## Cross-cutting requirements

- **Data integrity:** Prevent silent overwrites and unintended deletion of existing files, unrelated cards, and saved layout. Name conflicts and inaccessible or unsupported files must be explicitly reported. Card removal is the deliberate exception for the selected card note and its board membership; its media remains.
- **Readable content:** Keep card content in a user-readable file format and media in separate files, consistent with the feature spec's note-and-reference model. Exact file-format compatibility with Obsidian is not guaranteed by this standalone architecture.
- **Local-only operation:** No login, roles, remote persistence, object storage, or separate synchronization/sharing controls.
- **Performance observations:** Retain the feature spec's exploratory targets: board usable within 3 seconds for the 100-card fixture, and applying or clearing a tag filter within 500 milliseconds. These are observations, not release gates; record the machine, OS, fixture, and timings.
- **Test quality:** Unit-testable in-scope production logic targets 100% function coverage, with statement/line and branch coverage also reported. Document the coverage tool and exclusions.
- **Desktop validation:** Smoke-test the agreed cases on the latest macOS, Windows, and Ubuntu releases and the latest Obsidian desktop version if Obsidian interoperability remains in scope. Record the actual environment versions used.

## Key architecture decisions

| ID | Decision | Rationale and traceability |
| --- | --- | --- |
| D-01 | Use React for the user interface and Node.js for local application behavior. | Required MVP constraints. Supports the board/card workflows in FR-01 through FR-09 without adding a remote service. |
| D-02 | Use SQLite for structured metadata and keep user-facing content and media as local files. | Required MVP constraints; follows the separation between standalone card notes and media in FR-03, FR-06, and FR-10, while replacing vault storage. |
| D-03 | Keep application data in a local application-managed folder, organized into board, card, and media locations. | Required MVP constraint; aligns conceptually with A-02 and AC-10.1–10.3. Exact path and file formats need confirmation because the source spec places these files in an Obsidian vault. |
| D-04 | Treat board layout and tag filtering as separate: saved layout is durable, while filtering is a view operation. | Preserves FR-02 and FR-08, particularly AC-02.2–02.3 and AC-08.2–08.4. |
| D-05 | Never silently overwrite files; retain media when removing cards or references. | Preserves FR-05, FR-06, and FR-09, and NFR-03. |
| D-06 | Simulate PDF/DOCX reading rather than implement document parsing. | Required MVP constraint. Real document reading is not specified in the source feature specification and is intentionally not inferred from its media requirements. |
| D-07 | Do not add authentication or role-based access control. | Required MVP constraint, consistent with the individual-user assumption A-01. |

## Assumptions and decisions for human review

- **H-01 — Product boundary:** Confirm that the requested standalone React/Node application supersedes the feature spec's Obsidian desktop plugin context (A-01, A-08). Without an Obsidian runtime, the architecture cannot provide the spec's command/sidebar integration.
- **H-02 — Canvas interoperability:** Confirm whether native `.canvas` files and native, resizable Obsidian file cards are still required (FR-01, FR-02, FR-03; A-03). The proposed standalone rendering and SQLite-backed layout do not, by themselves, satisfy native Canvas interoperability.
- **H-03 — Storage and file compatibility:** Confirm that local application-folder storage replaces files in the current vault, including the `/stow`, `/stow/cards`, and `/stow/files` locations (FR-10; A-02, A-04). Also confirm whether Markdown notes must remain directly editable in Obsidian without Stow (AC-03.3, A-03).
- **H-04 — Simulated reading scope:** Define what users should see when they use simulated PDF/DOCX reading. The source feature spec does not define document-reading behavior, so this architecture makes no assumption about simulated output or interaction.
- **H-05 — Generated card filename collision:** The feature spec leaves the response to a timestamp-based note filename collision open (OQ-01; A-13). Confirm a non-destructive resolution before finalizing the persistence behavior.
- **H-06 — External media import:** The feature spec draft assumes importing a copy into the application folder while leaving the external original intact (OQ-02; AC-06.3). Confirm that assumption for this local application.
- **H-07 — Data-folder location and recovery:** Confirm the exact application data-folder location and whether users need export, backup, or recovery behavior. These are not specified in the feature spec or MVP constraints.
- **H-08 — Platform compatibility scope:** The feature spec names latest desktop Obsidian alongside macOS, Windows, and Ubuntu (NFR-04, A-11). Confirm whether Obsidian compatibility still applies to the standalone app, or whether validation should cover only those desktop operating systems.

## Traceability

The feature specification remains the source of product behavior. Relevant requirement groups are:

- Board selection, creation, and switching: FR-01; AC-01.1–01.6.
- Board navigation, resizing, and saved layout: FR-02; AC-02.1–02.3.
- Note-backed card creation and editing: FR-03–FR-04; AC-03.1–03.4 and AC-04.1–04.4.
- Explicit removal and media handling: FR-05–FR-06; AC-05.1–05.3 and AC-06.1–06.8.
- Freeform tags and all-selected-tags filtering: FR-07–FR-08; AC-07.1–07.4 and AC-08.1–08.6.
- Missing/renamed files and safe operations: FR-09; AC-09.1–09.3.
- Local storage divergence: FR-10; AC-10.1–10.4, subject to human review under H-03.
- Performance, coverage, integrity, and desktop support: NFR-01–NFR-04.
