# @openclaw/mezon

Mezon chat platform plugin for OpenClaw.

🌐 [Mezon Homepage](https://mezon.ai) • 💻 [GitHub](https://github.com/mezonai/mezon) • 📚 [OpenClaw Docs](https://docs.openclaw.ai/channels/mezon)

## What is Mezon?

[Mezon](https://mezon.ai) is a modern, high-performance team communication platform designed for live, work, and play. Positioned as a Discord alternative, Mezon offers:

- **End-to-end encryption** for all communications
- **Cross-platform support**: Web, Desktop (Windows/macOS/Linux), Mobile (iOS/Android)
- **Built-in AI capabilities** including real-time translation (100+ languages) and content moderation
- **High performance**: Sub-millisecond response times, support for millions of concurrent connections
- **Rich features**: HD voice/video (up to 1000 users), file sharing (up to 500MB), screen sharing, threads

This plugin integrates Mezon's bot SDK with OpenClaw, enabling your AI agents to communicate seamlessly across Mezon's clan channels, DMs, and threads.

---

## Features

This extension enables your OpenClaw bot to:

- ✅ **Direct Messages**: Receive and send encrypted DMs with pairing-based access control
- ✅ **Clan Channels**: Participate in team channels with mention-gating
- ✅ **Threads**: Thread-aware conversations with independent session contexts
- ✅ **Media Support**: Handle images, audio, video, and documents up to 500MB
- ✅ **Streaming Responses**: Real-time message streaming with intelligent coalescing
- ✅ **Multi-Account**: Run multiple bot accounts with independent configurations
- ✅ **Security**: Pairing workflow, allowlists, and role-based access control
- ✅ **Echo Prevention**: Smart deduplication to prevent bot message loops

---

## Installation

### Option 1: From npm (when published)

```bash
openclaw plugins install @openclaw/mezon
```

### Option 2: From local checkout

```bash
openclaw plugins install ./extensions/mezon
```

### Option 3: Via onboarding

Select **Mezon** during onboarding and confirm the install prompt.

---

## Quick Start

### 1. Create a Mezon Bot

1. Visit the [Mezon Developer Portal](https://mezon.ai/developers/applications)
2. Sign in with your Mezon account
3. Click **New Application** and give it a name
4. Navigate to the **Bot** section
5. Copy the **Bot Token** and **Bot ID**

### 2. Configure OpenClaw

**Minimal setup (recommended for testing):**

```json5
{
  channels: {
    mezon: {
      enabled: true,
      token: "your-bot-token",
      botId: "your-bot-id",
      dmPolicy: "pairing"  // Safe default: requires approval
    }
  }
}
```

**Using environment variables (recommended for production):**

```bash
export MEZON_TOKEN="your-bot-token"
export MEZON_BOT_ID="your-bot-id"
```

```json5
{
  channels: {
    mezon: {
      enabled: true
    }
  }
}
```

### 3. Start the Gateway

```bash
openclaw gateway run
```

The bot automatically connects to Mezon when a valid token is detected.

### 4. Test the Integration

1. Add your bot to a Mezon clan/channel
2. Send a DM or @mention the bot in a channel
3. For DMs, approve the pairing code:
   ```bash
   openclaw pairing list mezon
   openclaw pairing approve mezon <CODE>
   ```

---

## Access Control

Mezon integration provides flexible security controls to protect your bot from unauthorized access.

### DM Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `pairing` (default) | Unknown users receive a pairing code; admin must approve | **Recommended**: Secure access for private bots |
| `allowlist` | Only users in `allowFrom` can message | Strict team environments |
| `open` | Anyone can message the bot | Public bots (use with caution) |
| `disabled` | DMs are completely disabled | Channel-only bots |

### Group Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `allowlist` (default) | Only users in `groupAllowFrom` can trigger | Controlled team access |
| `open` | Any member can trigger (mention-gating still applies) | Public channels |
| `disabled` | Group messages are disabled | DM-only bots |

### Mention Gating

**Default**: `requireMention: true`

The bot only responds in clan/group channels when explicitly @mentioned. This prevents unwanted triggers in busy channels.

### Example Configurations

**Production team setup** (strict security):

```json5
{
  channels: {
    mezon: {
      enabled: true,
      token: "${MEZON_TOKEN}",
      botId: "${MEZON_BOT_ID}",
      dmPolicy: "allowlist",
      allowFrom: ["alice", "@bob", "user123"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["alice", "bob"],
      requireMention: true
    }
  }
}
```

**Community bot** (open DMs, mention-gated channels):

```json5
{
  channels: {
    mezon: {
      enabled: true,
      token: "${MEZON_TOKEN}",
      botId: "${MEZON_BOT_ID}",
      dmPolicy: "open",
      allowFrom: ["*"],
      groupPolicy: "open",
      requireMention: true
    }
  }
}
```

**Multi-account** (work + personal):

```json5
{
  channels: {
    mezon: {
      enabled: true,
      accounts: {
        work: {
          name: "Work Bot",
          token: "${MEZON_WORK_TOKEN}",
          botId: "${MEZON_WORK_BOT_ID}",
          dmPolicy: "allowlist",
          allowFrom: ["alice", "@bob"]
        },
        personal: {
          name: "Personal Assistant",
          token: "${MEZON_PERSONAL_TOKEN}",
          botId: "${MEZON_PERSONAL_BOT_ID}",
          dmPolicy: "pairing"
        }
      }
    }
  }
}
```

---

## Capabilities

| Feature | Status | Notes |
|---------|--------|-------|
| Direct messages | ✅ Supported | Pairing-gated by default |
| Clan channels | ✅ Supported | Mention-gated by default |
| Threads | ✅ Supported | Thread-aware session keys |
| Media (images, audio, video, docs) | ✅ Supported | Up to 500MB via URL embedding |
| Reactions | ✅ Supported | Full emoji support |
| Streaming responses | ✅ Supported | With intelligent coalescing |
| Message deduplication | ✅ Supported | 5-minute TTL, 2000 message cache |
| Echo loop prevention | ✅ Supported | Bot user ID + sent message tracking |
| Typing indicators | ❌ SDK limitation | Not available |
| Slash commands | ❌ SDK limitation | Text commands only |

---

## Programmatic Message Delivery

Send messages from CLI or cron jobs using target formats:

```bash
# Send to a user (DM)
openclaw message send --channel mezon --target user:12345abc --message "Hello!"

# Send to a channel
openclaw message send --channel mezon --target channel:67890def --message "Deploy complete"

# Alternative formats
--target @12345abc           # DM to user
--target mezon:12345abc      # DM to user
--target #67890def           # Channel
--target 67890def            # Channel (default)
```

**Finding IDs**: Check gateway logs (`openclaw logs --follow`) — each inbound message includes sender user ID and channel/thread ID.

---

## Configuration Reference

### Core Options

| Option | Type | Default | Required | Description |
|--------|------|---------|----------|-------------|
| `enabled` | boolean | `true` | No | Enable/disable channel |
| `token` | string | - | **Yes** | Bot token from Developer Portal |
| `botId` | string | - | **Yes** | Bot ID from Developer Portal |
| `dmPolicy` | string | `"pairing"` | No | `pairing` \| `allowlist` \| `open` \| `disabled` |
| `allowFrom` | string[] | `[]` | No | DM allowlist (user IDs or usernames) |
| `groupPolicy` | string | `"allowlist"` | No | `allowlist` \| `open` \| `disabled` |
| `groupAllowFrom` | string[] | `[]` | No | Group allowlist (user IDs) |
| `requireMention` | boolean | `true` | No | Require @mention in groups |
| `name` | string | - | No | Display name for account |

### Advanced Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `textChunkLimit` | number | `4000` | Max characters per outbound message chunk |
| `chunkMode` | string | `"length"` | Chunking strategy: `length` \| `newline` |
| `blockStreaming` | boolean | `false` | Disable streaming responses |
| `blockStreamingCoalesce.minChars` | number | `1500` | Min chars before sending streamed block |
| `blockStreamingCoalesce.idleMs` | number | `1000` | Idle milliseconds before sending block |
| `configWrites` | boolean | `true` | Allow channel-initiated config writes |

**Multi-account**: Use `accounts.<id>.*` to override any option per account.

---

## Troubleshooting

### Bot doesn't respond to messages

**Checklist:**
1. Verify token validity: `openclaw channels status --probe`
2. Check sender is approved (pairing or `allowFrom`)
3. Ensure bot is added to the clan/channel as a member
4. Check gateway logs: `openclaw logs --follow`
5. Verify `dmPolicy` is not `disabled`

### Bot ignores group messages

**Checklist:**
1. Ensure you are @mentioning the bot (`requireMention: true` by default)
2. Check `groupPolicy` is not `disabled`
3. Confirm sender is in `groupAllowFrom` (if using `allowlist` policy)
4. Verify bot has proper clan membership

### Pairing codes not working

**Solutions:**
- Codes expire after 1 hour — reissue: `openclaw pairing list mezon`
- Ensure user sends code as DM, not in a group channel
- Check pairing queue: `openclaw pairing list mezon`

### Media not delivered

**Solutions:**
- Confirm attachment URL is publicly accessible
- Check gateway logs for media download errors
- Verify media size is under 500MB (Mezon platform limit)

### Streaming feels delayed

**Solution**: Reduce coalescing thresholds for faster response:

```json5
{
  channels: {
    mezon: {
      blockStreamingCoalesce: {
        minChars: 500,   // Send sooner (default: 1500)
        idleMs: 500      // Shorter idle window (default: 1000)
      }
    }
  }
}
```

Or disable streaming entirely: `blockStreaming: true`

### Token rejected on startup

**Solutions:**
- Run `openclaw channels status --probe` to verify token
- Ensure token matches Developer Portal (no whitespace)
- Verify `MEZON_TOKEN` env var is exported in gateway shell
- Check `botId` matches the Bot ID from Developer Portal

---

## Architecture & Implementation

### Technical Details

- **SDK**: Built on [mezon-sdk](https://github.com/mezonai/mezon) for WebSocket-based real-time messaging
- **Session Management**: Session keys derived from account ID + channel ID + thread ID (hashed)
- **Deduplication**: 5-minute TTL cache holding up to 2000 recent message IDs
- **Echo Prevention**: Dual-layer protection via bot user ID filtering and sent message tracking
- **Streaming**: Intelligent coalescing prevents chat flooding while maintaining responsiveness
- **Security**: Envelope sanitization, access control policies, pairing workflow

### Platform Compatibility

- **Web**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Desktop**: Windows 10+, macOS 10.15+, Ubuntu 18.04+
- **Mobile**: iOS 13.0+, Android 8.0+ (API level 26+)

---

## Resources

### Mezon Platform

- **Homepage**: https://mezon.ai
- **Documentation**: https://mezon.ai/docs/
- **Developer Portal**: https://mezon.ai/developers/applications
- **GitHub Repository**: https://github.com/mezonai/mezon
- **SDK Documentation**: https://mezon.ai/docs/mezon-sdk-docs/

### OpenClaw Documentation

- **Full Channel Docs**: https://docs.openclaw.ai/channels/mezon
- **Plugin System**: https://docs.openclaw.ai/plugin
- **Pairing Workflow**: https://docs.openclaw.ai/start/pairing
- **Multi-Account Setup**: https://docs.openclaw.ai/gateway/configuration

---

## Security Best Practices

1. **Never commit tokens** — use environment variables (`MEZON_TOKEN`, `MEZON_BOT_ID`)
2. **Use pairing or allowlist** in production (avoid `open` policy unless necessary)
3. **Enable mention-gating** for channels (`requireMention: true`)
4. **Regularly review** pairing approvals: `openclaw pairing list mezon`
5. **Monitor logs** for unauthorized access attempts: `openclaw logs --follow`
6. **Leverage E2E encryption** — all Mezon communications are encrypted by default

---

## Contributing

This extension follows OpenClaw's plugin architecture and coding standards:

- **Language**: TypeScript (strict mode)
- **Linting**: Oxlint + Oxfmt
- **Testing**: Vitest with 70%+ coverage
- **Docs**: Comprehensive README + CHANGELOG + inline comments

See [CHANGELOG.md](./CHANGELOG.md) for release history.

---

## License

MIT

---

**Ready to use Mezon with OpenClaw?** Install the plugin and follow the Quick Start guide above. For advanced configuration and troubleshooting, see the [full documentation](https://docs.openclaw.ai/channels/mezon).
