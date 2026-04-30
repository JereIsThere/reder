# sprecher 👄

A single-file chat UI that talks to an [n8n](https://n8n.io) webhook of your choice. Sibling of [auge](https://github.com/JereIsThere/auge) — wo auge zeigt, spricht sprecher.

- Vanilla HTML / CSS / JS — no build step, no dependencies.
- Markdown rendering, code blocks with copy, image attachments (paste or pick), session rename, mobile drawer, Markdown export.
- Multiple sessions per user, persisted in `localStorage` and **scoped per signed-in user** (key `reder.v1.<user>` — kept stable across the rebrand so existing chats survive), so two people sharing a browser don't see each other's chats.
- Slash commands `/image` and `/video` if your n8n flow supports them.

Live: <https://reder.jeremias-groehl.de> (custom domain on the user's own server, not GitHub Pages — auto-deploy via GitHub Actions on every push to main)

> Repo and storage keys still use `reder` for stability. The codename `sprecher` is the user-facing brand only.

## Auth

Five hardcoded users: **jere**, **patrick**, **mamer**, **paper**, **daniel**. The page is gated by a client-side login that compares `SHA-256(salt + password)` against an embedded hash. This is a "keep casual visitors out" gate, not real security — anyone determined can extract the hash from the JS source and try to brute-force it. Use strong passwords and treat this as interim.

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
4. Commit and push — GitHub Actions will redeploy to the server within ~30s.

To force everyone to log out after rotation, also bump `SESSION_KEY` (e.g. `"reder.auth.user.v3"`).

### Invite link

Append `#u=<user>&p=<password>` to the URL — username and password are pre-filled and auto-submitted, then the hash is wiped from the address bar.

## Use

Open the live URL, sign in, paste your webhook URL into the sidebar, talk.

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

It expects JSON back. Plain text:

```json
{
  "model": "claude-sonnet-4-6",
  "message": { "content": "...", "type": "text" }
}
```

For media: `{"message": {"type": "image", "imageUrl": "..."}}`, or `{"type": "video", "videoUrl": "..."}`, or for parallel renders `{"type": "multi-image", "images": [{"model": "...", "imageUrl": "...", "costUsd": 0.025}, ...], "totalCostUsd": 0.065}`.
