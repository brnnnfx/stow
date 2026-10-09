# Project Overview

## What this project is

Stow is a personal hobby project to build an Obsidian plugin that explores replacing selected Google Keep workflows. The first goal is a desktop technical proof of concept that can grow over time, not a complete Google Keep replacement.

## Goals and priorities

The initial POC priorities, in order, are:

1. A scrollable desktop board of cards.
2. Create, edit, and delete cards.
3. Add and remove media on cards.
4. Add and remove tags on individual cards.
5. Filter the board by card tags.

Content search and theming are desirable, but are lower priority than the initial POC.

## Users and stakeholders

The intended target users are individuals who use Obsidian. The project owner, who also uses Google Keep, is the initial POC user. Team collaboration and multi-user workflows are not part of the initial scope.

## Boards and card model

- A board is an Obsidian Canvas file (`.canvas`). Stow should support multiple boards, with one board open at a time.
- A card is a native, resizable Canvas file card that references its own standalone Markdown note named `YYYY-MM-DD-HHmm-stow.md` using 24-hour local time (for example, `2026-10-08-2142-stow.md`). A timestamp collision remains an open behavior decision.
- The Canvas file owns board layout, including card placement and size. The Markdown note owns the card's content and should remain readable and editable in Obsidian without Stow.
- Stow creates or reuses `/stow` at the vault root when installed. Canvas boards are stored directly in `/stow`, card notes in `/stow/cards`, and files uploaded through Stow in `/stow/files`.
- Stow uses Obsidian Vault's existing storage, synchronization, and sharing behavior. It does not manage these options separately.
- Canvas is the chosen direction for the initial POC. A small interoperability spike should validate the note-backed card model before broader implementation.

## Cards and media

The Markdown note for a card may combine any of the following:

- One or more images, including animated GIFs
- Links
- Blocks of text

Media should be addable from the file system or from files already in the Obsidian vault. Existing vault media is linked without duplication. Files uploaded through Stow are stored with their original names in `/stow/files` and referenced from the card note. Images render in the native Canvas card and Obsidian note view; animated GIFs play even with Stow disabled. Removing a media reference does not delete the underlying file; removing a card deletes its note but preserves media files it referenced. Notes and media remain usable in Obsidian without Stow.

Tags should be freeform, support reuse of tags found in the vault, and allow new tags. The Markdown representation for tags is not yet decided.

## Workflows and likely modules

- Browse a scrollable board and filter cards by their tags.
- Create, edit, and delete cards.
- Attach or remove media from the file system or Obsidian vault.
- Manage tags on each card.

Likely implementation areas include the board/card interface, card content and metadata, media handling, and tag filtering. These are areas to investigate, not settled module boundaries.

## Constraints and open decisions

- The initial scope is an Obsidian desktop plugin only, targeting the latest macOS and Windows releases and the latest Ubuntu release available at validation time.
- Only the latest Obsidian desktop version is supported, with no backward-compatibility support. As of 2026-10-08, that version is 1.13.7.
- Canvas files and standalone Markdown notes are the selected storage direction for the initial POC; the interoperability spike should validate that Stow can update note content without damaging Canvas layout.
- Stow is opened from an Obsidian command or sidebar.
- Card removal uses an explicit Remove action, deletes the Canvas node and card note, and retains media files.
- Files uploaded through Stow are placed in `/stow/files` with their original names. A filename collision must be reported; the user is prompted to rename the new file. Existing vault media is linked rather than duplicated. Whether an upload keeps the original outside-vault file as well as a vault copy remains open.
- Storage, synchronization, and sharing use the existing Obsidian Vault behavior; Stow does not manage these settings.
- Tag representation (inline tags or frontmatter) is undecided. Multiple selected tags match all selected tags, and nonmatching cards are hidden without changing the Canvas file.
- Exploratory performance targets are to observe 100-card board readiness within 3 seconds and tag-filter updates within 500 milliseconds. They are not release gates; record the actual machine and environment used rather than preselecting a reference desktop.
- Search and theme support are later priorities; their exact scope is undecided.
- The plugin's internal architecture is undecided.