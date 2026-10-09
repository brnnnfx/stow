# Stow POC Design

## HLD

### Scope

This high-level design describes the user and system flow for creating a Canvas, adding and editing a note-backed card, importing supported media, adding a tag, and removing the card. It follows the [architecture](./architecture.md) and [feature specification](./feature-spec.md). Boards, card notes, and imported media are normal files in the active Obsidian vault; Obsidian provides the plugin runtime, Canvas experience, and vault file operations.

### Main user flow

1. The user opens Stow from its Obsidian command or sidebar and chooses to create a Canvas board.
2. The user names the board. Stow creates a native Canvas file directly in `/stow`. If the target already exists, Stow does not overwrite it and reports the conflict for non-destructive resolution.
3. With the new board active, the user chooses to add a card, enters a title and initial text, and saves.
4. Stow creates a standalone Markdown note in `/stow/cards` and adds a native Canvas file-card node referencing that note. The card note uses the specified timestamp filename pattern. If the generated path already exists, Stow does not overwrite it; the exact resolution remains open in the feature spec (OQ-01).
5. The user edits the card and selects a supported image or animated GIF from the local file system. Stow imports a copy into `/stow/files` under its original name and adds an Obsidian-compatible reference to the card note. If the file is unsupported or inaccessible, Stow reports the issue and leaves the note unchanged. If the destination filename already exists, Stow reports the conflict and asks the user to rename the new file; it does not overwrite or automatically rename either file.
6. The user opens the card's tag controls, selects an existing vault tag or enters a freeform tag, and saves. Stow updates only that card's tag data in its Markdown note.
7. The user activates the clearly labeled Remove action for the card. Stow deletes the selected Canvas node and its associated note. It retains the imported media file and any other media referenced by the note. Unrelated Canvas nodes and their layout remain unchanged.

### Main system flow

```text
User            Stow plugin         Obsidian Canvas       Vault
 |                   |                    |                 |
 | Create board      |                    |                 |
 |------------------>| Create native      |                 |
 |                   | Canvas board       |---------------->|
 |                   |                    |                 | /stow/*.canvas
 |                   |<--------------------------------------|
 |                   |                    |                 |
 | Add card + text   |                    |                 |
 |------------------>| Create Markdown note                |
 |                   |-------------------------------------->|
 |                   |                    | /stow/cards/*.md |
 |                   | Add file-card node |                 |
 |                   |------------------->|                 |
 |                   |                    | Save Canvas node |
 |                   |                    |---------------->|
 |                   |                    |                 |
 | Upload image/GIF  |                    |                 |
 |------------------>| Copy supported file                 |
 |                   |-------------------------------------->|
 |                   |                    | /stow/files/*    |
 |                   | Add media reference to card note     |
 |                   |-------------------------------------->|
 |                   |                    |                 |
 | Add tag           |                    |                 |
 |------------------>| Update card note tag data            |
 |                   |-------------------------------------->|
 |                   |                    |                 |
 | Remove card       |                    |                 |
 |------------------>| Remove selected Canvas node           |
 |                   |------------------->|---------------->|
 |                   | Delete card note; retain media        |
 |                   |-------------------------------------->|
```

The diagram shows responsibility and data movement at a conceptual level; it does not specify APIs, protocols, or internal Obsidian interfaces.

### Component interactions

- **User to Stow plugin:** Initiates board and card workflows, provides content and tags, selects local media, and invokes explicit removal.
- **Stow plugin to Obsidian Canvas:** Opens the active native board, adds or removes file-card nodes, and preserves the Canvas identity and layout of unrelated nodes.
- **Stow plugin to vault:** Discovers board files, creates Canvas files and Markdown notes, updates card content and tags, imports supported media, and removes the selected card note as required.
- **Obsidian Canvas to vault:** Persists native board membership and layout, including the addition or removal of a Canvas node.
- **Obsidian host to user:** Displays the board and card content using native Canvas and Markdown behavior, including supported image and animated GIF rendering.

### Data movement and persistence

| Data | Movement | Durable location / behavior |
| --- | --- | --- |
| Board identity, membership, and layout | Stow creates a native board and associates card nodes; Obsidian Canvas persists board changes. | Canvas file directly under `/stow`. |
| Card title and text | User edits through Stow; Stow writes the card content as Markdown. | Standalone note under `/stow/cards`; readable and editable in Obsidian without Stow. |
| Imported media bytes | User selects a supported local image or GIF; Stow copies it to the vault and references it from the note. | Separate file under `/stow/files`, using its original name. The note stores a reference, not binary content. |
| Card tags | User selects or enters tags; Stow updates the selected card's tag data. | Only the card's Markdown note is changed. |
| Remove operation | Stow removes the selected board node and associated note. | Referenced or uploaded media remains in the vault; unrelated board nodes and layout are preserved. |
| Tag filter state | User selects or clears tags; Stow changes which cards are visible on the active board. | View-only; does not alter saved Canvas membership or layout. |

### Traceability to the feature specification

| HLD flow or behavior | Feature specification trace |
| --- | --- |
| Open Stow and create a board in `/stow`; avoid overwriting an existing path. | FR-01; AC-01.1, AC-01.5, AC-01.6; A-02, A-08. |
| Add a titled, text-initialized note-backed card as a native Canvas file card. | FR-03; AC-03.1–03.4; A-03, A-13. |
| Handle generated card-note filename collisions without overwrite. | FR-03; AC-03.2; OQ-01. |
| Import supported local images/GIFs into `/stow/files` and reference from Markdown. | FR-06; AC-06.3, AC-06.4, AC-06.6–06.8; OQ-02; A-06. |
| Edit freeform tags on the card note. | FR-07; AC-07.1–07.4; A-07. |
| Remove the card's Canvas node and note, retaining media and unrelated layout. | FR-05; AC-05.1–05.3; NFR-03; A-05. |
| Keep filtering view-only if used before removal. | FR-08; AC-08.1–08.6; A-07. |
| Keep board, note, and media as vault files under the defined folders. | FR-10; AC-10.1–10.4; A-02, A-04. |
