# Secure Line

A single-file, end-to-end encrypted chat app. No backend, no accounts, no database — just `index.html`, a public MQTT relay for transport, and your browser for everything that matters.

**[Live demo →](#)** https://dinesh-d19.github.io/

---

## What it does

- Text, images, files (up to 8 MB), and voice notes
- Reactions, replies/quotes, typing indicators, presence ("X is here")
- **Persistent history** — close the tab, come back later, same room + secret restores the full conversation (stored encrypted on-device via IndexedDB)
- **Offline catch-up** — if you send messages while your friend is offline, they get replayed the next time you're both online together
- **Seen / delivered receipts** — ✓ sent, ✓✓ delivered, ✓✓ (teal) seen
- Optional email invite (Web3Forms) and phone push (ntfy.sh) when someone joins
- Installable as an offline app shell via a service worker

## How it works

- **Encryption**: room name + secret are run through PBKDF2 (200k iterations, SHA-256) to derive both an AES-256-GCM key and the MQTT topic name in one slow pass. Every message is encrypted client-side before it touches the network.
- **Transport**: a public MQTT broker (`broker.emqx.io` by default) relays encrypted bytes between anyone subscribed to the same derived topic. The broker never sees plaintext, only ciphertext + rough timing/size metadata.
- **Storage**: decrypted messages are re-encrypted and cached in the browser's IndexedDB, scoped per room. This is what makes history survive a reload — it isn't stored anywhere else.
- **No server, no signaling**: two browsers with the same room + secret find each other purely because they compute the same MQTT topic.

## Setup

1. Download `index.html`.
2. (Optional) Edit the two config blocks near the top of the `<script>` tag:
   - `NOTIFY.accessKey` — a free [Web3Forms](https://web3forms.com) key to email an invite link when someone opens the line.
   - `NTFY.topic` — a private [ntfy.sh](https://ntfy.sh) topic name for phone push notifications.
   Leave both blank to skip these — the app works fully without them.
3. Push to a GitHub repo and enable **GitHub Pages** (Settings → Pages → deploy from branch).
4. Open the hosted URL, pick a room name and a secret, share both with the other person privately (not over the same channel you're securing).

> Must be served over **HTTPS** (or `localhost`) — the Web Crypto API required for encryption won't run otherwise.

## Usage

1. Enter a name, room, and secret (both sides must use the exact same room + secret).
2. Tap **Open the line**.
3. Share the invite link (top-right icon) for one-tap join, or share room/secret manually.
4. Chat. History, receipts, and reconnection are all automatic.

## Security notes

- The secret **is** the encryption key. Anyone who has it can read everything — past and future messages in that room. Share it out-of-band (in person, a call, a different app).
- The relay is public and unauthenticated. It only ever sees encrypted bytes, but message timing, size, and presence are visible to anyone who knows the topic.
- There's no forward secrecy: if the secret ever leaks, previously captured ciphertext (if someone was logging it) becomes readable too. Rotate to a new room + secret if you suspect exposure.
- On-device history is encrypted with the same room key, but it lives in the browser's storage — anyone with device access while unlocked can open the app and read it (if they also know the secret) or inspect IndexedDB directly if you don't clear it.

## Limitations

- If both people are offline at the same time, there's no server queuing messages — delivery happens next time you're both online together.
- 8 MB file size cap.
- No group chats — one room is effectively a 1:1 (or small, trusted, static-group) line.
- Depends on a third-party public broker's uptime; no SLA.

## Tech

Vanilla JS, MQTT.js (bundled inline, no CDN dependency), Web Crypto API, IndexedDB. No build step, no dependencies to install.
