# Epic 01: Plugin Foundation and Board Access

**Status:** Proposed  
**Outcome:** The user can access multiple Obsidian Canvas boards through Stow and work with one active board at a time.

## Scope

- Establish a loadable Obsidian desktop plugin foundation.
- Create, find, open, and switch between Canvas boards stored in `/stow` at the vault root.
- Keep boards as normal `.canvas` files managed by Obsidian.
- Open Stow from an Obsidian command or sidebar and use the vault's existing storage, synchronization, and sharing behavior.

Out of scope: mobile support, board collaboration, cloud services, and custom rendering of Canvas cards.

## Dependencies

Depends on Epic 00's supported Canvas integration findings. Card authoring, media, and tag filtering depend on selecting an active board.

## Units of Work

### 01.1 Load Stow in Obsidian desktop

**Outcome:** The user can enable Stow and reach its board workflow in a supported Obsidian desktop installation.

**Acceptance criteria:**

- **Given** a compatible Obsidian desktop installation, **when** Stow is installed and enabled, **then** Obsidian loads it without a startup error.
- **Given** Stow is enabled, **when** the user invokes its documented entry point, **then** the board workflow opens and reports actionable errors if no usable board is available.
- The minimum Obsidian version and supported desktop operating systems are recorded before release.

### 01.2 Create and find boards

**Outcome:** The user can identify existing Stow boards and create another standard Canvas board.

**Acceptance criteria:**

- **Given** Stow is installed in a vault, **when** initial setup completes, **then** `/stow` exists at the vault root and existing files are not overwritten.
- **Given** a vault with Canvas files in `/stow`, **when** the user opens the board picker, **then** available boards can be distinguished by name.
- **Given** the user creates a board, **when** the operation completes, **then** a valid `.canvas` file exists directly in `/stow` and is visible through Obsidian file management.
- Creating a board never overwrites an existing file without an explicit confirmation.

### 01.3 Open and switch the active board

**Outcome:** The user can move between multiple boards while Stow presents one active board at a time.

**Acceptance criteria:**

- **Given** multiple boards exist, **when** the user selects one, **then** that board becomes the active board and its Canvas content is shown.
- **When** the user switches to another board, **then** the new board becomes active without changing the previous board's file.
- **When** the user closes and reopens Stow, **then** board selection follows the agreed persistence behavior and no board data is lost.
- **Given** the user opens Stow, **when** they use its documented Obsidian command or sidebar entry point, **then** the board workflow opens.

## Unit Tests

Add unit tests for `/stow` setup, board discovery, board path validation, create-without-overwrite behavior, and active-board selection state. Use the repository's eventual test framework (Jest preferred if compatible); target 100% function coverage for in-scope, unit-testable logic and report statement/line and branch coverage separately.

## Manual Validation

1. Enable Stow in the latest macOS, Windows, and Ubuntu Obsidian desktop environments and invoke it from the command and sidebar.
2. Verify `/stow` is created at the vault root without replacing pre-existing files or folders.
3. Create two boards and verify both are stored directly in `/stow` and visible in Obsidian file management.
4. Open each board in turn and verify only the selected board is active in Stow.
5. Close and reopen Stow and verify the agreed board-selection behavior.
6. Attempt to create a board with an existing name and verify no existing file is silently overwritten.

## Open Decisions

- Minimum supported Obsidian version for the latest macOS, Windows, and Ubuntu releases.
- Whether board selection persists between Stow sessions.