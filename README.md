# reder

A single-file chat UI that talks to an [n8n](https://n8n.io) webhook of your choice.

- Vanilla HTML / CSS / JS — no build step, no dependencies.
- Markdown rendering, code blocks with copy, image attachments (paste or pick), session rename, mobile drawer, Markdown export.
- Multiple sessions per user, persisted in `localStorage` and **scoped per signed-in user** (`reder.v1.<user>`), so two people sharing a browser don't see each other's chats.
- Slash commands `/image` and `/video` if your n8n flow supports them.

Live: <https://jereisthere.github.io/reder/>

## Auth

Two hardcoded users: **jere** and **patrick**. The page is gated by a client-side login that compares `SHA-256(salt + password)` against an embedded hash. This is a "keep casual visitors out" gate, not real security — anyone determined can extract the hash from the JS source and try to brute-force it. Use strong passwords and treat this as interim.

### Rotate a password

1. Pick a new password.
2. Generate a fresh salt and the matching hash:

   ```bash
   salt=$(openssl rand -hex 8)
   pass='your-new-password'
   hash=$(printf '%s' "${salt}${pass}" | openssl dgst -sha256 -hex | awk '{print $NF}')
   echo "salt=$salt hash=$hash"
   ```

3. In `index.html`, find `AUTH_USERS` and replace the user's `salt` and `hash` with the new values.
4. Commit and push — Pages will rebuild within a minute.

To force everyone to log out after rotation, also bump `SESSION_KEY` (e.g. `"reder.auth.user.v2"`).

## Use

Open `index.html`, sign in, paste your webhook URL into the sidebar, talk.

## n8n contract

The page POSTs JSON to your webhook:

```json
{
  "sessionId": "s_xxxx",
  "user": "jere",
  "model": "claude-sonnet-4-6",
  "systemPrompt": "...",
  "messages": [{"role": "user", "content": "...", "attachments": [...]}],
  "attachments": [{"name": "...", "type": "image/png", "dataUrl": "data:..."}]
}
```

It expects JSON back, shape:

```json
{
  "model": "claude-sonnet-4-6",
  "message": { "content": "...", "type": "text" }
}
```

For images/videos return `{"message": {"type": "image", "imageUrl": "..."}}` or `videoUrl`.
