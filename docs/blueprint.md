# VideoReacts Bot — Bot specification

**Archetype:** community

**Voice:** warm and encouraging — write every user-facing message, button label, error, and empty state in this voice.

A public Telegram bot that shares YouTube videos and collects user reactions, delivering real-time summaries to admins. Users reply with free-text reactions to posts, which are stored and aggregated for owner access via exports or periodic digests.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Any Telegram user
- Casual viewers

## Success criteria

- Collects user reactions to YouTube videos
- Delivers admin notifications with reaction summaries

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/post** (command, actor: admin, command: /post <YouTube URL>) — Share a new video post to the public chat
- **/setnotify** (command, actor: admin, command: /setnotify <chat_id or @username>) — Configure admin notification target
- **/digest** (command, actor: admin, command: /digest on|off) — Enable/disable reaction digest notifications
- **Reply with reaction** (message, actor: user) — User replies to a video post with a free-text reaction

## Flows

### Video posting
_Trigger:_ /post

1. Owner sends /post command with YouTube URL
2. Bot validates URL format
3. Bot fetches video metadata (title, thumbnail)
4. Bot posts video to public chat with call-to-action

_Data touched:_ VideoPost

### Reaction collection
_Trigger:_ User replies to video post

1. User sends message in reply to bot's video post
2. Bot verifies reply targets a known VideoPost
3. Bot records reaction with user metadata
4. Bot sends ephemeral confirmation message

_Data touched:_ Reaction, User

### Admin notifications
_Trigger:_ New reaction recorded

1. Bot formats reaction summary
2. Bot sends notification to configured admin target
3. If digest enabled, aggregates reactions periodically

_Data touched:_ Reaction, Admin

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **VideoPost** _(retention: persistent)_ — Shared YouTube video metadata
  - fields: video_url, title, thumbnail_url, description, post_id, timestamp
- **Reaction** _(retention: persistent)_ — User reaction to a video post
  - fields: user_id, display_name, username, reaction_text, post_id, timestamp
- **Admin** _(retention: persistent)_ — Bot owner/admin configuration
  - fields: notification_target, digest_enabled, digest_interval

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /post <YouTube URL>
- /setnotify <chat_id or @username>
- /digest on|off
- /digest interval <minutes|hours>
- /export <post_id>

## Notifications

- Admin reaction summaries
- Digest notifications (configurable)

## Permissions & privacy

- Reactions are tied to Telegram user metadata (username, display name, user ID)
- No anonymous reactions

## Edge cases

- Ignore non-reply messages to video posts
- Handle invalid YouTube URLs
- Prevent duplicate reactions from same user (per post)

## Required tests

- Verify reaction collection from public users
- Validate admin digest formatting
- Test notification target configuration

## Assumptions

- Default notification target is bot owner's private chat
- Digest disabled by default with 24-hour interval
