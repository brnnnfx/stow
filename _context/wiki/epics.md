# Stow POC Epic Draft

This is a working plan for a desktop-only technical POC that may grow into a hobby project. The goal is to validate native Obsidian Canvas cards backed by standalone Markdown notes, then deliver the core board and card workflows. Search, advanced theming, and mobile support are outside the initial POC.

The [POC feature specification](../../delivery/specs/poc/feature-spec.md) is the canonical source for detailed observable requirements and Given/When/Then acceptance criteria. This page summarizes the epics; keep its outcomes and decisions aligned with that specification.

## Epic 0: Canvas and Markdown Interoperability Spike

**Outcome:** Confirm that Stow can use native, resizable Canvas file cards backed by standalone Markdown notes without compromising editing in Obsidian or the Canvas layout.

**Acceptance criteria:**

- A `.canvas` board can reference a standalone Markdown note as a native file card.
- The note can contain a title, text, links, tags, and image embeds, and remains readable and editable with Stow disabled.
- Images are visibly rendered in the native Canvas card and Obsidian note view; animated GIFs play even when Stow is disabled.
- Stow-side note edits do not change the Canvas card's position or dimensions; edits made in Obsidian remain visible through the card.
- Rename and missing-note behavior is documented, along with the supported way for the plugin to read and write Canvas data.
- The spike records any limitations or reasons to revise the note-backed card model before feature work begins.

## Epic 1: Plugin Foundation and Board Access

**Outcome:** The user can create, find, open, and switch between multiple Canvas boards in the Obsidian desktop plugin, with one board active at a time.

**Acceptance criteria:**

- The plugin can be installed and opened in a supported Obsidian desktop version.
- The user can create or select a board in `/stow`, open it, and switch to another board.
- Boards are standard `.canvas` files that remain accessible through Obsidian's file management.
- Stow creates or reuses `/stow` at the vault root and does not overwrite existing files during setup.
- Stow is opened from an Obsidian command or sidebar. Target platforms are the latest macOS, Windows, and Ubuntu releases available at validation time.

## Epic 2: Card Authoring and Lifecycle

**Outcome:** The user can create, edit, and remove note-backed cards on the active board.

**Acceptance criteria:**

- Creating a card creates a standalone Markdown note and a native Canvas file card that references it.
- Card content can be edited in Stow or directly in Obsidian and remains coherent in both places.
- An explicit Remove action deletes the Canvas node and its Markdown note in `/stow/cards`, while retaining media files referenced by the card.
- Canvas layout and unrelated nodes are preserved when card content or membership changes.

## Epic 3: Media in Cards

**Outcome:** The user can add and remove text, links, images, and animated GIFs using files already in the vault or selected from the file system.

**Acceptance criteria:**

- Card notes can contain the supported content types in combination.
- Existing vault media can be selected and embedded in a card note.
- Media can be selected from the file system and referenced from the card note.
- Media remains usable in Obsidian without Stow.
- Files uploaded through Stow are stored in `/stow/files` with their original names and linked from the card note. Existing vault media is linked without duplication.
- A filename collision is reported and the user is prompted to rename the new file; no existing file is overwritten.
- Removing a media reference does not delete the underlying file. Obsidian Vault controls storage, synchronization, and sharing; Stow adds no separate management for these options.

## Epic 4: Tags and Board Filtering

**Outcome:** The user can add or remove freeform tags on cards and filter the active board using tags already present in the vault or newly created for a card.

**Acceptance criteria:**

- Tags are stored in the Markdown note using a representation editable in Obsidian.
- The user can reuse tags found in the vault and add new tags.
- The active board can be filtered by one or more tags without changing the underlying notes or Canvas layout.
- Multiple selected tags match all selected tags; nonmatching cards are hidden without changing saved Canvas layout or membership.
- The tag representation remains to be decided and documented.

## Cross-Cutting Acceptance Criteria

- Define supported Obsidian desktop versions and operating systems before release.
- Handle renamed or missing notes and media without data loss.
- Add focused tests for Canvas JSON and Markdown transformations, plus integration checks for file updates and layout preservation.
- Target 100% function coverage for in-scope, unit-testable production logic; report statement/line and branch coverage separately. Confirm whether React and Jest fit the Obsidian plugin setup.
- Provisional performance targets are a usable 100-card board within 3 seconds and tag-filter updates within 500 milliseconds on an agreed reference desktop.

## Dependencies and Open Decisions

The interoperability spike comes first. Board access depends on its findings; card authoring depends on board access and the note-backed card model. Media and tag work depend on the Markdown representation established for cards and can proceed after the card lifecycle is stable.

Still to decide: card-note filename and conflict rules; whether selecting a filesystem file copies it into `/stow/files` or links to its external path; tag syntax; the performance reference computer; and which Obsidian version to test on the agreed OS releases.