# Server setup

Veglia's server is a single standard-library Python file. No pip install, no database.

## 1. Generate a token

```bash
head -c 24 /dev/urandom | base64
```

Put it in `server/.env` as `VEGLIA_TOKEN`. The same value goes into the phone app.

## 2. Run it

```bash
cd server
cp .env.example .env      # edit VEGLIA_TOKEN
python3 veglia_server.py
```

The server binds to `127.0.0.1:8513` by default and refuses to start with a blank token.

Run it under a process manager so it survives reboots — e.g. systemd or pm2:

```ini
# /etc/systemd/system/veglia.service
[Unit]
Description=Veglia companion server
After=network.target

[Service]
WorkingDirectory=/path/to/veglia/server
ExecStart=/usr/bin/python3 /path/to/veglia/server/veglia_server.py
Restart=always

[Install]
WantedBy=multi-user.target
```

## 3. Put TLS in front

The phone talks to the server over the internet, so terminate HTTPS with nginx or caddy and keep Veglia bound to localhost. Minimal nginx:

```nginx
location /phone/ {
    proxy_pass http://127.0.0.1:8513;
    client_max_body_size 32m;   # screenshots can be a few MB
}
```

Point the phone app's "Server address" at `https://your-domain` (no trailing slash).

## 4. Wire the screenshot to your AI (`VEGLIA_HOOK`)

When a screenshot arrives, the server can run a command of your choosing with the
image's absolute path as its single argument. This is how the shot reaches your AI
as a message it can actually open.

```bash
# .env
VEGLIA_HOOK=/home/me/notify-my-ai.sh
```

```bash
#!/bin/bash
# notify-my-ai.sh — $1 is the absolute path to the new screenshot.
# Deliver it however your AI receives messages. For example, if your AI runs
# in a tmux session, inject a line telling it to open the image:
tmux load-buffer - <<< "[peek] new screenshot: $1 — open it with your file tool and tell me what you see."
tmux paste-buffer -t my-ai-session -p
tmux send-keys -t my-ai-session Enter
```

Without a hook, the server just saves the file and logs the path.

## Endpoints (reference)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET  | `/phone/poll?token=` | token | phone pulls next command |
| POST | `/phone/peek-enqueue?token=` | token | AI enqueues a "peek" |
| POST | `/phone/screenshot?token=` | token | phone uploads a screenshot |
