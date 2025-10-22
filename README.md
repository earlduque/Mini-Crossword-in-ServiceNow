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

### Completed ✅
- Database schema design and table creation (8 core tables)
- Table relationships and indexes
- Security rules (ACLs)
- **Automated puzzle generation algorithm**
  - Backtracking constraint satisfaction approach
  - Intelligent word placement with intersection validation
  - Supports category filtering or all-word puzzles
  - Minimum 5 words, allows blank spaces
  - First row guarantee
- **Puzzle validation logic**
- **Full MiniCrossword Script Include with JSDoc documentation**
- Import template for word data

### In Progress 🚧
- Service Portal widget development
- User gameplay validation
- Scoring system

### To Do 📋
- Timer functionality
- Hint reveal logic (reveal word, reveal letter)
- Leaderboard ranking algorithm
- Daily puzzle scheduling (scheduled job)
- Mobile-responsive design
- User statistics dashboard
- Game state persistence (save/resume)

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

#### Setting Up Word Database
1. Create word categories in the Word Category table (`x_1468549_mini_c_0_word_category`)
   - Example categories: Sports, Food, Technology, Movies, etc.
2. Add 5-letter words with hints to the Word table (`x_1468549_mini_c_0_word`)
   - Each word must be exactly 5 letters
   - Include engaging hints/clues
   - Assign to appropriate category (optional)

#### Generating Puzzles

**Option 1: Using Background Scripts**

Navigate to **System Definition > Scripts - Background** and run:

```javascript
// Generate a random puzzle from all available words
var mc = new MiniCrossword();
var result = mc.generatePuzzle(null, 'random', null);

if (result.success) {
    gs.info('✓ Puzzle generated successfully!');
    gs.info('  Puzzle ID: ' + result.puzzleSysId);
    gs.info('  Words placed: ' + result.wordCount);
    gs.info('  Grid coverage: ' + result.gridCoverage);
    gs.info('  Attempts used: ' + result.attemptsUsed);
} else {
    gs.error('✗ Generation failed: ' + result.message);
}
```

```javascript
// Generate a daily puzzle for today with specific category
var categoryGR = new GlideRecord('x_1468549_mini_c_0_word_category');
if (categoryGR.get('name', 'Sports')) {
    var mc = new MiniCrossword();
    var today = new GlideDateTime().getDate().getValue(); // YYYY-MM-DD
    var result = mc.generatePuzzle(categoryGR.getUniqueValue(), 'daily', today);

    gs.info('Daily puzzle: ' + result.message);
}
```

**Option 2: Using Business Rules** (Future enhancement)
- Create a Business Rule on the `daily_puzzle` table
- Trigger puzzle generation automatically when a new daily_puzzle record is created

**Option 3: Using Scheduled Jobs** (Future enhancement)
- Set up a daily scheduled script to auto-generate tomorrow's puzzle

#### Validating Puzzles

```javascript
// Validate an existing puzzle
var mc = new MiniCrossword();
var result = mc.validatePuzzle('your_puzzle_sys_id_here');

if (result.valid) {
    gs.info('✓ Puzzle is valid!');
    gs.info('  Words: ' + result.wordCount);
    gs.info('  Intersections checked: ' + result.intersectionCount);
} else {
    gs.error('✗ Puzzle has errors:');
    for (var i = 0; i < result.errors.length; i++) {
        gs.error('  - ' + result.errors[i]);
    }
}
```

### For Players (Once Service Portal is Complete)
1. Navigate to the Mini Crossword Service Portal page
2. Choose **"Times"** for daily competitive puzzle or **"Zen"** for random practice
3. Click cells to select across/down words
4. Type letters to fill in answers
5. Use hint buttons if needed (affects score in Times mode)
6. Complete the puzzle to see your time and leaderboard rank

## Technical Details

- **Scope**: `x_1468549_mini_c_0` (Mini-crossword)
- **Platform**: ServiceNow (Service Portal)
- **Primary Language**: JavaScript (both server-side and client-side)
- **UI Framework**: Service Portal (AngularJS + Bootstrap)

### Puzzle Generation Algorithm

The application uses an intelligent **backtracking constraint satisfaction algorithm** to generate valid crossword puzzles:

**Three-Phase Generation:**
1. **Phase 1**: Guarantees a horizontal word in the first row (tries columns 1-5)
2. **Phase 2**: Attempts to place additional horizontal words in rows 2-5
3. **Phase 3**: Attempts to place vertical words in columns 1-5

**Key Features:**
- **Intersection Validation**: Ensures words share matching letters at crossing points
- **Retry Logic**: Up to 10 attempts with different word shuffles if generation fails
- **Flexible Layout**: Allows blank spaces when no compatible words fit
- **Minimum Threshold**: Requires at least 5 words for a valid puzzle
- **Category Support**: Can filter by word category or use entire word database

**Algorithm Constraints:**
- Grid size: 5x5
- Word length: Exactly 5 letters
- Max attempts per generation: 1,000 iterations
- Max retries: 10 different starting configurations

**Example Grid Output:**
```
  1 2 3 4 5
1 W O R L D
2 □ P □ □ □
3 □ E □ □ □
4 □ N □ □ □
5 □ □ □ □ □

Words placed: 7
Grid coverage: 52%
```

For detailed implementation, see `sys_script_include_2f0e82fc53f83e10db6151a0a0490e40.xml`

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

- **Phase 1**: Complete database structure ✅ **COMPLETED**
- **Phase 2**: Implement puzzle generation algorithm ✅ **COMPLETED**
- **Phase 3**: Develop Service Portal widget 🚧 **IN PROGRESS**
- **Phase 4**: Implement game logic and user validation 🚧 **IN PROGRESS**
- **Phase 5**: Add timer and scoring ⏳ **UPCOMING**
- **Phase 6**: Create leaderboard displays ⏳ **UPCOMING**
- **Phase 7**: Implement daily puzzle automation (scheduled jobs) ⏳ **UPCOMING**
- **Phase 8**: Mobile optimization ⏳ **UPCOMING**
- **Phase 9**: Analytics and user statistics ⏳ **UPCOMING**

### Recent Updates

**2025-10-22**:
- ✅ Implemented complete puzzle generation algorithm with backtracking
- ✅ Added puzzle validation logic for intersection checking
- ✅ Created comprehensive JSDoc documentation for MiniCrossword Script Include
- ✅ Added support for category-filtered and all-word puzzle generation
- ✅ Implemented retry logic with seeded randomization for puzzle variety
