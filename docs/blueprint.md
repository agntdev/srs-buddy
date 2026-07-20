# SRS Buddy — Bot specification

**Archetype:** education

**Voice:** encouraging and supportive — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for spaced repetition vocabulary learning with private decks, daily new card limits, and progress tracking. Users create/edit decks, add word-translation cards with optional examples, and complete practice sessions with adaptive SRS scheduling based on self-assessment ratings.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- language learners
- Telegram users
- self-directed students

## Success criteria

- User completes daily practice session with SRS algorithm updating card intervals
- User manages at least 3 custom decks with 50+ cards total
- System sends 95%+ scheduled reminders on time

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with quick access buttons
- **/help** (command, actor: user, command: /help) — Show help documentation
- **/decks** (command, actor: user, command: /decks) — List all user's decks with options to create/edit/import
- **/stats** (command, actor: user, command: /stats) — Show learning statistics (streak, total cards, overdue count)
- **/settings** (command, actor: user, command: /settings) — Configure daily new cards limit and reminder time
- **/newdeck** (command, actor: user, command: /newdeck) — Create a new deck with title and description
- **/import** (command, actor: user, command: /import) — Import pre-made deck template
- **/export** (command, actor: user, command: /export) — Export selected deck as template
- **/practice** (command, actor: user, command: /practice) — Start a practice session with due cards
- **Start session** (button, actor: user, callback: practice:start) — Begin daily practice session with due cards
  - inputs: selected deck
  - outputs: card question, session progress tracking
- **Add word** (button, actor: user, callback: card:add) — Initiate card creation workflow
  - inputs: word, translation, example
  - outputs: card added to deck
- **My decks** (button, actor: user, callback: decks:list) — Show user's deck management interface
  - inputs: deck selection
  - outputs: deck details, card management options
- **Show answer** (button, actor: user, callback: card:show_answer) — Reveal card's translation and example
  - inputs: none
  - outputs: card answer, rating buttons
- **Again** (button, actor: user, callback: srs:again) — Rate card as needing immediate repetition
  - inputs: none
  - outputs: interval reset, next card
- **Hard** (button, actor: user, callback: srs:hard) — Rate card as challenging but remembered
  - inputs: none
  - outputs: smaller interval increase, next card
- **Good** (button, actor: user, callback: srs:good) — Rate card as well-remembered
  - inputs: none
  - outputs: moderate interval increase, next card
- **Easy** (button, actor: user, callback: srs:easy) — Rate card as very easy to remember
  - inputs: none
  - outputs: large interval increase, next card

## Flows

### Daily practice session
_Trigger:_ /practice or 'Start session' button

1. Select deck
2. Show card question
3. User requests answer
4. Present rating buttons
5. Update SRS metrics based on rating
6. Save session progress

_Data touched:_ Card, Session, Settings

### Card creation
_Trigger:_ /newdeck or 'Add word' button

1. Request word
2. Request translation
3. Request example (optional)
4. Assign to deck
5. Save card with default SRS parameters

_Data touched:_ Card, Deck

### Deck management
_Trigger:_ /decks or 'My decks' button

1. List decks
2. Show deck details
3. Edit deck properties
4. Delete deck
5. Import/export templates

_Data touched:_ Deck

### Settings configuration
_Trigger:_ /settings

1. Show current settings
2. Update daily new card limit
3. Change reminder time
4. Save preferences

_Data touched:_ Settings

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with private learning progress
  - fields: telegram_id, language_preference, daily_limit, reminder_time
- **Deck** _(retention: persistent)_ — User's collection of cards with metadata
  - fields: title, description, language, privacy_flag, created_at
- **Card** _(retention: persistent)_ — Spaced repetition learning unit
  - fields: question, answer, example, easiness, interval, repetitions, due_date, deck_id
- **Session** _(retention: session)_ — Active practice session state
  - fields: deck_id, current_card_index, started_at
- **Settings** _(retention: persistent)_ — User's learning preferences
  - fields: daily_new_cards, reminder_hour, language
- **Stats** _(retention: persistent)_ — Learning progress metrics
  - fields: current_streak, total_cards_learned, overdue_count

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Manage pre-made deck templates
- Configure subscription features
- Set data retention policies
- Monitor system usage statistics

## Notifications

- Daily reminder at 09:00 local time
- Session completion summary
- Overdue card alerts
- Streak milestones

## Permissions & privacy

- All user data private by default
- Deck templates read-only during import
- No third-party data sharing
- Session state persists across interruptions

## Edge cases

- Empty deck handling
- Session interruption recovery
- Daily new card limit exceeded
- No due cards for practice
- Invalid card format during import

## Required tests

- End-to-end practice session with SRS rating updates
- Deck import/export roundtrip validation
- Session resumption after interruption
- Reminder scheduling accuracy
- Privacy boundary enforcement between users

## Assumptions

- Default daily new cards limit is 10
- SM-2 algorithm used for SRS scheduling
- Pre-made decks are read-only templates
- Session state persists until explicitly ended
