# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ServiceNow scoped application that recreates the New York Times Mini Crossword game within the ServiceNow platform. The application features 5x5 crossword puzzles with timed gameplay, scoring, hints, and leaderboard tracking.

**Scope Name**: `x_1468549_mini_c_0` (Mini-crossword)
**Application ID**: `07038a37530036d0db6151a0a0490ed8`
**Instance**: dev271501

## ServiceNow Application Structure

This is a **ServiceNow XML-based repository**. The application files are stored as XML records under the `07038a37530036d0db6151a0a0490ed8/` directory:

- `/dictionary/` - Table definitions
- `/update/` - System updates (fields, choices, documentation, ACLs, modules, etc.)
- `/author_elective_update/` - Author-elective updates (widgets, pages, containers, security)
- `/sys_app_*.xml` - Application metadata

### Key Architecture Components

**Server-side code** is embedded in XML files:
- Script Includes: `sys_script_include_*.xml` files
- Business Rules: `sys_script_*.xml` files
- Scheduled Jobs: `sysauto_script_*.xml` files

**Service Portal components** are in `sp_widget_*.xml`, `sp_page_*.xml`, `sp_instance_*.xml` files with three sections:
- `<script>` - Server-side code (GlideRecord queries, data preparation)
- `<client_script>` - Client-side AngularJS controller
- `<template>` - HTML template with AngularJS directives
- `<css>` - Widget-specific styles

## Database Schema (8 Core Tables)

### Puzzle Construction Tables

**`x_1468549_mini_c_0_word`** - 5-letter words used in puzzles
- `word` (5 char max) - The actual word
- `hint` (2048 char) - Clue text
- `definition` (2048 char) - Full definition
- `category` - Reference to word_category

**`x_1468549_mini_c_0_word_category`** - Themed categories (Sports, Food, etc.)
- `name` (128 char)
- `description` (2048 char)

**`x_1468549_mini_c_0_daily_puzzle`** - Complete puzzle configuration
- `type` - "daily" or "random"
- `date` - For daily puzzles
- `category` - Optional theme

**`x_1468549_mini_c_0_puzzle_grid`** - Word placement in 5x5 grid
- `daily_puzzle` - Reference to puzzle
- `word` - Reference to word
- `row` (1-5) - Grid row position
- `column` (1-5) - Grid column position
- `direction` - "horizontal" or "vertical"
- Each puzzle has exactly 10 puzzle_grid entries (5 across + 5 down)

### Gameplay Tables

**`x_1468549_mini_c_0_instance`** (extends task table)
- `puzzle` - Reference to daily_puzzle
- `type` - "times" (competitive) or "zen" (practice)
- `assigned_to` - Player (from task)
- `state` - Progress state (from task)
- `opened_at` - Timer start (from task)

**`x_1468549_mini_c_0_hint_use`** - Hint usage tracking
- `instance` - Reference to play session
- `user` - Player who used hint
- `word` - Word hint was for
- `type` - "word" (reveal whole word) or "letter" (reveal single letter)

**`x_1468549_mini_c_0_leaderboard`** - High scores
- `user` - Player
- `score` - Time-based score

**`u_imp_tmpl_x_1468549_mini_c_0_word`** - Import template for bulk word loading
- `u_word` - Word text
- `u_category` - Category reference

## Data Flow

1. **Admin creates puzzle**: Creates `daily_puzzle` record and 10 `puzzle_grid` entries defining word positions
2. **User starts puzzle**: `instance` record created linking user to puzzle
3. **Gameplay**: User interacts with Service Portal widget to fill grid
4. **Validation**: Backend checks user input against `puzzle_grid` word assignments
5. **Completion**: `leaderboard` entry created with time/score
6. **Hints tracked**: Each hint request creates a `hint_use` record

## Development Status

### Completed
- Database schema (all 8 tables)
- Table relationships and indexes
- Security rules (ACLs)
- **Puzzle generation algorithm with backtracking and constraint satisfaction**
- **Puzzle validation logic**
- **MiniCrossword Script Include with full JSDoc documentation**
- Import template for word data

### In Progress
- Service Portal widget development
- User gameplay validation
- Scoring system

### To Do
- Timer functionality
- Hint reveal logic (reveal word, reveal letter)
- Leaderboard ranking algorithm
- Daily puzzle scheduling (scheduled job to auto-generate)
- Mobile-responsive design
- Game state persistence (save/resume)

## Working with ServiceNow Source Control

**Important**: This repository is managed by ServiceNow's built-in source control. Changes should typically be made in the ServiceNow instance UI, not by directly editing XML files.

### Making Changes

1. **Work in the ServiceNow instance** (dev271501)
2. **Link to source control** creates/updates XML files
3. **Commit from instance** to push changes to this repo

### File Naming Convention

ServiceNow uses sys_ids in filenames. For example:
- `sys_script_include_2f0e82fc53f83e10db6151a0a0490e40.xml` = MiniCrossword Script Include
- Table prefixes: `sys_dictionary_*`, `sys_choice_*`, `sys_security_acl_*`, `sp_widget_*`

## Script Includes

Currently only one Script Include exists:

**MiniCrossword** (`sys_script_include_2f0e82fc53f83e10db6151a0a0490e40.xml`)
- API Name: `x_1468549_mini_c_0.MiniCrossword`
- Access: `package_private`
- Status: **Fully implemented with puzzle generation and validation**
- Location: `/07038a37530036d0db6151a0a0490ed8/update/`

### Implementation Details

The MiniCrossword Script Include provides automated puzzle generation using a backtracking constraint satisfaction algorithm.

#### Public Methods

**`generatePuzzle(categorySysId, puzzleType, puzzleDate)`**
- Generates a complete crossword puzzle with word placement
- Parameters:
  - `categorySysId` (String, optional): Category to filter words. If null/empty, uses all words.
  - `puzzleType` (String, required): Either 'daily' or 'random'
  - `puzzleDate` (String, optional): Date for daily puzzles (YYYY-MM-DD format). Required for 'daily' type.
- Returns object with: `success`, `puzzleSysId`, `wordCount`, `gridCoverage`, `message`, `attemptsUsed`
- Creates both `daily_puzzle` and `puzzle_grid` records automatically

**`validatePuzzle(puzzleSysId)`**
- Validates an existing puzzle's word intersections
- Parameters:
  - `puzzleSysId` (String, required): sys_id of the daily_puzzle record
- Returns object with: `valid`, `errors`, `wordCount`, `intersectionCount`
- Useful for verifying manually-created puzzles or debugging

#### Algorithm Strategy

The generation uses a three-phase approach:

1. **Phase 1 - First Row Guarantee**: Places a horizontal word in row 1 (trying columns 1-5)
2. **Phase 2 - Additional Horizontal Words**: Attempts to place words in rows 2-5
3. **Phase 3 - Vertical Words**: Attempts to place words in columns 1-5

**Key Features:**
- Backtracking with retry logic (up to 10 attempts with different word shuffles)
- Intersection validation (letters must match at crossing points)
- Minimum 5 words required for valid puzzle
- Allows blank spaces when no compatible words fit
- Seeded random shuffling for variety while maintaining reproducibility

**Configuration Constants:**
- `GRID_SIZE`: 5 (5x5 grid)
- `MIN_WORD_COUNT`: 5 (minimum words for valid puzzle)
- `MAX_ATTEMPTS`: 1000 (max iterations per generation attempt)
- `MAX_RETRIES`: 10 (max retry attempts with different seeds)

#### Usage Examples

```javascript
// Generate random puzzle from all words
var mc = new MiniCrossword();
var result = mc.generatePuzzle(null, 'random', null);
gs.info('Puzzle created: ' + result.puzzleSysId + ' with ' + result.wordCount + ' words');

// Generate daily puzzle from specific category
var categoryGR = new GlideRecord('x_1468549_mini_c_0_word_category');
if (categoryGR.get('name', 'Sports')) {
    var result = mc.generatePuzzle(categoryGR.getUniqueValue(), 'daily', '2025-10-23');
    gs.info('Daily puzzle: ' + result.message);
}

// Validate a puzzle
var result = mc.validatePuzzle('puzzle_sys_id_here');
if (!result.valid) {
    gs.error('Puzzle validation failed: ' + result.errors.join(', '));
}
```

#### Calling from Business Rules or Flow Actions

```javascript
// Business Rule (e.g., on daily_puzzle table, before insert)
(function executeRule(current, previous /*null when async*/) {
    if (current.type == 'daily' && !current.isNewRecord()) {
        return; // Only generate on insert
    }

    var mc = new MiniCrossword();
    var category = current.getValue('category') || null;
    var date = current.getValue('date');

    // Generate puzzle grid
    var result = mc.generatePuzzle(category, 'daily', date);

    if (!result.success) {
        gs.addErrorMessage('Failed to generate puzzle: ' + result.message);
    } else {
        gs.addInfoMessage('Puzzle generated with ' + result.wordCount + ' words (' + result.gridCoverage + ' coverage)');
    }
})(current, previous);
```

```javascript
// Flow Action Script
(function execute(inputs, outputs) {
    var mc = new MiniCrossword();
    var result = mc.generatePuzzle(
        inputs.category_sys_id,
        inputs.puzzle_type,
        inputs.puzzle_date
    );

    outputs.success = result.success;
    outputs.puzzle_sys_id = result.puzzleSysId;
    outputs.word_count = result.wordCount;
    outputs.message = result.message;
})(inputs, outputs);
```

## Puzzle Grid Mathematics

Critical constraint: **Words must intersect correctly**

The implemented algorithm handles flexible puzzle layouts:

**Flexible Grid Rules:**
- Words can start at any row/column position (not just position 1)
- Minimum 5 words required (not necessarily 10)
- Blank spaces allowed when no compatible words fit
- First row must contain at least one horizontal word
- Words intersect only when they share grid cells
- At intersection points, letters MUST match exactly

**Intersection Validation:**
```javascript
// Example: Horizontal word at row 2, col 1: "APPLE"
// Vertical word at col 3, row 1: "PANEL"
// Cell (2,3) is shared: APPLE[2] = 'P', PANEL[1] = 'P' ✓ Valid

// Invalid example:
// Horizontal at (2,1): "APPLE"
// Vertical at (3,2): "CRANE"
// Cell (2,2) conflict: APPLE[1] = 'P', CRANE[missing] ✗ Invalid
```

**Grid Coordinate System:**
- Rows: 1-5 (top to bottom)
- Columns: 1-5 (left to right)
- Horizontal words: Fixed row, column increases
- Vertical words: Fixed column, row increases
