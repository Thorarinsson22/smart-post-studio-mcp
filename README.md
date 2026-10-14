# Smart Post Studio MCP — connect your AI to your social media queue

[Smart Post Studio](https://smartpoststudio.com) is a social media scheduler for 8 platforms (Instagram, Threads, TikTok, X, Facebook, YouTube, LinkedIn incl. Company Pages, Fanvue). It exposes a remote **[MCP](https://modelcontextprotocol.io) server** so any MCP-capable assistant — Claude, Claude Code, Cursor, VS Code + Copilot, Gemini CLI — can read your queue, draft posts and schedule them.

This repo is the public documentation and example configs for that server. The server itself is hosted; there is nothing to install or run.

```
Server URL:  https://app.smartpoststudio.com/v1/mcp
Transport:   Streamable HTTP
Auth:        OAuth (browser sign-in) or Bearer token
Plan:        included on every paid plan, from $8/mo · 14-day free trial, no card
```

Full guide: **https://smartpoststudio.com/mcp**

---

## 60-second setup

### Claude (claude.ai or Claude Desktop)
Settings → Connectors → **Add custom connector**
- Name: `Smart Post Studio`
- URL: `https://app.smartpoststudio.com/v1/mcp`
- Leave OAuth Client ID / Secret empty

Click Add → sign in with your Smart Post Studio email/password → choose what the AI may do (**read**, **draft**, **schedule**) → Allow.

### Claude Code
```bash
claude mcp add sps --transport http https://app.smartpoststudio.com/v1/mcp
```
Then run `/mcp` and finish the sign-in in your browser.

### Cursor / VS Code / any client that uses headers
Create a token in the web app → **AI access → Create token** (shown once), then see [`examples/`](examples/) for a ready-made config:

```json
{ "mcpServers": { "sps": {
  "url": "https://app.smartpoststudio.com/v1/mcp",
  "headers": { "Authorization": "Bearer YOUR_TOKEN" } } } }
```

### Test it
Ask: *"What's in my Smart Post Studio queue?"*

---

## Tools

| Tool | What it does | Notes |
|---|---|---|
| `get_connected_platforms` | Lists the platforms you've connected and whether each publishes from the cloud or from your desktop app | The AI can never connect a platform itself |
| `list_posts` | Reads your queue, filterable by `status` (e.g. `scheduled`) | Newest first |
| `create_post` | Creates a draft or scheduled post: `caption`, `platforms[]`, optional `media_urls[]`, optional `scheduled_at` | Per-platform limits enforced server-side (X ≤ 280 chars, Threads ≤ 500, Instagram requires media) |
| `update_post` | Edits caption, platforms, media or time | Requires `expected_updated_at` — optimistic locking, so two agents can't clobber each other |
| `cancel_post` | Returns a scheduled post to drafts | Not a delete |
| `get_post_results` | Per-platform publish result for a post | Use it for "did yesterday's post go out?" |

Platform keys: `instagram`, `threads`, `tiktok`, `x`, `facebook`, `youtube`, `linkedin` (personal), `linkedin_page` (Company Page), `fanvue`.

## Guardrails (by design)

- **There is no "post now" tool.** Everything an AI creates is a draft or a scheduled post at least **10 minutes** out, visible in your Queue with an AI badge. That is a window to intervene, not an approval step.
- **Permissions are per connection:** read / draft / schedule. Start with read + draft.
- **Rate limits:** 30 requests/min, 100 AI-created posts/day.
- **Media by URL only**, public https, ≤ 25 MB each.
- **Tokens** last 90 days, max 5 per account, revocable instantly under AI access.
- A single post can't mix cloud-published and desktop-published platforms (`mixed_fire_platforms`) — split it in two.
- Every action is written to your account history.

## Error reference

| Error | Meaning |
|---|---|
| `mcp_not_enabled` | No active paid plan on the account |
| `platform_not_connected` | Connect it in the web or desktop app first |
| `scheduled_at_too_soon` | Pick a time ≥ 10 minutes ahead |
| `caption_too_long` | Over the platform limit (X 280 raw characters, Threads 500) |
| `mixed_fire_platforms` | Split into one cloud post and one desktop post |
| `rate_limited` | Wait a minute and retry |
| `media_url_too_large` | Image/video over 25 MB |
| `unauthorized` | Token revoked or expired — create a new one |

## Example prompts

See [`examples/prompts.md`](examples/prompts.md) for a full week. The short version:

- *"What's scheduled across all my platforms for the next seven days? Group it by day and flag any empty day."*
- *"Draft posts about [topic] for Instagram, Threads and LinkedIn — same idea, different openings. Keep Threads under 500 characters. Save as drafts, don't schedule."*
- *"Schedule the Instagram draft for Tuesday 10:00 with this image: https://…"*
- *"Move that post to Friday."* / *"Cancel it."*
- *"Did yesterday's post go out everywhere?"*

## Which clients can write?

Tested: Claude (all tiers incl. free), Claude Code, Cursor — read + write. ChatGPT: full MCP including writes is Business/Enterprise/Edu only per OpenAI's docs; Pro is read-only. Details and dates: [Which AI can actually schedule your posts over MCP?](https://smartpoststudio.com/blog/mcp-clients-for-social-media-scheduling-2026)

## Links

- Setup guide: https://smartpoststudio.com/mcp
- 7 copy-paste prompts: https://smartpoststudio.com/blog/claude-prompts-for-social-media-mcp
- How-to with every tool that supports MCP: https://smartpoststudio.com/blog/social-media-schedulers-with-mcp-2026
- Pricing: https://smartpoststudio.com/pricing
- Support: support@smartpoststudio.com

## License

Documentation and examples in this repository: MIT. The Smart Post Studio service is proprietary.
