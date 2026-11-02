# Chirp — a Twitter/X clone

A full-stack Twitter/X clone with a Next.js frontend and an Express + Prisma + PostgreSQL backend.

## Features

- **Home feed** with "For you" and "Following" tabs, composer, and media uploads
- **Tweet detail** pages with replies, likes, reposts, bookmarks, and share
- **Profiles** with Posts / Replies / Media / Likes tabs and follow/unfollow
- **Explore** with live post and people search
- **Notifications** (likes, replies, reposts, follows, messages) with read state
- **Direct messages** with conversation list, chat view, and new-message search
- **Auth** with JWT in an httpOnly cookie, plus login by email or username
- **Light/dark theme** with a simple Context API (lightseagreen / darkslategrey navbar)
- **Image/video uploads** through Cloudinary

## Tech Stack

| Layer | Tech |
| --- | --- |
| Frontend | Next.js 16 (App Router), React 19, JavaScript, Tailwind CSS |
| Backend | Node.js, Express 4, Prisma 5, PostgreSQL |
| Auth | JWT (httpOnly cookie), bcryptjs |
| Uploads | Cloudinary + Multer |
| Realtime | Socket.io |

## Project Structure

```
chirp-app/
├── src/                       # Next.js frontend
│   ├── app/                   # App Router pages
│   │   ├── home/
│   │   ├── explore/
│   │   ├── notifications/
│   │   ├── messages/
│   │   ├── u/[username]/      # Profile pages
│   │   ├── status/[tweetId]/  # Tweet detail
│   │   ├── login/
│   │   ├── register/
│   │   └── layout.jsx
│   ├── components/
│   │   ├── feed/              # TweetCard, TweetComposer
│   │   ├── layout/            # Sidebars, MainLayout
│   │   ├── providers/         # AuthProvider
│   │   └── ui/                # Button, Input, Modal, Avatar, Dropdown
│   ├── context/
│   │   └── ThemeContext.jsx   # Theme context from 21-context-api
│   └── lib/
│       ├── api.js             # Typed-ish API client for every endpoint
│       ├── adapters.js        # Backend → frontend data mapping
│       ├── mockData.js        # Demo trends (used for static trend lists)
│       └── utils.js
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma      # PostgreSQL models
│   │   └── seed.js            # Demo users/tweets
│   ├── scripts/db.mjs         # Start/stop the project-local PostgreSQL cluster
│   ├── src/
│   │   ├── controllers/       # auth, tweets, users, notifications, messages
│   │   ├── middleware/        # protect, optionalAuth, validate
│   │   ├── routes/            # Express routers + uploads
│   │   ├── lib/               # Prisma singleton, tweet include/formatter
│   │   └── server.js
│   └── .env
└── next.config.js             # Proxies /api/* → http://localhost:5000/api/*
```

## Getting Started

### Prerequisites

- Node.js 18.17+ (Node 20+ recommended)
- PostgreSQL installed locally (this project ships a script that manages its own
  cluster in `backend/.pgdata` on port **5433**, so it will not touch any
  existing PostgreSQL service)

### 1. Install dependencies

```bash
# Frontend (run from chirp-app/)
npm install

# Backend
cd backend
npm install
```

### 2. Configure environment

Create `backend/.env`:

```env
PORT=5000
NODE_ENV=development

DATABASE_URL="postgresql://postgres@127.0.0.1:5433/chirp?schema=public"

JWT_SECRET="replace-with-a-long-random-string"
JWT_EXPIRES_IN=7d

CLOUDINARY_CLOUD_NAME="your-cloud"
CLOUDINARY_API_KEY="your-key"
CLOUDINARY_API_SECRET="your-secret"

CLIENT_URL=http://localhost:3000
```

### 3. Set up the local database

The first time only, create the cluster (uses trust auth on localhost):

```bash
cd backend
& "C:\Program Files\PostgreSQL\18\bin\initdb.exe" -D .pgdata -U postgres --auth=trust --encoding=UTF8 --locale=C
```

Then start it, create the database, and push the schema:

```bash
npm run db:start
& "C:\Program Files\PostgreSQL\18\bin\createdb.exe" -h 127.0.0.1 -p 5433 -U postgres chirp
npx prisma db push
```

`npm run db:stop` and `npm run db:status` manage the cluster afterwards. Set
`PG_BIN` if PostgreSQL is installed somewhere other than the default path.

### 4. Seed demo data (optional)

```bash
npm run db:seed
```

Demo login: **alex@chirp.dev / password123**

### 5. Run both servers

```bash
# Terminal 1 — backend
cd backend
npm run dev

# Terminal 2 — frontend (from chirp-app/)
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

The frontend calls relative `/api/*` URLs, and `next.config.js` rewrites them to
`http://localhost:5000/api/*`, so cookies work without CORS configuration.

## API Overview

All routes are prefixed with `/api`.

| Method | Route | Auth | Description |
| --- | --- | --- | --- |
| POST | `/auth/register` | — | Create account |
| POST | `/auth/login` | — | Log in with email or username |
| POST | `/auth/logout` | — | Clear auth cookie |
| GET | `/auth/me` | ✅ | Current user |
| PUT | `/auth/me` | ✅ | Update profile fields |
| GET | `/tweets` | optional | Feed (`?following=true`, `?username=`) |
| GET | `/tweets/search?q=` | optional | Search posts |
| GET | `/tweets/:id` | optional | Single post |
| GET | `/tweets/:id/replies` | optional | Replies |
| POST | `/tweets` | ✅ | Create post or reply |
| DELETE | `/tweets/:id` | ✅ | Delete own post |
| POST | `/tweets/:id/like` | ✅ | Toggle like |
| POST | `/tweets/:id/retweet` | ✅ | Toggle repost |
| POST | `/tweets/:id/bookmark` | ✅ | Toggle bookmark |
| GET | `/users/suggestions` | optional | "Who to follow" |
| GET | `/users/search?q=` | optional | Search people |
| GET | `/users/:username` | optional | Profile + counts |
| GET | `/users/:username/tweets?tab=` | optional | posts / replies / media / likes |
| POST | `/users/:id/follow` | ✅ | Toggle follow |
| GET | `/users/bookmarks` | ✅ | Bookmarked posts |
| GET | `/notifications` | ✅ | List + unread count |
| PUT | `/notifications/:id/read` | ✅ | Mark one read |
| PUT | `/notifications/all/read` | ✅ | Mark all read |
| GET | `/messages` | ✅ | Conversations |
| POST | `/messages` | ✅ | Start conversation |
| GET | `/messages/:id` | ✅ | Messages + mark read |
| POST | `/messages/:id/messages` | ✅ | Send message |
| POST | `/upload/image` | ✅ | Upload image |
| POST | `/upload/video` | ✅ | Upload video |
| POST | `/upload/avatar` | ✅ | Upload + set avatar |
| POST | `/upload/cover` | ✅ | Upload + set cover |
| GET | `/health` | — | Server + database health |

## Scripts

Frontend (`chirp-app/`):

- `npm run dev` — Next.js dev server
- `npm run build` — production build
- `npm start` — start the production build
- `npm run lint` — ESLint

Backend (`chirp-app/backend/`):

- `npm run dev` — Express with `node --watch`
- `npm start` — production start
- `npm run db:start` / `db:stop` / `db:status` — local PostgreSQL cluster
- `npm run db:push` — push Prisma schema
- `npm run db:studio` — Prisma Studio
- `npm run db:seed` — demo data

## License

MIT — for learning and personal projects.
