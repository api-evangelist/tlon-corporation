---
name: Message and manage groups on Tlon Messenger
description: >-
  Operate Tlon Messenger through a running Urbit ship — send direct messages and
  channel posts, manage groups and channels, and read activity. Grounded in the
  real tool surface of the first-party Tlon MCP Server (tloncorp/tlon-mcp-server)
  and the @tloncorp/api client.
api: mcp/tlon-corporation-mcp.yml
method: generated
source: https://github.com/tloncorp/tlon-mcp-server
provider_published_skill: "@tloncorp/tlon-skill (OpenClaw skill; https://github.com/tloncorp/tlon-skill)"
operations:
  - send-dm
  - read-dm-history
  - list-contacts
  - list-groups
  - create-channel
  - send-to-channel
  - read-channel-history
  - get-activity
  - get-unreads
---

# Message and manage groups on Tlon Messenger

Tlon Messenger data lives on your own **Urbit ship**. All operations below run
against that ship over its HTTP (Eyre) channel API, either through the
`@tloncorp/api` TypeScript client or the **Tlon MCP Server**
(`tloncorp/tlon-mcp-server`). Ships and people are addressed by Urbit **ship IDs**
(e.g. `~sampel-palnet`); channels by their **nest** (e.g. `chat/~sampel-palnet/general`);
groups by their **flag** (e.g. `~sampel-palnet/weekend-projects`).

## Authenticate

You need three things for the target ship:

- `URBIT_URL` — the full ship URL (a Tlon-hosted ship lives under `*.tlon.network`).
- `URBIT_SHIP` — the ship name without the leading `~`.
- `URBIT_CODE` — the ship's `+code` access code.

With `@tloncorp/api`:

```ts
import { configureClient, getGroups, getContacts } from "@tloncorp/api";
configureClient({ shipName, shipUrl, getCode: async () => code });
```

The client opens an Eyre channel and authenticates with the access code to
obtain a session cookie. Do not hard-code the `+code`; source it from a secret.

## Send a direct message

1. `list-contacts` — resolve a nickname (e.g. "Brian") to a ship ID.
2. `send-dm` with `recipient` (ship ID with `~` or nickname) and `message`.
3. `read-dm-history` with `correspondent` to confirm delivery / read the thread.

## Post to a group channel

1. `list-groups` — find the group flag you belong to.
2. `list-channels` (or `get-group-info`) — pick the channel nest.
3. `send-to-channel` with `channel` (nest) and `message`.
4. `read-channel-history` to verify, then `react-to-post` / `edit-post` as needed.

## Stay on top of activity

- `get-unreads` for unread counts across all channels.
- `get-activity` with `type: mentions` (or `replies`) for notifications.

## Conventions and cautions

- Post IDs are `@ud` values written with dots (e.g. `170.141...`); pass them
  verbatim to `react-to-post`, `edit-post`, and `delete-post`.
- Writes (`send-dm`, `send-to-channel`, `delete-post`, `assign-role`) are
  irreversible from the API's perspective — confirm the target ship/channel/group
  before acting. `delete-post` and `remove-role` are destructive.
- The channel API is **not idempotency-keyed**: a retried `send-*` posts again.
  De-duplicate at the caller before retrying.
- See `conventions/tlon-corporation-conventions.yml` for the ship auth model,
  addressing scheme, and channel-event semantics.
