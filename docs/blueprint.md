# Crypto Price Alert Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot for private cryptocurrency price monitoring with customizable price-threshold and percentage-change alerts, on-demand price checks, quiet hours, and optional morning summaries. Users manage watchlists via inline buttons and text commands while the owner gains anonymized usage metrics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto traders
- casual crypto holders

## Success criteria

- User successfully receives first price alert
- User configures and manages 3+ watchlist items
- Owner views admin metrics dashboard

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize profile and request timezone selection
- **/price** (command, actor: user, command: /price) — Check price of specific ticker or full watchlist
  - inputs: ticker symbol
  - outputs: price data with 24h change
- **/admin** (command, actor: owner, command: /admin) — View aggregated usage metrics
  - inputs: none
  - outputs: user count, active users, top alerts
- **Add Coin** (button, actor: user, callback: watchlist:add) — Open coin selection interface
  - inputs: ticker symbol
  - outputs: updated watchlist

## Flows

### onboarding
_Trigger:_ /start

1. Request timezone selection
2. Confirm price source
3. Display main menu

_Data touched:_ user profile

### alert_management
_Trigger:_ watchlist item selection

1. Display coin details
2. Configure price threshold alert
3. Configure percent change alert

_Data touched:_ watchlist item, alert rules

### morning_summary
_Trigger:_ scheduled daily event

1. Check user's quiet hours
2. Generate price summary with changes
3. Send formatted summary

_Data touched:_ price feed, user profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User preferences and settings
  - fields: telegram_id, timezone, quiet_hours, summary_time, notification_prefs
- **watchlist_item** _(retention: persistent)_ — Monitored cryptocurrency with alert rules
  - fields: ticker, display_name, price_alerts, percent_alerts, last_alert_ts
- **price_snapshot** _(retention: session)_ — Latest price data from external source
  - fields: ticker, price, timestamp, source_status
- **owner_metrics** _(retention: persistent)_ — Aggregated usage statistics
  - fields: total_users, active_30d, top_alerts

## Integrations

- **Telegram** (required) — User messaging and inline interface
- **Crypto Price API** (required) — Price data source with retry logic
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View /admin metrics dashboard
- Configure default cooldown durations
- Select price API source

## Notifications

- Price threshold crossed alert
- Percent change alert
- Daily morning summary
- Error notifications for price unavailability

## Permissions & privacy

- All user data remains private and isolated
- Admin metrics show only aggregated counts
- No cross-user data visibility

## Edge cases

- Unknown/invalid ticker input
- Price API temporary unavailability
- Alert spam suppression during price volatility
- Timezone conversion errors

## Required tests

- End-to-end alert triggering with suppression logic
- Morning summary during quiet hours
- Watchlist management with 10+ items

## Assumptions

- Default price API selected automatically
- Timezone defaults to Telegram account setting
- Cooldown periods prevent alert spam
