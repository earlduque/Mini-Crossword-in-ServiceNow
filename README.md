# Mini Crossword in ServiceNow

A ServiceNow application that recreates the experience of the New York Times Mini Crossword game within the ServiceNow platform. Players can solve 5x5 crossword puzzles with timed gameplay, scoring, hints, and leaderboard tracking.

## Overview

This application brings the addictive mini crossword puzzle experience to ServiceNow, allowing users to play quick 5-minute crossword puzzles directly within the platform. Like the NYT Mini Crossword, this app features both daily puzzles and unlimited random puzzles for continuous play.

## Features

### Current Features
- **Structured puzzle system** with words, categories, and grid positioning
- **Multiple play modes**:
  - **Times Mode**: Daily puzzle with leaderboard competition
  - **Zen Mode**: Random puzzles for unlimited practice
- **Hint system** supporting both word and letter hints
- **Leaderboard tracking** for competitive gameplay
- **Category-based word organization** for themed puzzles
- **Data import capability** for bulk word loading

### Planned Features
- Service Portal widget for interactive crossword gameplay
- Timer and scoring system
- Real-time puzzle validation
- Hint usage tracking and penalties
- Daily puzzle rotation
- User statistics and progress tracking

## Table Structure

The application uses the following core tables:

### `x_1468549_mini_c_0_word`
Stores individual 5-letter words that can be used in puzzles.
- **word**: The 5-letter word (max 5 characters)
- **hint**: Clue text for the word (2048 characters)
- **definition**: Full definition of the word (2048 characters)
- **category**: Reference to word_category table
- **number**: Auto-generated identifier

### `x_1468549_mini_c_0_word_category`
Organizes words into themed categories (e.g., "Sports", "Food", "Technology").
- **name**: Category name (128 characters)
- **description**: Category description (2048 characters)
- **number**: Auto-generated identifier

### `x_1468549_mini_c_0_daily_puzzle`
Represents a complete crossword puzzle configuration.
- **number**: Auto-generated puzzle identifier
- **category**: Optional category theme for the puzzle
- **date**: Date for daily puzzles
- **type**:
  - `daily` - Scheduled daily puzzle
  - `random` - On-demand random puzzle

### `x_1468549_mini_c_0_puzzle_grid`
Defines the placement of words within a specific puzzle's 5x5 grid.
- **number**: Auto-generated identifier
- **daily_puzzle**: Reference to the puzzle this grid entry belongs to
- **word**: Reference to the word being placed
- **row**: Grid row position (1-5)
- **column**: Grid column position (1-5)
- **direction**:
  - `horizontal` - Word reads left to right
  - `vertical` - Word reads top to bottom

### `x_1468549_mini_c_0_instance` (extends task)
Represents a user's play session of a puzzle.
- **puzzle**: Reference to the daily_puzzle being played
- **type**:
  - `times` - Timed competitive mode
  - `zen` - Casual practice mode
- Inherits task fields including:
  - **assigned_to**: The user playing
  - **state**: Progress state (Open, In Progress, Complete)
  - **opened_at**: Start time for timer calculation

### `x_1468549_mini_c_0_hint_use`
Tracks when users request hints during gameplay.
- **instance**: Reference to the play session
- **user**: Reference to the user
- **word**: Reference to the word the hint was for
- **type**:
  - `word` - Revealed entire word
  - `letter` - Revealed single letter
- **number**: Auto-generated identifier

### `x_1468549_mini_c_0_leaderboard`
Records high scores and completion times.
- **user**: Reference to the player
- **score**: Numeric score (likely time-based)
- **number**: Auto-generated identifier

### `u_imp_tmpl_x_1468549_mini_c_0_word` (Import Template)
Import set table for bulk loading words into the word table.
- **u_word**: Word text
- **u_category**: Category reference

## Application Architecture

### Data Flow
1. **Puzzle Creation**: Admin creates a daily_puzzle record and defines word placement via puzzle_grid records
2. **Instance Generation**: When a user starts a puzzle, an instance record is created linking the user to the puzzle
3. **Gameplay**: User interacts with Service Portal widget (to be developed) to fill in the grid
4. **Validation**: Backend validates user input against puzzle_grid word positions
5. **Completion**: On successful completion, leaderboard entry is created with time/score
6. **Hint Tracking**: Any hints used are recorded in hint_use table

### Service Portal Widget (Planned)
The interactive crossword interface will be built as a Service Portal widget featuring:
- 5x5 grid rendering
- Click/tap to select cells
- Keyboard input for letters
- Word/cell highlighting
- Hint buttons (reveal letter, reveal word)
- Timer display
- Score calculation
- Puzzle validation
- Completion celebration

## Data Model Relationships

```
word_category
    ↓ (1:N)
word ←──────┐
    ↓ (1:N) │ (N:1)
daily_puzzle ←──┐
    ↓ (1:N)     │ (N:1)
puzzle_grid ────┘
    ↓ (1:N)
instance (extends task)
    ↓ (1:N)
hint_use

instance → leaderboard (implicit via user/score)
```

## How Puzzles Work

1. **Grid Construction**: Each daily_puzzle has 10 puzzle_grid entries (5 horizontal + 5 vertical words)
2. **Word Intersection**: Words must intersect correctly at shared grid cells
3. **Clue Generation**: Each word's hint is displayed for its row/column number
4. **User Input**: Players type letters into the grid cells
5. **Validation**: System checks if filled letters match the puzzle_grid word assignments
6. **Scoring**: Based on completion time minus penalties for hints used

## Development Status

### Completed
- Database schema design and table creation
- Table relationships and indexes
- Security rules (ACLs)
- Basic script include structure (`MiniCrossword`)
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
- User statistics dashboard

## Installation

This is a ServiceNow scoped application. To install:

1. Import the application from source control into your ServiceNow instance
2. Verify all tables and relationships are created
3. Load initial word data via import set or manual entry
4. Create word categories
5. Build puzzle configurations
6. Deploy Service Portal page with crossword widget

## Usage

### For Administrators
1. Create word categories in the Word Category table
2. Add 5-letter words with hints to the Word table
3. Create daily puzzle records
4. Define grid layouts by adding puzzle_grid entries for each word position
5. Schedule daily puzzles by setting the date field

### For Players (Once Service Portal is Complete)
1. Navigate to the Mini Crossword Service Portal page
2. Choose "Times" for daily puzzle or "Zen" for random practice
3. Click cells to select across/down words
4. Type letters to fill in answers
5. Use hint buttons if needed (affects score)
6. Complete the puzzle to see your time and rank

## Technical Details

- **Scope**: `x_1468549_mini_c_0` (Mini-crossword)
- **Platform**: ServiceNow (Service Portal)
- **Primary Language**: JavaScript (both server-side and client-side)
- **UI Framework**: Service Portal (AngularJS + Bootstrap)

## Contributing

When contributing to this project:
1. Follow ServiceNow best practices for scoped application development
2. Test puzzle validation logic thoroughly
3. Ensure grid positioning is mathematically correct
4. Document any new tables or business rules
5. Consider mobile experience in UI development

## License

This project is intended for educational and internal use within ServiceNow instances.

## Roadmap

- **Phase 1**: Complete database structure (✓)
- **Phase 2**: Develop Service Portal widget (In Progress)
- **Phase 3**: Implement game logic and validation
- **Phase 4**: Add timer and scoring
- **Phase 5**: Create leaderboard displays
- **Phase 6**: Implement daily puzzle automation
- **Phase 7**: Mobile optimization
- **Phase 8**: Analytics and user statistics
