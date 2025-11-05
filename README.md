# Mini Crossword in ServiceNow

A ServiceNow application that recreates the experience of the New York Times Mini Crossword game within the ServiceNow platform. Players can solve 5x5 crossword puzzles with timed gameplay, scoring, hints, and leaderboard tracking.

## Overview

This application brings the addictive mini crossword puzzle experience to ServiceNow, allowing users to play quick 5-minute crossword puzzles directly within the platform. Like the NYT Mini Crossword, this app features both daily puzzles and unlimited random puzzles for continuous play.

## Features

### Current Features ✅
- **Automated puzzle generation** with intelligent algorithm
  - Variable-length word support (2-5 letters)
  - Cross-word validation ensuring all formed words are valid
  - True randomization with timestamp-based seeding
  - Automatic detection and recording of all words including cross-words
- **Interactive Service Portal widget**
  - 5x5 crossword grid with classic design
  - Keyboard navigation and input
  - Real-time answer validation (green/red feedback)
  - Auto-completion detection
  - Mobile-responsive design
  - Clue display organized by direction
- **Structured puzzle system** with words, categories, and grid positioning
- **Category-based word organization** for themed puzzles
- **Data import capability** for bulk word loading

### Planned Features 📋
- **Multiple play modes**:
  - **Times Mode**: Daily puzzle with leaderboard competition
  - **Zen Mode**: Random puzzles for unlimited practice
- Timer and scoring system
- **Hint system** supporting both word and letter hints
- Hint usage tracking and penalties
- **Leaderboard tracking** for competitive gameplay
- Daily puzzle rotation (scheduled job)
- User statistics and progress tracking
- Instance record creation for gameplay sessions
- Game state persistence (save/resume)

## Table Structure

The application uses the following core tables:

### `x_1468549_mini_c_0_word`
Stores individual words that can be used in puzzles (supports 2-5 letter words).
- **word**: The word text (max 5 characters, minimum 2)
- **hint**: Short clue text for the word (2048 characters)
- **definition**: Full definition of the word (2048 characters) - **used as puzzle clue**
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
- **row**: Grid row starting position (1-5)
- **column**: Grid column starting position (1-5)
- **direction**:
  - `horizontal` - Word reads left to right
  - `vertical` - Word reads top to bottom

**Note**: Puzzles have variable word counts. The generation algorithm automatically records ALL valid words in the grid, including cross-words formed by word intersections.

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

1. **Grid Construction**: Puzzles have variable word counts (minimum 3 words). The algorithm places words and automatically detects/records all formed words including cross-words.
2. **Word Intersection**: ALL letter sequences (horizontal and vertical) must be valid dictionary words. The algorithm validates this during generation.
3. **Clue Display**: Each word's definition is displayed organized by direction (Across/Down)
4. **User Input**: Players type letters into the grid cells
5. **Real-time Validation**: Widget instantly shows if letters are correct (green) or incorrect (red)
6. **Completion**: Success message displays when all cells are correctly filled
7. **Scoring** (planned): Will be based on completion time minus penalties for hints used

## Development Status

### Completed ✅
- Database schema design and table creation (8 core tables)
- Table relationships and indexes
- Security rules (ACLs)
- **Automated puzzle generation algorithm** (`MiniCrossword` Script Include)
  - Backtracking constraint satisfaction approach
  - **Variable-length word support** (2-5 letters)
  - **Cross-word validation** - ALL formed words must be valid
  - **Comprehensive word extraction** - Automatically records intentional + cross-words
  - **True randomization**:
    - Random puzzles: Timestamp-based seed for unique results
    - Daily puzzles: Date-hash seed for consistency
  - Intelligent word placement with intersection validation
  - Supports category filtering or all-word puzzles
  - Minimum 3 words, allows blank spaces
  - First row guarantee
- **Puzzle validation logic**
- **Full MiniCrossword Script Include with JSDoc documentation**
- **Interactive Service Portal Widget** (`minicrossword`)
  - Auto-loads random puzzle on page load
  - 5x5 interactive grid with classic crossword design
  - Keyboard navigation (letters, arrows, backspace)
  - Real-time answer validation (green=correct, red=incorrect)
  - Completion detection with success message
  - Clue display organized by direction (Across/Down)
  - Mobile-responsive design (desktop/tablet/mobile)
  - Black/white square rendering for blank vs. fillable cells
- Import template for word data

### To Do 📋
- Timer functionality
- Hint reveal logic (reveal word, reveal letter)
- Hint usage tracking with penalties
- Leaderboard ranking algorithm
- Leaderboard display widget
- Daily puzzle scheduling (scheduled job to auto-generate)
- User statistics dashboard
- Game state persistence (save/resume)
- Instance record creation for gameplay tracking
- Score calculation and submission
- Puzzle picker/selector interface
- Times mode vs. Zen mode implementation

## Installation

This is a ServiceNow scoped application. To install:

1. Import the application from source control into your ServiceNow instance
2. Verify all tables and relationships are created
3. Load initial word data via import set or manual entry
   - Each word needs a `definition` field filled (used as the clue)
   - Words can be 2-5 letters long
4. Create word categories (optional, for themed puzzles)
5. Generate puzzles using the `MiniCrossword` Script Include (see usage examples below)
6. Create a Service Portal page and add the `minicrossword` widget:
   ```html
   <widget id="minicrossword"></widget>
   ```
7. Navigate to the page - widget will auto-load a random puzzle

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

### For Players
1. Navigate to the Mini Crossword Service Portal page
2. A random puzzle loads automatically
3. **Controls**:
   - **Click** any white cell to select it
   - **Type letters** (A-Z) to fill in answers - automatically advances to next cell
   - **Backspace** to delete and move back
   - **Arrow keys** to navigate up/down/left/right
4. **Visual Feedback**:
   - Yellow highlight = currently selected cell
   - Green cell = correct letter
   - Red cell = incorrect letter
5. **Clues** are displayed below the grid, organized by direction (Across/Down)
6. Complete the puzzle to see the success message!

**Note**: Timer, scoring, and hint features are coming soon!

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
- **Variable-Length Words**: Supports 2-5 letter words in the same puzzle
- **Cross-Word Validation**: ALL horizontal and vertical letter sequences must be valid dictionary words
- **Comprehensive Word Recording**: Automatically detects and records all words including those formed by intersections
- **Intersection Validation**: Ensures words share matching letters at crossing points
- **True Randomization**:
  - Random puzzles: Uses timestamp for unique puzzles each load
  - Daily puzzles: Uses date hash for consistent daily results
- **Retry Logic**: Up to 10 attempts with different word shuffles if generation fails
- **Flexible Layout**: Allows blank spaces when no compatible words fit
- **Minimum Threshold**: Requires at least 3 words for a valid puzzle
- **Category Support**: Can filter by word category or use entire word database

**Algorithm Constraints:**
- Grid size: 5x5
- Word length: 2-5 letters (variable)
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
- **Phase 3**: Develop Service Portal widget ✅ **COMPLETED**
- **Phase 4**: Implement game logic and user validation ✅ **COMPLETED** (real-time validation working)
- **Phase 5**: Add timer and scoring ⏳ **UPCOMING**
- **Phase 6**: Create leaderboard displays ⏳ **UPCOMING**
- **Phase 7**: Implement daily puzzle automation (scheduled jobs) ⏳ **UPCOMING**
- **Phase 8**: Mobile optimization ✅ **COMPLETED** (responsive design implemented)
- **Phase 9**: Analytics and user statistics ⏳ **UPCOMING**

### Recent Updates

**2025-11-05**:
- ✅ Completed Service Portal widget (`minicrossword`) with full interactivity
- ✅ Implemented keyboard navigation (letters, arrows, backspace)
- ✅ Added real-time answer validation with visual feedback
- ✅ Created mobile-responsive design with classic crossword styling
- ✅ Improved puzzle generation with variable-length word support (2-5 letters)
- ✅ Added cross-word validation ensuring ALL formed words are valid
- ✅ Implemented automatic cross-word detection and recording
- ✅ Fixed randomization to use timestamp for truly unique random puzzles
- ✅ Changed clue source from `hint` to `definition` field

**2025-10-22**:
- ✅ Implemented complete puzzle generation algorithm with backtracking
- ✅ Added puzzle validation logic for intersection checking
- ✅ Created comprehensive JSDoc documentation for MiniCrossword Script Include
- ✅ Added support for category-filtered and all-word puzzle generation
- ✅ Implemented retry logic with seeded randomization for puzzle variety
