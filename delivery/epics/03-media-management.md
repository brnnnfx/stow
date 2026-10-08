# Epic 03: Media Management

**Status:** Proposed  
**Outcome:** The user can add and remove supported media references on a card using existing vault files or files selected from the file system, while keeping the card note usable in Obsidian without Stow.

## Scope

- Add and remove text, links, images, and animated GIFs on a card.
- Select media that already exists in the Obsidian vault.
- Upload media selected from the file system to `/stow/files`, retain its original name, and reference it from the vault-backed Markdown note.
- Reference existing vault media in place without creating duplicates.
- Preserve the underlying source asset when a card reference is removed.

Out of scope: audio/video handling, media editing, remote asset hosting, automatic garbage collection, and bulk media migration.

## Dependencies

Depends on Epic 00's Markdown/Canvas findings and Epic 02's card creation and editing workflows.

## Units of Work

### 03.1 Add and remove text or links

**Outcome:** The user can combine text and links in a card note and edit or remove them later.

**Acceptance criteria:**

- **Given** a card note, **when** the user adds text or a link, **then** it is stored as ordinary Markdown and opens correctly in Obsidian.
- **When** the user edits or removes a text/link item, **then** the note reflects the change without affecting other content or Canvas layout.

### 03.2 Add existing vault media

**Outcome:** The user can attach an image or animated GIF already in the vault.

**Acceptance criteria:**

- **Given** an image or GIF in the vault, **when** the user selects it for a card, **then** the card note references that existing file without creating a duplicate, and the media is visibly rendered in the native Canvas card and Obsidian note view.
- **When** the note is opened without Stow, **then** the media remains accessible and the Canvas file card continues to reference the note.
- Selecting the same asset more than once follows a consistent, documented behavior and does not corrupt the note.

### 03.3 Add file-system media

**Outcome:** The user can upload an image or GIF from the file system into `/stow/files` and add a Markdown reference to it.

**Acceptance criteria:**

- **Given** a supported file selected from the file system, **when** the user uploads it to a card, **then** Stow stores it in `/stow/files` with its original name and writes a usable Markdown reference to it; the media is visibly rendered in the native Canvas card and Obsidian note view.
- **Given** an unsupported or inaccessible file, **when** the user attempts to add it, **then** Stow reports the issue and leaves the card note unchanged.
- **Given** a file with the same name already exists in `/stow/files`, **when** the user uploads another file with that name, **then** Stow reports a name collision and prompts the user to rename the new file without overwriting either file.

### 03.4 Remove media references

**Outcome:** The user can remove media from a card without unexpectedly deleting the underlying file.

**Acceptance criteria:**

- **Given** a card with an image/GIF reference, **when** the user clicks its Remove button, **then** the reference is removed from the note and the source file remains in the vault.
- **When** the source file is missing or renamed, **then** Stow reports the unresolved reference and does not silently substitute or delete another asset.

## Unit Tests

Add unit tests for Markdown link/embed formatting, vault path resolution, supported-file validation, filename collision behavior, duplicate-reference behavior, and removal that preserves the source file and unrelated note content. Target 100% function coverage for in-scope, unit-testable media logic; report statement/line and branch coverage separately. Use Jest if compatible with the selected project setup.

## Manual Validation

1. Add a vault image and animated GIF to one card; verify both render and remain usable in Obsidian without Stow.
2. Add text and a link to the same card and verify all content coexists in the Markdown note.
3. Select a file from outside the vault and verify it is stored in `/stow/files` with its original name and referenced from the note.
4. Remove an embed and verify the card note changes while the original asset remains.
5. Upload a same-named file and verify the collision error asks the user to rename the new file without overwriting either file.
6. Rename or remove an asset and verify Stow reports the broken reference without altering unrelated files.

## Open Decisions

- Which file extensions are supported beyond images and animated GIFs?
- Whether the original file outside the vault remains in place and Stow uploads a copy into `/stow/files`, or the note should link directly to an external path.
- Whether Stow permits selecting multiple media files in one action.
- Whether the UI edits Markdown directly or provides structured text/link/media controls.