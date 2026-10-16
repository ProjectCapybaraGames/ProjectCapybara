# Capybara Games — Full-Stack Multiplayer & Gameplay Expansion Plan

Comprehensive architectural and design plan to upgrade Capybara Games with authentic in-game thumbnails, an expanded 2D Bee Swarm simulator, fluid sprite and game animations across all 10 mini-games, a full-stack Express server storing numbered user accounts to file (`data/accounts.json`), a real-time Friends & Chat system with mutual request acceptance, Google Sign-In, and email password recovery codes.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> The following architectural decisions were confirmed in Phase 1:
> - **Account Storage & Server**: Full-stack Express server mounting Vite middleware in dev (`server.ts`) with a file-backed database (`data/accounts.json`) indexing numbered user IDs (Account #1, Account #2300, etc.).
> - **Bee Swarm Expansion**: All four key mechanics requested will be integrated: Bear NPC (Black Bear & Mother Bear) with quests and honey rewards, hostile Beetle/Ladybug mobs with combat animations, Sprinklers/Field Boosts for accelerated pollen gathering, and Rare/Gifted bees with special boost abilities.
> - **Authentication & Password Recovery**: Native Google Sign-In integration alongside standard registration with email recovery code generator for password resets.

---

## 1. Overview & Core Concept

Capybara Games evolves from a single-device browser portal into a full-stack social indie gaming hub:
- **Accurate In-Game Thumbnails**: Every game card receives a newly designed, high-precision SVG artwork thumbnail reflecting exact gameplay mechanics, color schemes, and characters (e.g. true perspective Cube Canyon road, authentic Bee Swarm field layout, Capy Blaster starship cockpit, and Pond Snake clover tiles).
- **Expanded Bee Swarm Simulator**: A massive expansion to the 2D top-down farming game adding a friendly Quest Bear NPC, roaming Beetle/Ladybug mobs with combat health bars and bee attack stingers, field sprinklers with pollen regeneration rings, and hatching rare/gifted bees.
- **Vibrant Game Animations**: Smooth sprite walk cycles, coin spin frames, enemy death squash/stretch, paddle hit deformations, ball speed trails, and floating popups across all 10 titles.
- **Numbered Accounts & Friends System**: File-backed numbered player IDs (`Account #1`, `Account #2`, ... `Account #2300`). Players can search friends by username or ID, send pending friend requests, accept/decline, and exchange real-time chat messages.
- **Google Sign-In & Email Password Reset**: One-click Google sign-in support and verified email association with a 6-digit verification code reset flow.

---

## 2. User Experience & Visual Design

### Key User Flows

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────────┐
│  Top Navigation │ ────> │  Numbered Account    │ ────> │ Friends & Chat Tab  │
│  · Discover     │       │  · User #1, #2, etc. │       │ · Search Player ID  │
│  · Game Library │       │  · Google / Password │       │ · Pending Requests  │
│  · Friends Tab  │       │  · Email Recovery    │       │ · Real-time Chat    │
└─────────────────┘       └──────────────────────┘       └─────────────────────┘
        │
        ▼
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────────┐
│ 10 Accurate     │ ────> │ Fullscreen GameModal │ ────> │ In-Game Animations  │
│ Game Thumbnails │       │ · Pause, HUD, VFX    │       │ · Combat, Walk-Cycle│
│ (SVG Artworks)  │       │ · Bee Shop & Quests  │       │ · Squash & Stretch  │
└─────────────────┘       └──────────────────────┘       └─────────────────────┘
```

1. **Friends Tab & Messaging**:
   - New top-level navigation item: `"Friends"` (with a badge indicating pending friend requests).
   - Search bar to find users by exact username or numbered ID (e.g., `#1` or `capy_king`).
   - `"Send Friend Request"` button.
   - Incoming Requests tab with `"Accept"` and `"Decline"`.
   - Once accepted, friends appear in the Active Friends list showing online status and furthest game stages.
   - Clicking a friend opens a dedicated conversation drawer to chat back and forth in real-time.
2. **Account Sign-In & Password Reset Flow**:
   - Form modal with tabs: `"Log in"`, `"Register"`, and `"Sign in with Google"`.
   - Registration assigns the next sequential integer ID (e.g., ID: 1, 2, ...).
   - `"Forgot Password?"` link prompts for registered email, generates an active recovery security code, and updates the password securely.
3. **Accurate Thumbnails & Card Presentation**:
   - Replaced all placeholder SVG art with game-accurate vector artwork matching the actual canvas dimensions, colors, and player models.
   - Clean unboxed metadata badges (`Category · Subgenre · # Stages`).
4. **Expanded Bee Swarm UI**:
   - In-game Bear NPC standing near the Hive with an interactive `E — Talk to Bear` indicator.
   - Active Quest HUD (`"Bear Quest: Collect 500 Sunflower Pollen & Defeat 1 Beetle"`).
   - Sprinkler placement button (`Spacebar` to place a sprinkler in fields).
   - Hostile Beetle mobs emerging in the Pine Tree and Mushroom fields that your bees actively swarm and attack!

---

## 3. Key Product Decisions & Trade-Offs

- **Full-Stack Express with Vite Middleware**:
  - *Decision*: Convert `server.ts` into a unified Express app serving API endpoints (`/api/accounts`, `/api/friends`, `/api/chat`, `/api/auth`) and mounting Vite middlewares in dev mode.
  - *Why*: Provides real persistent server-authoritative state across browser refreshes, handles numbered account allocation reliably on disk, and fulfills user's explicit preference.
- **Disk Persistence Format (`data/accounts.json` & `data/chats.json`)**:
  - *Decision*: Clean JSON files formatted with atomic write backups to persist numbered user IDs, passwords, emails, friends lists, and messages.
  - *Why*: Exact fulfillment of user request ("accounts stored on a file as a number so if i make account 1 then my user id in the file would be 1").
- **Real-Time Friends & Chat Protocol**:
  - *Decision*: REST endpoints with short-interval polling + server-sent events / optimistic updates for low latency, zero build friction, and rock-solid reliability in the AI Studio preview environment.
- **Rich Procedural In-Game Animation**:
  - *Decision*: Build animation tick frames directly into the game update/render cycles (sine-wave leg rotations, projectile squash/stretch, enemy death particle flurries, and bee wing flutter).

---

## 4. Technical Architecture & Data Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                    React Client (Port 3000)                 │
│ ┌───────────────┐ ┌────────────────┐ ┌────────────────────┐ │
│ │  Game Library │ │  Friends & Chat│ │  Auth & Google Sign│ │
│ │  (10 Games)   │ │  Component     │ │  In Modal          │ │
│ └───────┬───────┘ └────────┬───────┘ └─────────┬──────────┘ │
└─────────┼──────────────────┼───────────────────┼────────────┘
          │                  │                   │
          ▼                  ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                 Express Backend (server.ts)                 │
│  · GET/POST /api/auth/register (assigns sequential ID)      │
│  · POST     /api/auth/google                                │
│  · POST     /api/auth/reset-password-request & confirm      │
│  · GET/POST /api/friends/search, /request, /accept, /list   │
│  · GET/POST /api/chat/messages & /send                      │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   File Storage (data/)                      │
│  · data/accounts.json (numbered IDs: 1, 2, ... 2300)        │
│  · data/friends.json  (friendships & pending requests)      │
│  · data/messages.json (chat threads)                        │
└─────────────────────────────────────────────────────────────┘
```

### Data Models

```typescript
export interface UserAccount {
  id: number; // 1, 2, 2300...
  username: string;
  email?: string;
  passwordHash?: string;
  isGoogleAuth?: boolean;
  avatarSeed?: string;
  createdAt: string;
  resetCode?: string;
  resetCodeExpiry?: number;
  data: {
    favorites: string[];
    bestScores: Record<string, number>;
    stageBest: Record<string, number>;
    beeSave?: any;
  };
}

export interface FriendRequest {
  id: string;
  fromId: number;
  fromUsername: string;
  toId: number;
  toUsername: string;
  status: 'PENDING' | 'ACCEPTED' | 'DECLINED';
  timestamp: number;
}

export interface ChatMessage {
  id: string;
  senderId: number;
  senderUsername: string;
  recipientId: number;
  recipientUsername: string;
  text: string;
  timestamp: number;
}
```

---

## Verification Plan

### Automated Build & Compilation
1. Run `compile_applet` to verify TypeScript types, Express routes, and React component integration.
2. Run `lint_applet` to ensure zero syntax or lint errors.

### Manual Feature Walkthrough
1. **Numbered Accounts**: Register a new account and verify its numeric ID is assigned (e.g. `#1`), stored in `data/accounts.json`.
2. **Password Reset**: Add an email to an account, request a reset code, and confirm password update with the 6-digit code.
3. **Google Sign-In**: Click "Sign in with Google" to test the instant Google authentication profile linking.
4. **Friends & Chat**: Register Account #1 and Account #2 in different tabs, search Account #2 from Account #1, send friend request, switch to Account #2 to accept it, and exchange chat messages.
5. **Thumbnails**: Inspect all 10 game thumbnails on the home grid to verify game-accurate illustrations.
6. **Bee Swarm 2.0**: Open Bee Swarm, talk to the Bear NPC, accept quests, fight wandering Beetle mobs, place a sprinkler, and hatch rare/gifted bees.
7. **Game Animations**: Launch each game and verify enhanced animations (running feet, squash/stretch, laser glows, and death particle blooms).
