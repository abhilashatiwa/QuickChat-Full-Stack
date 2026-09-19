# SayHi

Realtime one-to-one chat app with text, images, online status, and unread counts.

The repo has two apps:

- `client` — React SPA (Vite)
- `server` — Express API + Socket.IO

## Features

- Email/password signup and login (JWT)
- Profile name, bio, and avatar
- Sidebar of users with search, online/offline, and unseen-message badges
- 1:1 threads persisted in MongoDB
- Live delivery of new messages over Socket.IO
- Image messages and avatars stored on Cloudinary
- Media gallery for the open conversation

## Tech stack

- **Client:** React 19, Vite 6, React Router 7, Tailwind CSS 4, Axios, Socket.IO Client, react-hot-toast
- **Server:** Node.js, Express 5, Socket.IO, Mongoose 8, JWT, bcryptjs
- **Data:** MongoDB
- **Media:** Cloudinary
- **Deploy:** Vercel (`client/vercel.json`, `server/vercel.json`)

## Project layout

```text
.
├── client/                 # Frontend
│   ├── context/            # AuthContext, ChatContext
│   └── src/
│       ├── pages/          # Login, Home, Profile
│       └── components/     # Sidebar, ChatContainer, RightSidebar
└── server/                 # Backend
    ├── controllers/
    ├── models/             # User, Message
    ├── routes/
    ├── middleware/         # JWT protectRoute
    └── lib/                # db, Cloudinary, token helper
```

## How it works

1. The browser stores a JWT in `localStorage` and sends it as a `token` header on API calls.
2. After login, the client opens a Socket.IO connection with `userId` in the handshake query.
3. The server keeps an in-memory map of `userId → socketId` and broadcasts `getOnlineUsers`.
4. Sending a message is an HTTP `POST`. The server saves the document, uploads images if needed, then emits `newMessage` to the receiver’s socket.
5. If that chat is open, the client appends the message and marks it seen; otherwise it increments the unread badge.

There are no group chats, typing indicators, or voice/video.

## Prerequisites

- Node.js 18+
- A MongoDB database (Atlas or local)
- A Cloudinary account (for avatars and chat images)

## Environment variables

Copy the examples and fill in your own values. Do not commit real secrets.

**`server/.env`**

```env
PORT=5000
NODE_ENV=development
JWT_SECRET=replace-with-a-long-random-string
MONGODB_URI=mongodb+srv://USER:PASSWORD@HOST
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=
```

The server connects to `${MONGODB_URI}/chat-app`.

**`client/.env`**

```env
VITE_BACKEND_URL=http://localhost:5000
```

Use your deployed API URL in production.

## Run locally

Install and start the API:

```bash
cd server
npm install
npm run server
```

The API listens on `PORT` (default `5000`). Health check: `GET /api/status`.

Install and start the client (second terminal):

```bash
cd client
npm install
npm run dev
```

Open the Vite URL (usually `http://localhost:5173`), create two accounts in different browsers, and send messages between them.

## API

- `GET /api/status` (no auth): health check
- `POST /api/auth/signup` (no auth): create account
- `POST /api/auth/login` (no auth): login
- `GET /api/auth/check` (auth): current user
- `PUT /api/auth/update-profile` (auth): update name, bio, avatar
- `GET /api/messages/users` (auth): users for sidebar and unseen counts
- `GET /api/messages/:id` (auth): thread with user; marks incoming as seen
- `PUT /api/messages/mark/:id` (auth): mark one message seen
- `POST /api/messages/send/:id` (auth): send text and/or image

Protected routes expect header `token: <jwt>`.

## Scripts

**Server:** `npm run server` (nodemon), `npm start` (node)

**Client:** `npm run dev`, `npm run build`, `npm run preview`

## Deploy notes

- Client: static Vite build on Vercel; SPA rewrites are in `client/vercel.json`.
- Server: Node entry `server.js`. In production it does not call `listen` itself; Vercel uses the exported HTTP server.
- Set the same env vars on the host. Point `VITE_BACKEND_URL` at the live API **before** building the client.
- Socket.IO uses an in-memory `userSocketMap`. Online status will not stay consistent across multiple serverless instances.

## License

Use and modify this project for your own learning and deployment. If you publish a copy, keep third-party licenses for the libraries listed in each `package.json`.
