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
- Basic Script Include structure (`MiniCrossword`)
- Import template for word data

### In Progress
- Service Portal widget development
- Game logic implementation
- Validation algorithms
- Scoring system

### To Do
- Timer functionality
- Hint reveal logic
- Leaderboard ranking algorithm
- Daily puzzle scheduling
- Random puzzle generation
- Mobile-responsive design

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
- Status: Empty scaffold (initialize function only)
- Location: `/07038a37530036d0db6151a0a0490ed8/update/`

This is where puzzle generation, validation, and scoring logic should be implemented.

## Puzzle Grid Mathematics

Critical constraint: **Words must intersect correctly**

For a valid 5x5 puzzle:
- 5 horizontal words (rows 1-5, all starting at column 1)
- 5 vertical words (columns 1-5, all starting at row 1)
- Each cell is the intersection of exactly one horizontal and one vertical word
- Word at row R, column C must have matching letter at position C with vertical word at column C (at its position R)

Example validation logic needed:
```javascript
// Horizontal word at row 2: "APPLE"
// Vertical word at column 3: "PANEL"
// These must share 'P' at position (row:2, col:3)
// Horizontal word position 3 = Vertical word position 2
```
