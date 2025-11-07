# WhatsApp Life OS - Updates Summary

## Changes Made Based on Feedback

### 1. ✅ Local SQLite Database

**Changed from**: Deco Durable Objects SQLite (remote/cloud)  
**Changed to**: Local SQLite file in repository (`./data/whatsapp.db`)

**Rationale**:
- **Data Locality**: All WhatsApp data stays on the same machine as the bot
- **Privacy**: Sensitive WhatsApp data never leaves the local machine
- **Performance**: Direct file access is faster than remote DB calls
- **Simplified Architecture**: No need to sync data over HTTP
- **Easy Backup**: Just copy the SQLite file
- **Cost**: No database hosting costs

**Impact**:
- Bot writes directly to local SQLite
- MCP server queries SQLite via bot's HTTP API endpoints
- Frontend calls MCP server which calls bot API
- Database file is gitignored (`.gitignore` includes `/data/whatsapp.db`)

### 2. ✅ WhatsApp Web.js API Integration

**Updated based on official documentation**:

#### Authentication & Session
- Using `LocalAuth` strategy (recommended)
- Session stored in `./data/sessions/` directory
- QR code authentication via `client.on('qr')` event
- Auto-reconnect on `disconnected` event

#### Event Handling
Added all real WhatsApp Web.js events:
- `message` - New messages received
- `message_create` - Messages sent by user
- `message_ack` - Delivery status (1=sent, 2=delivered, 3=read)
- `message_revoke_everyone` - Deleted messages
- `message_revoke_me` - Revoked messages
- `message_reaction` - Reactions to messages
- `group_join` - Member joins group
- `group_leave` - Member leaves group
- `group_update` - Group info changes
- `contact_changed` - Contact updates
- `change_state` - Connection state changes

#### Media Handling
- `message.downloadMedia()` returns MessageMedia object
- Base64 data in `media.data`
- MIME type in `media.mimetype`
- Filename in `media.filename`
- 64MB file size limit
- Upload to R2 via HTTP endpoint

#### Rate Limiting
- **Critical**: 40 messages per minute limit
- Implemented queue system with delays
- Prevents WhatsApp bans

#### Message Properties
Updated schema to match real WhatsApp properties:
- `message.id._serialized` - Unique message ID
- `message.fromMe` - Boolean for own messages
- `message.type` - text, image, video, audio, document, sticker, location, etc.
- `message.hasMedia` - Boolean for media presence
- `message.timestamp` - Unix timestamp
- `message.ack` - Delivery status (0-5)

#### Group Methods
- `client.createGroup(name, participants)`
- `group.addParticipants()`
- `group.removeParticipants()`
- `group.promoteParticipants()` - Make admin
- `group.demoteParticipants()` - Remove admin
- `group.setSubject()` - Change name
- `group.setDescription()`
- `group.setPicture()`
- `group.getInviteCode()`
- `group.revokeInvite()`
- `group.leave()`

#### Interactive Messages Support
- **Buttons**: `new Buttons(text, buttons, title, footer)`
- **Lists**: `new List(text, buttonText, sections)`
- **Polls**: `new Poll(question, options)`
- **Location**: `new Location(lat, lng, description)`
- **VCards**: Send contact cards

### 3. ✅ Complete UI Layouts

Created comprehensive wireframes for all 11 views:

1. **Dashboard** - Central hub with bot status, stats, quick actions
2. **Messages View** - WhatsApp-style chat interface with conversation list
3. **Contacts View** - Full CRUD for contacts with tags and filtering
4. **Groups View** - Browse, manage, view group details
5. **Channel Rules View** - Configure allow/block rules with priorities
6. **Message Planner Dashboard** - Overview of active and completed plans
7. **Group Selection Wizard** - Filter by members/tags, visual selection
8. **Message Review** - Edit AI-generated messages before sending
9. **Sending Progress** - Real-time progress monitoring
10. **Bot Configuration** - Connection, database, security settings
11. **Analytics** (Future) - AI-powered insights placeholder

**UI Features**:
- WhatsApp-style message bubbles with delivery status
- Contact cards with profile pictures
- Group member lists
- Priority-based rule system
- Step-by-step wizard for message planning
- Side-by-side AI vs. final message comparison
- Real-time sending progress with circular indicator
- Comprehensive config panels with tabs
- Responsive design notes (mobile/tablet/desktop)
- Accessibility considerations
- Common UI patterns library

## New Architecture Diagram

```
Local Machine
┌────────────────────────────────────────────┐
│  WhatsApp Web.js Bot                       │
│  - LocalAuth session in ./data/sessions/   │
│  - Puppeteer + Chrome                      │
│  - Event listeners (message, ack, etc.)    │
│  - Rate limiter (40 msgs/min)              │
│                                             │
│  SQLite Database                            │
│  - File: ./data/whatsapp.db                │
│  - Messages, contacts, groups, etc.        │
│  - Direct writes from bot                  │
│                                             │
│  HTTP API Server (Express on :3001)        │
│  - POST /send-message (for planner)        │
│  - POST /query (for MCP server)            │
│  - GET /status                             │
│                                             │
│  R2 Media Storage                          │
│  - Images, videos, documents               │
│  - Thumbnails                              │
└─────────────────┬──────────────────────────┘
                  │ HTTPS
                  ▼
┌────────────────────────────────────────────┐
│  Deco MCP Server (Cloudflare Workers)      │
│  - Calls bot HTTP API to query SQLite      │
│  - AI generation tools                     │
│  - Message planning tools                  │
│  - Frontend views                          │
└────────────────────────────────────────────┘
```

## Bot Project Structure

```
/whatsapp-bot/
  /src/
    /bot.js              - WhatsApp Web.js client init
    /handlers/
      /message.js        - message, message_ack events
      /contact.js        - contact_changed events
      /group.js          - group_join, group_leave events
      /auth.js           - qr, authenticated events
    /database/
      /db.js             - SQLite operations
      /migrations/       - Schema migrations
    /storage/
      /media.js          - R2 upload/download
    /api/
      /server.js         - Express HTTP API
      /client.js         - MCP server client
    /config.js           - Configuration
  /data/
    whatsapp.db          - SQLite database
    /sessions/           - WhatsApp sessions (LocalAuth)
  package.json
  .env
  .gitignore             - Ignore sessions/ and data/
```

## Key Implementation Details

### Bot Database Operations (Direct SQLite)

```javascript
// whatsapp-bot/src/handlers/message.js
const db = require('../database/db');

client.on('message', async (message) => {
  // Write directly to local SQLite
  await db.messages.insert({
    whatsappId: message.id._serialized,
    conversationId: message.from,
    fromWhatsappId: message.author || message.from,
    type: message.type,
    body: message.body,
    timestamp: message.timestamp,
    isFromMe: message.fromMe,
    ack: 1,
    hasMedia: message.hasMedia
  });
  
  // Handle media
  if (message.hasMedia) {
    const media = await message.downloadMedia();
    const r2Key = await uploadToR2(media);
    await db.messages.updateMedia(message.id._serialized, r2Key);
  }
});

// Update ACK status
client.on('message_ack', async (message, ack) => {
  await db.messages.updateAck(message.id._serialized, ack);
});
```

### MCP Server Queries SQLite via HTTP

```typescript
// server/tools/whatsapp/messages.ts
export const createGetMessagesToolool = (env: Env) =>
  createPrivateTool({
    id: "WHATSAPP_GET_MESSAGES",
    execute: async ({ context }) => {
      // Call bot HTTP API to query local SQLite
      const response = await fetch(`${env.BOT_API_URL}/query`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${env.BOT_API_KEY}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          sql: `SELECT * FROM messages WHERE conversationId = ? LIMIT ?`,
          params: [context.conversationId, context.limit]
        })
      });
      
      const messages = await response.json();
      return { messages };
    }
  });
```

### Rate Limiting Implementation

```javascript
// whatsapp-bot/src/utils/rateLimiter.js
class RateLimiter {
  constructor(maxPerMinute = 40) {
    this.maxPerMinute = maxPerMinute;
    this.queue = [];
    this.processing = false;
  }
  
  async wait() {
    return new Promise((resolve) => {
      this.queue.push(resolve);
      if (!this.processing) {
        this.processQueue();
      }
    });
  }
  
  async processQueue() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const resolve = this.queue.shift();
      resolve();
      
      // Wait 1500ms between messages (40/min = 1.5s interval)
      await new Promise(r => setTimeout(r, 1500));
    }
    
    this.processing = false;
  }
}

const rateLimiter = new RateLimiter(40);

// Usage in message planner
async function sendPlannedMessage(to, message) {
  await rateLimiter.wait();
  return await client.sendMessage(to, message);
}
```

## Environment Variables

### Bot (.env)
```bash
# WhatsApp
CHROME_PATH=/usr/bin/google-chrome
SESSION_PATH=./data/sessions

# Database
DB_PATH=./data/whatsapp.db

# R2 Storage
R2_ACCOUNT_ID=xxx
R2_ACCESS_KEY_ID=xxx
R2_SECRET_ACCESS_KEY=xxx
R2_BUCKET_NAME=whatsapp-media

# API
BOT_API_PORT=3001
BOT_API_KEY=shared_secret_with_mcp_server

# Rate Limiting
MAX_MESSAGES_PER_MINUTE=40
MESSAGE_DELAY_MS=1500
```

### MCP Server (wrangler.toml)
```toml
[vars]
BOT_API_URL = "http://localhost:3001"
BOT_API_KEY = "shared_secret_with_bot"
```

## Migration Path

### Week 1: Setup
1. Create `/whatsapp-bot/` directory structure
2. Install dependencies: `whatsapp-web.js`, `puppeteer`, `express`, `better-sqlite3`
3. Set up SQLite database with migrations
4. Configure LocalAuth session storage

### Week 2: Bot Development
1. Implement WhatsApp Web.js client initialization
2. Add all event handlers (message, ack, group, contact)
3. Implement rate limiter
4. Test QR authentication flow
5. Test message persistence to SQLite

### Week 3: API & Integration
1. Build Express HTTP API
2. Implement `/query` endpoint for MCP server
3. Implement `/send-message` endpoint for message planner
4. Set up R2 media storage
5. Test media upload/download

### Week 4: MCP Tools
1. Create MCP tools that call bot API
2. Test message retrieval
3. Test contact/group queries
4. Implement message planner tools
5. Test batch sending with rate limiting

### Week 5: Frontend
1. Build all 11 UI views
2. Implement TanStack Query hooks
3. Test real-time updates
4. Responsive design testing
5. Accessibility audit

### Week 6: Polish & Deploy
1. Performance optimization
2. Error handling improvements
3. Logging and monitoring
4. Documentation
5. User testing

## Testing Checklist

- [ ] QR code authentication works
- [ ] Bot reconnects automatically after disconnect
- [ ] All message types are persisted (text, image, video, etc.)
- [ ] ACK status updates correctly (sent → delivered → read)
- [ ] Media downloads and uploads to R2
- [ ] Rate limiter prevents >40 msgs/min
- [ ] Group events are captured (join, leave, update)
- [ ] Contact changes are synced
- [ ] SQLite database doesn't corrupt under load
- [ ] MCP server can query SQLite via HTTP
- [ ] Message planner generates contextual messages
- [ ] Batch sending works with proper delays
- [ ] UI is responsive on mobile/tablet/desktop
- [ ] All CRUD operations work correctly

## Security Considerations

1. **Session Security**: LocalAuth session files are encrypted by whatsapp-web.js
2. **API Authentication**: Shared secret between bot and MCP server
3. **Database Access**: SQLite file permissions (0600)
4. **Media Storage**: R2 signed URLs with expiration
5. **Rate Limiting**: Prevents abuse and bans
6. **Input Validation**: Zod schemas on all endpoints
7. **Error Handling**: Never expose sensitive data in errors

## Performance Optimizations

1. **SQLite Pragmas**: WAL mode, cache optimization
2. **Database Indexes**: On whatsappId, timestamp, conversationId
3. **Media Caching**: Store R2 URLs in database
4. **Query Batching**: Batch reads when possible
5. **Virtual Scrolling**: For long message lists in UI
6. **Debounced Search**: 300ms debounce on filters
7. **Lazy Loading**: Images load on scroll

## Known Limitations

1. **Single Instance**: Bot can only run on one machine at a time
2. **Browser Required**: Needs Chrome/Chromium installed
3. **Not Official API**: May break with WhatsApp updates
4. **Rate Limits**: 40 messages/minute hard limit
5. **File Size**: 64MB max for media files
6. **Local Only**: Database is not distributed

## Future Enhancements

1. **Multi-Account Support**: Run multiple WhatsApp accounts
2. **Cloud Backup**: Automatic backup to S3/R2
3. **Message Search**: Full-text search with FTS5
4. **AI Analysis**: Sentiment, topics, insights
5. **Export Tools**: PDF, CSV, JSON exports
6. **Webhook Notifications**: Real-time webhooks for events
7. **Docker Container**: Easy deployment via Docker
8. **Monitoring Dashboard**: Grafana/Prometheus integration

---

**Status**: All plans updated and ready for implementation! 🚀

See:
- `plans/mcp-whatsapp.md` - Main WhatsApp integration plan
- `plans/mcp-whatsapp-message-planner.md` - Message planner extension
- `plans/mcp-whatsapp-ui-layouts.md` - Complete UI wireframes (NEW!)
