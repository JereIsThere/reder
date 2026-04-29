# reder

A single-file chat UI that talks to an [n8n](https://n8n.io) webhook of your choice.

- Vanilla HTML / CSS / JS — no build step, no dependencies.
- Markdown rendering, code blocks with copy, image attachments (paste or pick), session rename, mobile drawer, Markdown export.
- Multiple sessions persisted in `localStorage`.
- Slash commands `/image` and `/video` if your n8n flow supports them.

## Use

Open `index.html`, paste your webhook URL into the sidebar, talk.

## Deploy

GitHub Pages on this repo serves it live: <https://jereisthere.github.io/reder/>

## n8n contract

The page POSTs JSON to your webhook:

```json
{
  "sessionId": "s_xxxx",
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
