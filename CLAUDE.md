# Smith Legacy 2.0

A classroom achievement tracking system for Mr. Smith's class, Season 2. Students sign up, claim achievements based on funny classroom moments, and moderators verify them.

## Architecture

- **Single-file app**: Everything lives in `index.html` — HTML, CSS, and JS in one file
- **Backend**: Firebase Firestore (client-side SDK) on project `smithlagacy2`
- **Auth**: Firebase Authentication with Email/Password. Emails are auto-generated as `username@smithlegacy.app` — students only see a username + password field
- **Hosting**: Firebase Hosting at `smithlagacy2.web.app`
- **Security**: Firestore rules enforce auth, ownership, and mod-only operations server-side

## Firebase Project

- Project ID: `smithlagacy2`
- Hosting URL: https://smithlagacy2.web.app
- Deploy command: `firebase deploy` (hosting + rules) or `firebase deploy --only hosting` (just the site)

## Key Concepts

- **Achievements**: Defined in `ACHIEVEMENTS` and `SECRET_ACHIEVEMENTS` arrays. Each has id, icon, name, desc, rarity, and category
- **Categories**: Achievements are grouped into themed sections — "Mr. Smith's Greatest Hits", "Student Spotlight", "Class Chaos", "The Outside World", "Meta"
- **Rarity levels**: Easy (green), Normal (blue), Hard (red), Extreme (purple)
- **SC Values**: Easy=25, Normal=75, Hard=150, Extreme=300 Smithcoins
- **Claims flow**: Student claims → `pendingClaims` field on user doc + doc in `claims` collection → mod verifies or denies → mod writes to `claims` field on user doc. Users CANNOT write to their own `claims` field (only `pendingClaims`)
- **Denied claims**: Show red outline, student can retry after 1 day. Denied timestamp stored in `deniedTime` map on user doc
- **Secret achievements**: Locked behind SC cost. Students spend SC to reveal, then can claim normally
- **Daily reward**: 25 SC per day, claimed on the Achievements tab

## Mod System

- **Two mod accounts**: `milesthegoat` (super mod) and `thecreator`
- Mod status is set on signup if username matches `MOD_USERNAMES` array
- Mod status is hardcoded in Firestore rules — can't be faked from console
- **Both mods can**: Post/clear announcements, set daily achievements, verify/deny claims, add/delete mod notes
- **Only milesthegoat can**: Delete accounts, reset server

## Firestore Collections

- `users` — profiles, claims (verified/denied by mods), pendingClaims (by users), smithcoins, period, etc.
- `claims` — pending approval queue (created by users, deleted by mods on verify/deny)
- `firstEarners` — tracks who first unlocked each achievement
- `announcements` — single doc `current` with banner message
- `dailyAchievement` — single doc `current` with free-text daily challenge
- `suggestions` — student-submitted achievement ideas
- `chat` — live chat messages with profanity filter
- `modNotes` — private notes between mods

## Security Rules (firestore.rules)

- All reads/writes require Firebase Auth
- Users can only create/update their own user doc
- Users CANNOT change: `isMod`, `period`, `claims` (only mods can)
- Users CAN change: `smithcoins` (>=0), `lastClaimed`, `unlockedAchievements`, `pendingClaims`
- Mod operations hardcoded to `milesthegoat@smithlegacy.app` and `thecreator@smithlegacy.app`
- Delete operations restricted to `milesthegoat@smithlegacy.app` only
- Chat messages max 200 chars, no editing, only mods can delete
- Catch-all rule denies everything not explicitly allowed

## UI Design

- Dark navy theme with gold accents
- Pill/rounded style (border-radius: 50px on most elements)
- Achievement cards: circular icon with rarity-colored border, pill-shaped card
- Top bar: logo left, centered nav tabs, SC balance + avatar + logout right
- Profile: modal popup (not a full tab)
- Chat: YouTube live chat style with avatars, floating input bar
- Announcements: show on all tabs

## Conventions

- All CSS in `<style>` block, all JS in `<script>` block
- CSS uses custom properties in `:root`
- Pages shown/hidden via `.active` class
- Tabs shown/hidden via `display:none/block`
- Forms use proper `<form>` tags with `autocomplete` attributes for browser password saving
- Chat uses Firestore `onSnapshot` for real-time updates
- Profanity filter normalizes text (number/symbol substitution) before checking against blocked word list
