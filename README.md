# Stow

Stow is a personal Obsidian plugin project exploring a desktop card board for replacing selected Google Keep workflows.

## Initial POC

- Create, find and switch between Canvas boards stored in `/stow`, with one active board at a time.
- Browse, pan, and resize native Canvas file cards; layout changes persist without moving or changing unrelated cards.
- Create and edit standalone card notes in `/stow/cards`, named `YYYY-MM-DD-HHmm-stow.md` (for example, `2026-10-08-2142-stow.md`); card notes remain readable and editable in Obsidian without Stow.
- Remove a card with its Remove action; its Canvas node and card note are deleted, while referenced media files remain.
- Add text and links, link existing vault media without duplication, and upload image/GIF files to `/stow/files` with their original names.
- Images render in the native Canvas card and Obsidian note view; animated GIFs play even when Stow is disabled.
- Resolve upload name collisions by prompting the user to rename the new file; never overwrite existing files.
- Add and remove freeform tags. Filtering requires all selected tags to match, hides nonmatching cards, and does not change saved Canvas layout.

Stow uses Obsidian Vault's storage, synchronization, and sharing behavior; it does not manage these options separately. The initial support target is the latest macOS, Windows, and Ubuntu releases.

Content search and theming are useful follow-up features, but are not required for the initial POC.

Only the latest Obsidian desktop version is supported, with no backward-compatibility support; as of 2026-10-08, that is Obsidian 1.13.7. Performance targets are exploratory measurements, not release gates.

## Project direction

The initial scope is an Obsidian desktop plugin for the latest macOS, Windows, and Ubuntu releases. The POC uses Canvas boards in `/stow`, native resizable file cards, and standalone Markdown notes in `/stow/cards`. Uploaded files go in `/stow/files`; existing vault media is linked without duplication. Obsidian Vault handles storage, synchronization, and sharing using its existing behavior.

For detailed requirements and acceptance criteria, see the [POC feature specification](delivery/specs/poc/feature-spec.md). The [project wiki](_context/wiki/project.md) records project context and open decisions; the [epic draft](_context/wiki/epics.md) summarizes the proposed work. Working preferences and links to the detailed docs are in the [wiki index](_context/wiki/index.md).
