# Habit Buddy — Bot specification

**Archetype:** workflow

**Voice:** warm and encouraging — write every user-facing message, button label, error, and empty state in this voice.

A private, per-user Telegram bot that helps users build and maintain habit streaks with gentle reminders and one-tap tracking. Users create habits with custom schedules, receive local-time reminders, and track progress through one-tap actions, milestone achievements, and weekly summaries.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual Telegram users who want a lightweight, private habit tracker with minimal friction and an encouraging tone.

## Success criteria

- Users can create and manage habits with one-tap tracking
- Users receive timely local-time reminders for their habits
- Users see accurate streak tracking and milestone celebrations
- Weekly summaries are delivered on schedule with progress insights

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with active and paused habit lists
- **New Habit** (button, actor: user, callback: habit:new) — Start the habit creation flow
  - inputs: Habit title, Schedule type, Reminder time(s), Optional description
  - outputs: New habit added to active list
- **Weekly Summary** (button, actor: user, callback: summary:view) — View the most recent weekly summary
  - inputs: User preferences for summary period
  - outputs: Weekly summary card with progress stats
- **Settings** (button, actor: user, callback: settings:view) — Access timezone, notification, and milestone preferences
  - inputs: Timezone selection, Notification preferences, Milestone preferences
  - outputs: Updated user settings

## Flows

### Onboarding
_Trigger:_ /start (first-time user)

1. Ask for timezone selection
2. Show quick tutorial cards
3. Confirm habit tracking preferences

_Data touched:_ User

### Create Habit
_Trigger:_ New Habit button

1. Enter habit title
2. Select schedule type (Daily/Weekdays/N times/week)
3. Choose reminder time(s)
4. Add optional description
5. Save and confirm

_Data touched:_ Habit

### Track Habit
_Trigger:_ Reminder message or Manual Action

1. Show quick-action buttons: Mark Done/Skip/Fail/Pause/Edit
2. Record occurrence with status
3. Update streak metrics
4. Show confirmation message

_Data touched:_ HabitOccurrence, Metrics

### Edit Habit
_Trigger:_ Edit button

1. Select habit to edit
2. Update title/schedule/reminder times
3. Save changes
4. Confirm update

_Data touched:_ Habit

### Weekly Summary
_Trigger:_ Scheduled weekly summary time

1. Generate summary of past week's progress
2. Include per-habit stats
3. Add friendly encouragement note
4. Send summary message

_Data touched:_ Metrics

### Milestone Celebration
_Trigger:_ Streak reaches 7/14/21/30/100 days

1. Generate celebratory message
2. Show concise stats
3. Offer to continue tracking

_Data touched:_ Metrics

### Pause/Unpause
_Trigger:_ Pause button

1. Select habit to pause/unpause
2. Update paused flag
3. Confirm change

_Data touched:_ Habit

### Manual Correction
_Trigger:_ History correction request

1. Select habit and date
2. Choose new status (Done/Skipped/Failed/Unmarked)
3. Update occurrence
4. Recalculate streaks

_Data touched:_ HabitOccurrence, Metrics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with preferences
  - fields: Telegram ID, Preferred timezone, Notification preferences, Milestone preferences, Language preference
- **Habit** _(retention: persistent)_ — User-defined habit with schedule and tracking rules
  - fields: Title, Description, Schedule type, Target times, Reminder window, Paused flag, Creation date, Timezone, Milestone preferences
- **HabitOccurrence** _(retention: persistent)_ — Daily tracking record for a habit
  - fields: Habit ID, Date, Status, Action timestamp, Source (reminder/manual)
- **Metrics** _(retention: persistent)_ — Streak and completion statistics
  - fields: Current streak, Best streak, Completion percentage, Weekly summary data

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure free/paid tier limits
- Enable/disable milestone notifications
- Set default weekly summary timing
- Adjust habit creation limits
- Manage language preferences

## Notifications

- Scheduled reminders for habit tracking
- Milestone celebration messages
- Weekly summary notifications
- Manual correction confirmations

## Permissions & privacy

- All user data is private and stored securely
- No cross-user data sharing
- Users can delete their data at any time
- Data is stored in user's chosen timezone

## Edge cases

- User changes timezone after creating habits
- Multiple reminder times for a habit in different time zones
- Manual corrections for past dates
- Streak recalculation after corrections
- Handling skipped days that don't break streaks
- Milestone notifications for paused habits

## Required tests

- End-to-end habit creation and tracking flow
- Timezone-aware reminder scheduling
- Streak calculation after manual corrections
- Weekly summary generation with accurate stats
- Milestone detection and celebration messages
- Edge case: user creates 5 habits (free tier limit)

## Assumptions

- Users will prefer simple one-tap interactions over complex menus
- Most users will want daily reminders rather than multiple times per day
- Milestone notifications will be appreciated but not intrusive
- Weekly summaries will be most effective on Sunday evenings
- Free tier users will upgrade when they need more habits
