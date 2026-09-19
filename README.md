# SayHi

SayHi is a full-stack chat app for one-to-one conversations. You sign up, see who else is online, open a thread, and send text or pictures. New messages show up live. Unread counts sit on the sidebar until you open that chat.

The UI is a React app. The API is Node and Express. MongoDB stores users and messages. Socket.IO keeps presence and incoming messages in sync.

![SayHi signup page](docs/login.png)

The first screen is signup and login. Name, email, and password, then a short bio on the next step. After that you land in the chat layout.

## What it does

- Create an account or log in. Passwords are hashed. The client keeps a JWT in `localStorage` and sends it on API calls.
- Edit name, bio, and avatar from the profile page.
- Search the user list. Green/grey labels show online vs offline.
- Open a 1:1 thread, load history, send text or an image.
- Unread badges when a message arrives in a chat you are not looking at.
- A right-hand panel with the other person’s bio and every image from that thread.

There are no group rooms, typing indicators, or calls. It is a direct-message product.

## Tech stack

**Frontend (`client/`)**

- React 19 and Vite
- React Router for `/`, `/login`, and `/profile`
- Tailwind CSS for layout and the glass-style chat shell
- Axios for REST
- Socket.IO client for live events
- react-hot-toast for errors and success messages

**Backend (`server/`)**

- Node.js, Express 5
- Socket.IO on the same HTTP server
- MongoDB with Mongoose (`User` and `Message`)
- JWT for protected routes (`token` header)
- bcryptjs for passwords
- Cloudinary for avatars and chat images (the client sends a base64 data URL; the server uploads it)

**Hosting**

- Client and API are set up for Vercel (`client/vercel.json`, `server/vercel.json`)

## How a conversation actually works

Recruiters often ask “where is the realtime part?” Here it is, in order.

1. **Auth.** Signup and login hit `/api/auth`. The response includes a JWT. Protected routes (users, messages, profile) check that token in middleware and attach `req.user`.
2. **Socket.** After a successful auth check, the browser opens Socket.IO and passes `userId` in the handshake. The server stores `{ userId: socketId }` and broadcasts `getOnlineUsers` so every client can paint online dots.
3. **Sidebar.** `GET /api/messages/users` returns every other user plus how many unseen messages each one has sent you.
4. **Open a chat.** `GET /api/messages/:id` loads the thread both ways and marks their messages to you as seen.
5. **Send.** The client does **not** emit the message over the socket first. It `POST`s `/api/messages/send/:id`. The server writes MongoDB, uploads an image to Cloudinary if there is one, then `io.to(receiverSocket).emit("newMessage", ...)`.
6. **Receive.** If that chat is open, the UI appends the message and marks it seen. If not, the unread map for that sender goes up by one.

Disconnect removes you from the in-memory map and broadcasts the online list again. That map lives in one process, so online status is best-effort on serverless.

## Repo layout

```text
client/                 React SPA
  context/              AuthContext (token, socket, online users)
                        ChatContext (users, messages, unseen counts)
  src/pages/            Login, Home, Profile
  src/components/       Sidebar, ChatContainer, RightSidebar

server/
  server.js             Express + HTTP + Socket.IO
  models/               User, Message
  controllers/          signup, login, send message, list users
  middleware/           JWT protectRoute
  lib/                  Mongo connect, Cloudinary, token helper
```

Home is a three-column grid when someone is selected (people / thread / profile + media). On a phone it shows either the list or the chat.

## API (short)

Public: `GET /api/status`, `POST /api/auth/signup`, `POST /api/auth/login`

Auth header `token: <jwt>`: check session, update profile, list users, get/send/mark messages.

## Run it locally

You need Node 18+, a MongoDB URI (Atlas is fine), and Cloudinary keys if you want images.

`server/.env`

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

`client/.env`

```env
VITE_BACKEND_URL=http://localhost:5000
```

Then:

```bash
cd server
npm install
npm run server
```

```bash
cd client
npm install
npm run dev
```

Open the Vite URL (usually `http://localhost:5173`). Use two browsers to talk to yourself.

For a live deploy, put those keys in the Vercel dashboard (not in git). Build the client with `VITE_BACKEND_URL` pointing at the real API.

## Scripts

- Server: `npm run server` (nodemon), `npm start`
- Client: `npm run dev`, `npm run build`, `npm run preview`
