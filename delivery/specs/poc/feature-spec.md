# Stow Desktop POC Feature Specification

**Status:** First draft for review  
**Source:** `delivery/epics/00-canvas-markdown-interoperability.md` through `delivery/epics/04-tags-and-board-filtering.md`  
**Target users:** Individual Obsidian users; the project owner is the initial POC user

## 1. Goal and Expected Outcome

Stow is a personal Obsidian desktop plugin POC exploring whether specific workflows can be supported with a board of media-rich cards. The POC should let the project owner manage multiple boards, work with note-backed cards, attach content and media, and filter the active board by tags.

Each board is an Obsidian Canvas file. Each card is a native, resizable Canvas file card referencing a standalone Markdown note. The board remains usable in Obsidian without Stow, and filtering must not alter the saved Canvas layout or membership.

The first outcome is to validate this user-visible model and deliver its core workflows. This is a technical POC that can grow as a hobby project. Stow-managed files live inside the current Obsidian vault and use the vault's existing storage, synchronization, and sharing behavior; Stow does not provide separate storage or sharing settings.

## 2. Target Users and Scenarios

### Target user

Individual users of Obsidian.

### Primary scenarios

1. The user opens Stow from an Obsidian command or sidebar, finds or creates a board, and switches between boards while working with one active board.
2. The user creates a card, edits its content in Stow or directly in Obsidian, and later reopens it as a native Canvas card.
3. The user combines text, links, images, and animated GIFs in a card, using existing vault media or importing media from the file system.
4. The user adds freeform tags, reuses tags already present in the vault, and filters the active board to cards matching all selected tags.
5. The user removes a card with its explicit Remove action, deleting its Canvas node and Markdown note while retaining referenced media files; removing a media reference leaves the underlying media file intact.

## 3. Scope and Out of Scope

### In scope

- Obsidian desktop plugin access to multiple Canvas boards, one active at a time.
- Navigating a Canvas board and resizing native Canvas file cards.
- Creating, editing, and removing note-backed cards.
- Card notes that remain readable and editable in Obsidian without Stow.
- Mixed text, links, images, and animated GIFs in card notes.
- Adding media from the vault or file system and removing media references.
- Freeform card tags, reuse of vault tags, new tags, and active-board filtering.
- Safe handling of conflicting, renamed, missing, unsupported, or inaccessible files.
- Stow-managed board, card, and uploaded-media files stored in the vault under `/stow`, `/stow/cards`, and `/stow/files` respectively.
- Existing vault media referenced in place rather than duplicated.

### Out of scope for the initial POC

- Mobile support, collaboration features, and Stow-managed storage, synchronization, or sharing controls. Vault storage/sync/sharing remains governed by Obsidian's existing behavior.
- Full-text search and advanced theming.
- Global tag management, tag hierarchies, renaming tags across the vault, and saved filter presets.
- Audio/video support, media editing, remote asset hosting, and automatic cleanup of unused files.
- Multi-board filtering at the same time.

## 4. Functional Requirements

### FR-01: Open, create, and switch boards

Stow must let the user find and open a Canvas board, create a new board, and switch between multiple boards while presenting one active board at a time. On installation, Stow creates or reuses a `/stow` folder at the root of the current vault. Stow discovers its boards there; board files are stored directly in `/stow`, card notes in `/stow/cards`, and uploaded media in `/stow/files`.

**Acceptance criteria:**

- **AC-01.1** Given Stow is enabled in an Obsidian desktop vault, when the user opens the Stow board workflow, then the user can find and select an available board.
- **AC-01.2** Given `/stow` contains multiple Canvas files, when the user views the board list, then each Stow board is distinguishable by its name.
- **AC-01.3** Given the user selects a board, when it opens, then it becomes the only active Stow board and its saved content is shown.
- **AC-01.4** Given the user switches to a different board, when the switch completes, then the newly selected board is active and the previous board's saved content remains unchanged.
- **AC-01.5** Given the user creates a board at a path that already exists, when the create action is submitted, then the existing file is not silently overwritten.
- **AC-01.6** Given Stow is installed in a vault, when its initial setup completes, then `/stow` exists at the vault root and Stow does not overwrite existing files or folders.

### FR-02: Navigate and resize native Canvas cards

The active board must retain native Canvas behavior: the user can navigate across the board and resize its file cards, with layout changes saved to the board.

**Acceptance criteria:**

- **AC-02.1** Given an active board with cards outside the current viewport, when the user pans or scrolls the Canvas, then those cards can be brought into view.
- **AC-02.2** Given a native file card on the active board, when the user resizes it, then the card's new dimensions are visible and persist after the board is reopened.
- **AC-02.3** Given a board contains multiple cards, when one card is moved or resized, then the other cards retain their own saved positions and dimensions.

### FR-03: Create a note-backed card

Stow must create a standalone Markdown note in `/stow/cards` and a corresponding native Canvas file card for each new card.

**Acceptance criteria:**

- **AC-03.1** Given an active board, when the user creates a card with a title and initial text, then Stow creates a Markdown note in `/stow/cards` and a native Canvas file card that opens that note.
- **AC-03.2** Given the requested note path already exists, when the user creates a card, then Stow does not overwrite the existing note and presents a non-destructive resolution.
- **AC-03.3** Given a card is created, when the user opens its note directly in Obsidian with Stow unavailable, then its Markdown content is readable and editable.
- **AC-03.4** Given a new card is added to a board, when the board is reopened, then the card remains a native, resizable Canvas file card referencing the same note.

### FR-04: Edit cards in Stow and Obsidian

The user must be able to edit card content through Stow or directly in Obsidian and observe the latest saved content in both contexts.

**Acceptance criteria:**

- **AC-04.1** Given a card note, when the user edits supported content through Stow and saves, then the Markdown note contains the saved content.
- **AC-04.2** Given a card note is edited directly in Obsidian, when Stow refreshes or reopens the board, then the card reflects the latest saved note content.
- **AC-04.3** Given card content is changed, when the board is refreshed, then the associated Canvas card's position, dimensions, and identity remain unchanged.
- **AC-04.4** Given content is changed on one card, when the board is reopened, then unrelated cards and Canvas nodes retain their saved content and layout.

### FR-05: Explicitly remove a card

The user must be able to remove a card through an explicit Remove button action. Removing a card deletes its Canvas node and standalone Markdown note. Files referenced or uploaded for that card are retained.

**Acceptance criteria:**

- **AC-05.1** Given a card on the active board, when the user clicks its clearly labeled Remove button, then its Canvas node and associated Markdown note in `/stow/cards` are deleted.
- **AC-05.2** Given a card is removed, when the user checks the vault, then media files referenced by that card, whether uploaded through Stow or linked from elsewhere in the vault, remain available.
- **AC-05.3** Given a card is removed, when the board is reopened, then all unrelated nodes and their positions and dimensions remain unchanged.

### FR-06: Add and remove mixed card content and media

The user must be able to combine text, links, images, and animated GIFs in a card note, add media from the vault or file system, and explicitly remove a media reference. Existing vault media is referenced in place. Files uploaded through Stow are stored as separate files in `/stow/files` with their original names, and the card note contains an Obsidian-compatible link or embed reference; binary file contents are not written into the Markdown note. Removing a reference does not delete the underlying media file.

**Acceptance criteria:**

- **AC-06.1** Given a card note, when the user adds text or a link, then the content is saved in a form that remains readable and editable in Obsidian without Stow.
- **AC-06.2** Given an image or animated GIF already in the vault, when the user selects it for a card, then the note links to the existing file and Stow does not create a duplicate.
- **AC-06.3** Given a supported image or animated GIF selected from the file system, when the user uploads it through Stow, then Stow stores it as a separate file in `/stow/files` with its original name and adds a usable Obsidian-compatible link or embed reference to the card note.
- **OQ-02:** When a file is selected from outside the vault, should Stow preserve the external original and also copy it into `/stow/files`, or should it link directly to the external path? The draft assumes it creates a vault copy in `/stow/files` and links that copy because the upload directory was explicitly requested. **Owner:** [Project owner]
- **AC-06.4** Given an unsupported or inaccessible file, when the user attempts to add it, then Stow reports the problem and leaves the card note unchanged.
- **AC-06.5** Given a media reference on a card, when the user clicks its Remove button, then the reference is removed from the note and the underlying asset remains in the vault.
- **AC-06.6** Given content and media are mixed in one note, when the note is opened directly in Obsidian, then all supported content remains usable and the Canvas card still references that note.
- **AC-06.7** Given an upload would create a file whose name already exists in `/stow/files`, when the user submits it, then Stow reports a name collision and prompts the user to rename the new file without overwriting or automatically renaming either file.
- **AC-06.8** Given a card contains an image or animated GIF, when its native Canvas file card and note are viewed in Obsidian, then the image is visibly rendered in the card content and the animated GIF plays; this remains true when Stow is disabled.

### FR-07: Add and remove freeform card tags

The user must be able to add or remove freeform tags for a card, reuse tags found in the vault, and create new tags. Tag edits must remain readable and editable in Obsidian.

**Acceptance criteria:**

- **AC-07.1** Given a card note with tags, when the user opens its tag controls, then Stow displays the tags currently associated with that card.
- **AC-07.2** Given tags already exist in the vault, when the user edits card tags, then the user can select an existing tag or enter a new freeform tag.
- **AC-07.3** Given the user adds or removes a tag, when the change is saved, then only that card's tag data changes in its Markdown note.
- **AC-07.4** Given an empty or duplicate tag value is submitted, when Stow validates it, then malformed or duplicate tag data is not added.

### FR-08: Filter the active board by tags

The user must be able to select one or more tags and filter the active board so that cards matching **all** selected tags remain visible and nonmatching cards are hidden. Filtering is a view operation and must not change saved board membership or layout.

**Acceptance criteria:**

- **AC-08.1** Given cards with different tag combinations, when the user selects one tag, then only cards containing that tag are visible.
- **AC-08.2** Given the user selects multiple tags, when filtering is applied, then only cards containing every selected tag are visible.
- **AC-08.3** Given a filter is active, when the user clears it, then all cards return to view with their prior positions, dimensions, and board membership.
- **AC-08.4** Given a filter is active, when the user changes the active board, then the filter does not alter either board's saved content.
- **AC-08.5** Given a card's tags change in Obsidian, when Stow refreshes the board, then filter results use the latest saved tags.
- **AC-08.6** Given no cards match the selected tags, when the filter is applied, then Stow indicates that no cards match and provides a way to clear the filter.

### FR-09: Handle renamed or missing files safely

Stow must report unresolved note and media references and must not silently substitute, overwrite, or delete other vault files.

**Acceptance criteria:**

- **AC-09.1** Given a referenced note or media file is renamed or missing, when Stow next opens or refreshes the board, then the unresolved item is identifiable to the user.
- **AC-09.2** Given an unresolved reference, when Stow reports it, then unrelated notes, assets, and Canvas nodes remain unchanged.
- **AC-09.3** Given a user action would overwrite or delete an existing note or source asset, when the action is submitted, then Stow requires an explicit user action and does not perform a silent destructive change.

### FR-10: Use Obsidian vault storage and sharing behavior

Stow-managed boards, card notes, and uploaded files must be normal files in the current Obsidian vault. Stow must rely on the vault's existing storage, synchronization, and sharing behavior and must not provide separate controls for these options.

**Acceptance criteria:**

- **AC-10.1** Given a board is created, when the user views the vault in Obsidian, then its `.canvas` file is stored directly in `/stow`.
- **AC-10.2** Given a card is created, when the user views the vault in Obsidian, then its Markdown note is stored in `/stow/cards`.
- **AC-10.3** Given a file is uploaded through Stow, when the upload succeeds, then the file is stored in `/stow/files` with its original name and referenced from the card note.
- **AC-10.4** Given Stow is open, when the user looks for storage, synchronization, or sharing settings, then Stow provides no separate settings and the files remain subject to the current Obsidian vault's behavior.

## 5. Non-Functional Requirements

### NFR-01: Board and filter responsiveness (provisional)

- On the agreed reference desktop, a test vault containing 100 card notes (each with a title, up to 200 characters of text, and up to three tags) must present a usable active board within **3 seconds** of selecting the board.
- On the same fixture, applying or clearing a tag filter must update visible-card results within **500 milliseconds**.
- The reference desktop and exact timing start/end events must be recorded before performance results are treated as release evidence. These thresholds are proposed for review.

### NFR-02: Unit function coverage

- Unit tests must achieve **100% function coverage** for in-scope, unit-testable production logic.
- The test report must also include statement/line and branch coverage percentages. The coverage tool and any exclusions must be documented; exclusions must not be used to hide untested in-scope behavior.

### NFR-03: Data integrity

- Across the acceptance suite, there must be **zero silent overwrites or unintended deletions** of existing notes, media assets, unrelated Canvas nodes, or board layout.
- Card removal explicitly deletes the selected card note and Canvas node but retains linked/uploaded media. The Remove control must make the card being removed identifiable.
- Upload name collisions must never overwrite a file; Stow must report the conflict and ask the user to rename the new upload.

### NFR-04: Desktop compatibility

- The POC must pass **100% of the agreed smoke-test cases** on the latest macOS and Windows releases and the latest Ubuntu release available at validation time.
- The exact OS and Obsidian versions used for each validation run must be recorded.

## 6. Acceptance Criteria Traceability

Every functional requirement above includes its own Given/When/Then acceptance criteria:

| Requirement | Acceptance criteria |
| --- | --- |
| FR-01: Open, create, and switch boards | AC-01.1 to AC-01.6 |
| FR-02: Navigate and resize cards | AC-02.1 to AC-02.3 |
| FR-03: Create note-backed card | AC-03.1 to AC-03.4 |
| FR-04: Edit cards in Stow and Obsidian | AC-04.1 to AC-04.4 |
| FR-05: Remove a card | AC-05.1 to AC-05.3 |
| FR-06: Mixed content and media | AC-06.1 to AC-06.8 |
| FR-07: Card tags | AC-07.1 to AC-07.4 |
| FR-08: Tag filtering | AC-08.1 to AC-08.6 |
| FR-09: Missing/renamed files | AC-09.1 to AC-09.3 |
| FR-10: Vault-managed storage and sharing | AC-10.1 to AC-10.4 |

## 7. Assumptions and Open Questions

Each remaining item is assigned to the project owner for confirmation.

### Assumptions for this draft

- **A-01:** The POC targets individual Obsidian desktop users, with the project owner as the initial user. **Owner:** [Project owner]
- **A-02:** Stow creates or reuses `/stow` at the vault root on installation; Canvas boards are stored directly in `/stow`, card notes in `/stow/cards`, and uploaded files in `/stow/files`. **Owner:** [Project owner]
- **A-03:** A card is a native, resizable Canvas file card referencing a standalone Markdown note; the note remains usable without Stow. **Owner:** [Project owner]
- **A-04:** Obsidian Vault's existing storage, synchronization, and sharing behavior applies; Stow provides no separate management for these options. **Owner:** [Project owner]
- **A-05:** Removing a card deletes its Canvas node and Markdown note, but retains any media files it referenced. **Owner:** [Project owner]
- **A-06:** Existing vault media is linked in place; files uploaded through Stow are stored in `/stow/files` with original names, and the card note references them. **Owner:** [Project owner]
- **A-07:** Tags are freeform and reusable from the vault; filtering multiple selected tags matches all selected tags and hides nonmatches. **Owner:** [Project owner]
- **A-08:** Stow is accessed through an Obsidian command and sidebar. **Owner:** [Project owner]
- **A-09:** Search, advanced theming, and mobile support are deferred. **Owner:** [Project owner]
- **A-10:** Unit function coverage is 100% for in-scope, unit-testable production logic. **Owner:** [Project owner]
- **A-11:** Supported validation platforms are the latest macOS, Windows, and Ubuntu releases available at test time. **Owner:** [Project owner]
- **A-12:** The 100-card/3-second and 500-millisecond filter targets are provisional until evaluated on an agreed reference desktop. **Owner:** [Project owner]

### Open questions

- **OQ-01:** What filename should Stow use for a new card note, and what should happen when that card-note name already exists in `/stow/cards`? **Owner:** [Project owner]
- **OQ-02:** Which specific computer should be the reference desktop for measuring the provisional 3-second board-open and 500-millisecond filter targets? For example, identify a machine you use or authorize a current desktop as the baseline. **Owner:** [Project owner]
- **OQ-03:** Which Obsidian desktop version should be tested on the latest macOS, Windows, and Ubuntu releases? **Owner:** [Project owner]