# Epic 04: Tags and Board Filtering

**Status:** Proposed  
**Outcome:** The user can add or remove freeform tags on cards, reuse tags found in the vault, create new tags, and filter the active board without changing saved Canvas layout.

## Scope

- Read and update card tags using a Markdown representation editable in Obsidian.
- Offer reuse of tags found in the vault while allowing new freeform tags.
- Filter the active board by one or more tags and clear the filter.

Out of scope: global search, tag hierarchies, tag renaming across the vault, saved filter presets, and filtering multiple boards at once.

## Dependencies

Depends on Epic 00 proving tag filtering can work with the native Canvas view without destructive board changes. Tag editing depends on Epic 02's note-backed card lifecycle. The all-selected-tags matching rule is settled; the Markdown tag syntax remains open and must be chosen before final acceptance tests are fixed.

## Units of Work

### 04.1 Read and edit card tags

**Outcome:** The user can see, add, and remove freeform tags on an individual card, including tags already used elsewhere in the vault.

**Acceptance criteria:**

- **Given** a card note with tags in the agreed Markdown representation, **when** the user opens its tag editor, **then** Stow shows the card's current tags.
- **Given** tags found in the vault, **when** the user edits a card's tags, **then** those existing tags can be reused and a new freeform tag can be entered.
- **When** the user adds or removes a tag, **then** only that card note's tag data changes and it remains editable in Obsidian.
- Duplicate tags and empty tag values are handled consistently and do not create malformed Markdown.

### 04.2 Filter the active board

**Outcome:** The user can display cards matching selected tags on the active board and return to the unfiltered board.

**Acceptance criteria:**

- **Given** cards with different tags on the active board, **when** the user selects a tag filter, **then** only cards containing all selected tags remain visible and nonmatching cards are hidden.
- **When** the user clears the filter, **then** all cards return and their Canvas positions, sizes, and saved membership are unchanged.
- **When** the active board changes, **then** filtering applies only to the newly active board and does not mutate other boards.
- **Given** a card's tag is edited in Obsidian, **when** Stow refreshes, **then** the filter uses the latest saved tag data.

### 04.3 Handle empty and changing results

**Outcome:** Filtering remains understandable when no card matches or tags change.

**Acceptance criteria:**

- **Given** no card matches the selected tags, **when** filtering is applied, **then** Stow indicates that the result is empty and provides a clear way to remove the filter.
- **When** a tag is removed from a card or no longer exists in the vault, **then** the card's saved note is not changed by the filter itself and the displayed results update after refresh.

## Unit Tests

Add unit tests for parsing, normalizing, adding, and removing tags in the selected Markdown syntax; discovering/reusing vault tags; duplicate/empty values; and all-selected-tags filter matching. Add tests proving filtering does not alter Canvas node data or layout. Target 100% function coverage for in-scope, unit-testable tag/filter logic; report statement/line and branch coverage separately. Use Jest if compatible with the project setup.

## Manual Validation

1. Add an existing vault tag to a card, create a new tag, and remove a tag; verify the Markdown note remains readable and editable in Obsidian.
2. Add several cards with overlapping tags and verify that all selected tags must match and nonmatching cards are hidden.
3. Clear the filter and verify all cards return in their original positions and sizes.
4. Edit a card's tags directly in Obsidian and verify Stow refreshes its tag list and filter results.
5. Test a filter with no matches and verify the empty state and clear-filter action.

## Open Decisions

- Inline tags, frontmatter tags, or a defined combination.
- How tags in nested paths or aliases are displayed, if those are supported.
- The interaction used to visually filter native Canvas cards while preserving their saved layout.