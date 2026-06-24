# Tactical Chess ♟️

A real-time tactical chess app with a built-in coach, online multiplayer, computer AI, and user progress tracking.

Live site: https://tactical-chess.pages.dev

---

## Services & How They Connect

```
User's Browser
     │
     ├─── Cloudflare Pages (hosting)
     │         └── serves index.html from GitHub
     │
     ├─── Firebase Auth (Google OAuth + Anonymous)
     │         └── issues uid tokens used by all other Firebase services
     │
     ├─── Firebase Realtime Database
     │         ├── games/{code}       — live game state & lobby approval
     │         └── users/{uid}        — win/loss stats, game history, trophies
     │
     └─── Firebase Analytics (optional, via measurementId)
```

### 1. Cloudflare Pages — Hosting

| | |
|---|---|
| **What it does** | Serves the static `index.html` to users |
| **Connected to** | GitHub repo `omridunour/tactical-chess` (auto-deploys on push to `master`) |
| **Dashboard** | https://dash.cloudflare.com → Pages → tactical-chess |
| **Live URL** | https://tactical-chess.pages.dev |

Every `git push` to `master` triggers a new Cloudflare Pages build automatically — no manual deploy step needed.

---

### 2. Firebase Authentication

| | |
|---|---|
| **What it does** | Handles user identity: Google sign-in and anonymous (guest) sessions |
| **Providers enabled** | Google OAuth, Anonymous |
| **Connected to** | Firebase Realtime Database (rules check `auth.uid`) |
| **Dashboard** | Firebase Console → tactical-chess-c8b9c → Authentication |

**Authorized domains** that must be listed in Firebase Console → Authentication → Settings → Authorized domains:
- `tactical-chess.pages.dev`
- `localhost` (for local development)

**Guest mode**: Users who click "שחק כאורח" get an anonymous Firebase session. They can play local and online games. After winning, a "שמור חשבון" button lets them link their anonymous session to a real Google account — preserving their game history.

---

### 3. Firebase Realtime Database

| | |
|---|---|
| **What it does** | Stores live game state for online multiplayer and user stats |
| **Region** | `europe-west1` |
| **Dashboard** | Firebase Console → tactical-chess-c8b9c → Realtime Database |

#### Database structure

```
/games
  /{gameCode}
    board[][]          — current board state
    currentPlayer      — "white" | "black"
    castlingRights     — object
    enPassantTarget    — object | null
    moveHistory[]      — array of move strings
    status             — "waiting" | "active"
    players/
      white/           — { uid, name, photo }
      black/           — { uid, name, photo } (null until approved)
    requests/
      /{uid}           — { name, photo, status: "pending"|"approved"|"rejected" }
    createdAt          — timestamp

/users
  /{uid}
    name               — display name
    email
    photo              — avatar URL
    lastSeen           — timestamp
    stats/
      wins
      losses
      draws
    gameHistory/
      /{pushId}        — { date, result, opponent, moves, reason, mode, hasTrophy }
    trophies/
      /{pushId}        — { id, date, reason, moves }
```

#### Recommended database rules

```json
{
  "rules": {
    "users": {
      "$uid": {
        ".read": "auth != null && auth.uid == $uid",
        ".write": "auth != null && auth.uid == $uid"
      }
    },
    "games": {
      "$gameId": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    }
  }
}
```

Set these in Firebase Console → Realtime Database → Rules.

---

## Game Modes

| Mode | Description |
|---|---|
| 🏠 מקומי | Two players on the same device |
| 🤖 נגד מחשב | Play as white against a minimax AI (depth 1 = easy, 2 = medium, 3 = hard) |
| 🌐 מקוון | Host creates a lobby, shares a 6-character code, approves join requests before the game starts |

---

## Local Development

```bash
# Install wrangler (Cloudflare CLI — optional, for deploying manually)
npm install

# Open in browser
open index.html
```

The app is a single `index.html` file with no build step. Firebase config is hardcoded in the file (it's safe to expose — Firebase security is enforced by database rules and authorized domains, not by keeping the config secret).

---

## Environment Variables

A `.env` file (git-ignored) holds the Firebase config for reference. The values are also hardcoded in `index.html` because the app has no server-side build step.

```
FIREBASE_API_KEY
FIREBASE_AUTH_DOMAIN
FIREBASE_DATABASE_URL
FIREBASE_PROJECT_ID
FIREBASE_STORAGE_BUCKET
FIREBASE_MESSAGING_SENDER_ID
FIREBASE_APP_ID
FIREBASE_MEASUREMENT_ID
```

> **Note:** Firebase web API keys are not secret — they identify the project, not grant admin access. Security comes from Firebase Auth rules and the authorized domains list.

---

## Deployment

Push to `master` → Cloudflare Pages auto-deploys within ~60 seconds.

```bash
git add .
git commit -m "your message"
git push
```
