# WhatsApp Web.js Integration - Complete Life OS Foundation

## Overview

This plan outlines a complete integration with WhatsApp Web.js to create a Life OS foundation. The system will persist all messages, media, contacts, groups, and conversations with full synchronization capabilities and future AI analysis support.

## Architecture

### Hybrid Architecture Model

Due to WhatsApp Web.js requiring Chrome/browser execution, we'll use a hybrid architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                    Local Machine                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │  WhatsApp Web.js Bot (Node.js + Puppeteer)        │    │
│  │  - QR Code Authentication (LocalAuth)             │    │
│  │  - Message Listening (40 msgs/min limit)          │    │
│  │  - Event Processing (message, message_ack, etc)   │    │
│  │  - Send/Receive Messages                           │    │
│  │  - Media Download (base64 MessageMedia)           │    │
│  └──────────────────┬─────────────────────────────────┘    │
│                     │                                        │
│  ┌─────────────────┴──────────┬────────────────────────┐   │
│  │                            │                         │   │
│  │  SQLite Database (Local)   │  R2 Storage (Media)    │   │
│  │  - Messages                │  - Images              │   │
│  │  - Contacts                │  - Videos              │   │
│  │  - Groups                  │  - Audio               │   │
│  │  - Conversations           │  - Documents           │   │
│  │  - Channel Rules           │  - Stickers            │   │
│  │  File: ./data/whatsapp.db  │  - Thumbnails          │   │
│  └────────────────────────────┴────────────────────────┘   │
│                     │ HTTPS API                             │
└─────────────────────┼─────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│           Deco MCP Server (Cloudflare Workers)              │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Tools & Workflows (RPC Interface)                 │    │
│  │  - Query local SQLite via HTTP                     │    │
│  │  - AI Analysis & Generation                        │    │
│  │  - Message Planning                                │    │
│  │  - Frontend Views                                  │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Key Architecture Decision: Local SQLite Database**

Instead of using Deco's Durable Objects SQLite, we use a **local SQLite file** (`./data/whatsapp.db`) stored in the repository. This decision is made because:

1. **Data Locality**: All WhatsApp data stays on the same machine as the bot
2. **Simplified Sync**: No need to sync data to remote database
3. **Performance**: Direct file access is faster than remote DB calls
4. **Backup**: Easy to backup - just copy the SQLite file
5. **Privacy**: Sensitive WhatsApp data never leaves the local machine
6. **Cost**: No database hosting costs

The MCP server will expose HTTP endpoints that the local bot can call to:
- Query the SQLite database
- Trigger AI analysis
- Manage message planning
- Serve the frontend UI

The bot will directly write to the local SQLite file and call MCP endpoints for advanced operations.

## Phase 1: Core Infrastructure

### 1.1 Database Schema

**Tables to Create:**

```typescript
// server/schema.ts additions

// WhatsApp Accounts (for managing bot connection)
export const whatsappAccountsTable = sqliteTable("whatsapp_accounts", {
  id: integer("id").primaryKey(),
  phoneNumber: text("phone_number").unique().notNull(),
  displayName: text("display_name"),
  status: text("status").notNull().default("disconnected"), // disconnected, connecting, connected, error
  lastSeen: integer("last_seen", { mode: "timestamp" }),
  sessionData: text("session_data"), // Encrypted session data
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Contacts
export const contactsTable = sqliteTable("contacts", {
  id: integer("id").primaryKey(),
  whatsappId: text("whatsapp_id").unique().notNull(), // e.g., "5511999999999@c.us"
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name"),
  pushName: text("push_name"), // Name set by user in WhatsApp
  number: text("number").notNull(),
  isMyContact: integer("is_my_contact").default(0), // Boolean: in contact list
  isBlocked: integer("is_blocked").default(0), // Boolean
  isGroup: integer("is_group").default(0), // Boolean
  profilePicUrl: text("profile_pic_url"),
  about: text("about"), // WhatsApp status/bio
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Groups
export const groupsTable = sqliteTable("groups", {
  id: integer("id").primaryKey(),
  whatsappId: text("whatsapp_id").unique().notNull(), // e.g., "123456789@g.us"
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name").notNull(),
  description: text("description"),
  pictureUrl: text("picture_url"),
  owner: text("owner"), // WhatsApp ID of group owner
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Group Members (many-to-many relationship)
export const groupMembersTable = sqliteTable("group_members", {
  id: integer("id").primaryKey(),
  groupId: integer("group_id").notNull().references(() => groupsTable.id),
  contactId: integer("contact_id").notNull().references(() => contactsTable.id),
  isAdmin: integer("is_admin").default(0), // Boolean
  joinedAt: integer("joined_at", { mode: "timestamp" }).notNull(),
});

// Conversations (chat threads)
export const conversationsTable = sqliteTable("conversations", {
  id: integer("id").primaryKey(),
  whatsappId: text("whatsapp_id").unique().notNull(), // Chat ID
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  type: text("type").notNull(), // "private", "group"
  contactId: integer("contact_id").references(() => contactsTable.id), // For private chats
  groupId: integer("group_id").references(() => groupsTable.id), // For group chats
  lastMessageId: integer("last_message_id"),
  lastMessageAt: integer("last_message_at", { mode: "timestamp" }),
  unreadCount: integer("unread_count").default(0),
  isArchived: integer("is_archived").default(0), // Boolean
  isPinned: integer("is_pinned").default(0), // Boolean
  isMuted: integer("is_muted").default(0), // Boolean
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Messages (core table)
export const messagesTable = sqliteTable("messages", {
  id: integer("id").primaryKey(),
  whatsappId: text("whatsapp_id").unique().notNull(), // Message ID from WhatsApp
  conversationId: integer("conversation_id").notNull().references(() => conversationsTable.id),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  fromContactId: integer("from_contact_id").notNull().references(() => contactsTable.id),
  
  // Message Content
  type: text("type").notNull(), // "text", "image", "video", "audio", "document", "sticker", "location", "vcard", "revoked"
  body: text("body"), // Text content
  caption: text("caption"), // Media caption
  
  // Media Reference (if applicable)
  hasMedia: integer("has_media").default(0), // Boolean
  mediaKey: text("media_key"), // R2 storage key for media file
  mediaMimeType: text("media_mime_type"),
  mediaSize: integer("media_size"), // Bytes
  mediaFilename: text("media_filename"),
  
  // Message Metadata
  timestamp: integer("timestamp", { mode: "timestamp" }).notNull(),
  isFromMe: integer("is_from_me").default(0), // Boolean
  isForwarded: integer("is_forwarded").default(0), // Boolean
  isStarred: integer("is_starred").default(0), // Boolean
  isBroadcast: integer("is_broadcast").default(0), // Boolean
  
  // Reply/Quote Context
  quotedMessageId: integer("quoted_message_id").references(() => messagesTable.id),
  
  // Delivery Status
  ack: integer("ack").default(0), // 0=error, 1=pending, 2=server, 3=delivery, 4=read, 5=played
  
  // System Messages
  isSystemMessage: integer("is_system_message").default(0), // Boolean
  systemMessageType: text("system_message_type"), // "group_create", "group_add", "group_remove", etc.
  
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Message Media (detailed media info)
export const messageMediaTable = sqliteTable("message_media", {
  id: integer("id").primaryKey(),
  messageId: integer("message_id").unique().notNull().references(() => messagesTable.id),
  r2Key: text("r2_key").notNull(), // Full R2 storage path
  mimeType: text("mime_type").notNull(),
  size: integer("size").notNull(), // Bytes
  filename: text("filename"),
  width: integer("width"), // For images/videos
  height: integer("height"), // For images/videos
  duration: integer("duration"), // For audio/video (seconds)
  thumbnailR2Key: text("thumbnail_r2_key"), // Thumbnail for videos/documents
  uploadedAt: integer("uploaded_at", { mode: "timestamp" }).notNull(),
});

// Channel Rules (allow/block configurations)
export const channelRulesTable = sqliteTable("channel_rules", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  channelType: text("channel_type").notNull(), // "contact", "group"
  channelId: text("channel_id").notNull(), // WhatsApp ID (contact or group)
  ruleType: text("rule_type").notNull(), // "allow", "block"
  priority: integer("priority").default(0), // Higher priority rules override lower
  applyToMessages: integer("apply_to_messages").default(1), // Boolean
  applyToMedia: integer("apply_to_media").default(1), // Boolean
  reason: text("reason"), // Optional reason for the rule
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Sync Status (track synchronization state)
export const syncStatusTable = sqliteTable("sync_status", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  entityType: text("entity_type").notNull(), // "contacts", "groups", "messages"
  entityId: text("entity_id"), // Specific entity ID being synced
  status: text("status").notNull(), // "pending", "in_progress", "completed", "error"
  lastSyncAt: integer("last_sync_at", { mode: "timestamp" }),
  errorMessage: text("error_message"),
  metadata: text("metadata"), // JSON string with additional sync info
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Indexes for performance
export const contactsWhatsappIdIndex = index("contacts_whatsapp_id_idx").on(contactsTable.whatsappId);
export const contactsAccountIdIndex = index("contacts_account_id_idx").on(contactsTable.accountId);
export const messagesConversationIdIndex = index("messages_conversation_id_idx").on(messagesTable.conversationId);
export const messagesTimestampIndex = index("messages_timestamp_idx").on(messagesTable.timestamp);
export const messagesFromContactIdIndex = index("messages_from_contact_id_idx").on(messagesTable.fromContactId);
export const conversationsAccountIdIndex = index("conversations_account_id_idx").on(conversationsTable.accountId);
export const channelRulesAccountIdIndex = index("channel_rules_account_id_idx").on(channelRulesTable.accountId);
```

### 1.2 R2 Storage Structure

```
/whatsapp-media/
  /{account_id}/
    /images/
      /{year}/{month}/{message_id}_{hash}.jpg
    /videos/
      /{year}/{month}/{message_id}_{hash}.mp4
    /audio/
      /{year}/{month}/{message_id}_{hash}.ogg
    /documents/
      /{year}/{month}/{message_id}_{hash}.pdf
    /stickers/
      /{year}/{month}/{message_id}_{hash}.webp
    /thumbnails/
      /{year}/{month}/{message_id}_{hash}_thumb.jpg
```

## Phase 2: WhatsApp Bot (Local Component)

### 2.1 Bot Architecture

Create a separate Node.js service that runs locally:

```
/whatsapp-bot/ (new directory, in the repository root)
  /src/
    /bot.js           - Main WhatsApp Web.js initialization
    /handlers/
      /message.js     - Message event handlers
      /contact.js     - Contact sync handlers
      /group.js       - Group sync handlers
      /auth.js        - QR code and authentication
    /database/
      /db.js          - SQLite connection and queries
      /migrations/    - Database migrations
    /storage/
      /media.js       - R2 media upload/download
    /api/
      /server.js      - Express server for MCP to call
      /client.js      - HTTP client to communicate with MCP server
    /config.js        - Bot configuration
  /data/
    whatsapp.db       - SQLite database file
    /sessions/        - WhatsApp Web.js session data (LocalAuth)
  package.json
  .env
  .gitignore          - Ignore sessions/ and data/
```

### 2.2 Bot Responsibilities

1. **WhatsApp Connection Management**
   - QR code generation for authentication (`client.on('qr')`)
   - Session persistence using LocalAuth strategy
   - Auto-reconnection on disconnect (`client.on('disconnected')`)
   - Health checks and state monitoring
   - Graceful shutdown handling

2. **Event Listening** (All WhatsApp Web.js Events)
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

3. **Direct SQLite Operations**
   - Write messages, contacts, groups to local SQLite
   - Query for duplicate detection
   - Update delivery status (ack)
   - No need to forward to MCP - data is local!

4. **Media Handling**
   - Download media using `message.downloadMedia()`
   - Returns MessageMedia object with base64 data
   - Upload to R2 via HTTP endpoint
   - Store R2 keys in SQLite
   - Handle 64MB file size limit
   - Generate thumbnails for videos

5. **Rate Limiting**
   - Enforce 40 messages/minute limit
   - Queue messages with delays
   - Prevent WhatsApp bans

## Phase 3: MCP Server Tools

### 3.1 Domain Organization

```
server/tools/
  /whatsapp/
    index.ts          - Aggregates all WhatsApp tools
    account.ts        - Account management tools
    contacts.ts       - Contact sync and management
    groups.ts         - Group sync and management
    messages.ts       - Message persistence and retrieval
    media.ts          - Media upload/download to R2
    rules.ts          - Allow/block channel rules
    sync.ts           - Synchronization orchestration
```

### 3.2 Tool Definitions

#### Account Management Tools (`account.ts`)

```typescript
// Tool: WHATSAPP_REGISTER_ACCOUNT
// Registers a new WhatsApp account in the system
inputSchema: {
  phoneNumber: string;
  displayName?: string;
}
outputSchema: {
  accountId: number;
  status: string;
}

// Tool: WHATSAPP_UPDATE_ACCOUNT_STATUS
// Updates connection status from bot
inputSchema: {
  phoneNumber: string;
  status: "disconnected" | "connecting" | "connected" | "error";
  sessionData?: string; // Encrypted session
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_GET_ACCOUNT_INFO
// Retrieves account information
inputSchema: {
  accountId?: number;
  phoneNumber?: string;
}
outputSchema: {
  account: AccountInfo;
}
```

#### Contact Sync Tools (`contacts.ts`)

```typescript
// Tool: WHATSAPP_SYNC_CONTACT
// Syncs a single contact (idempotent)
inputSchema: {
  accountId: number;
  whatsappId: string;
  name?: string;
  pushName?: string;
  number: string;
  isMyContact: boolean;
  isGroup: boolean;
  profilePicUrl?: string;
  about?: string;
}
outputSchema: {
  contactId: number;
  isNew: boolean; // true if created, false if updated
}

// Tool: WHATSAPP_SYNC_CONTACTS_BULK
// Syncs multiple contacts in a transaction
inputSchema: {
  accountId: number;
  contacts: ContactData[];
}
outputSchema: {
  synced: number;
  created: number;
  updated: number;
  errors: ErrorInfo[];
}

// Tool: WHATSAPP_GET_CONTACTS
// Retrieves contacts with filtering
inputSchema: {
  accountId: number;
  isGroup?: boolean;
  isMyContact?: boolean;
  search?: string;
  limit?: number;
  offset?: number;
}
outputSchema: {
  contacts: ContactInfo[];
  total: number;
}

// Tool: WHATSAPP_CHECK_CONTACT_DUPLICATE
// Checks if contact exists (for deduplication)
inputSchema: {
  accountId: number;
  whatsappId: string;
}
outputSchema: {
  exists: boolean;
  contactId?: number;
  lastUpdated?: string;
}
```

#### Group Sync Tools (`groups.ts`)

```typescript
// Tool: WHATSAPP_SYNC_GROUP
// Syncs group information (idempotent)
inputSchema: {
  accountId: number;
  whatsappId: string;
  name: string;
  description?: string;
  pictureUrl?: string;
  owner: string;
  members: Array<{
    whatsappId: string;
    isAdmin: boolean;
  }>;
}
outputSchema: {
  groupId: number;
  isNew: boolean;
  membersAdded: number;
  membersRemoved: number;
}

// Tool: WHATSAPP_GET_GROUPS
// Retrieves groups with filtering
inputSchema: {
  accountId: number;
  search?: string;
  limit?: number;
  offset?: number;
}
outputSchema: {
  groups: GroupInfo[];
  total: number;
}

// Tool: WHATSAPP_GET_GROUP_MEMBERS
// Gets members of a specific group
inputSchema: {
  groupId: number;
}
outputSchema: {
  members: GroupMemberInfo[];
}
```

#### Message Persistence Tools (`messages.ts`)

```typescript
// Tool: WHATSAPP_SAVE_MESSAGE
// Saves a single message (idempotent by whatsappId)
inputSchema: {
  accountId: number;
  whatsappId: string;
  conversationWhatsappId: string;
  fromWhatsappId: string;
  type: MessageType;
  body?: string;
  caption?: string;
  hasMedia: boolean;
  mediaData?: {
    mimeType: string;
    size: number;
    filename?: string;
    base64Data?: string; // For small media
  };
  timestamp: number;
  isFromMe: boolean;
  isForwarded: boolean;
  quotedMessageWhatsappId?: string;
  ack: number;
  isSystemMessage: boolean;
  systemMessageType?: string;
}
outputSchema: {
  messageId: number;
  isNew: boolean;
  mediaUploaded: boolean;
  mediaUrl?: string;
}

// Tool: WHATSAPP_SAVE_MESSAGES_BULK
// Saves multiple messages in batch
inputSchema: {
  accountId: number;
  messages: MessageData[];
}
outputSchema: {
  saved: number;
  created: number;
  updated: number;
  errors: ErrorInfo[];
}

// Tool: WHATSAPP_GET_MESSAGES
// Retrieves messages with filtering
inputSchema: {
  accountId: number;
  conversationId?: number;
  conversationWhatsappId?: string;
  fromDate?: string;
  toDate?: string;
  messageType?: string;
  hasMedia?: boolean;
  isFromMe?: boolean;
  search?: string; // Search in body/caption
  limit?: number;
  offset?: number;
}
outputSchema: {
  messages: MessageInfo[];
  total: number;
}

// Tool: WHATSAPP_UPDATE_MESSAGE_ACK
// Updates message delivery status
inputSchema: {
  whatsappId: string;
  ack: number;
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_CHECK_MESSAGE_DUPLICATE
// Checks if message already exists
inputSchema: {
  whatsappId: string;
}
outputSchema: {
  exists: boolean;
  messageId?: number;
}
```

#### Media Management Tools (`media.ts`)

```typescript
// Tool: WHATSAPP_UPLOAD_MEDIA
// Uploads media to R2 storage
inputSchema: {
  accountId: number;
  messageWhatsappId: string;
  mediaType: "image" | "video" | "audio" | "document" | "sticker";
  mimeType: string;
  size: number;
  filename?: string;
  base64Data: string; // or buffer
  width?: number;
  height?: number;
  duration?: number;
  generateThumbnail?: boolean;
}
outputSchema: {
  r2Key: string;
  url: string; // Signed URL for access
  thumbnailR2Key?: string;
  thumbnailUrl?: string;
}

// Tool: WHATSAPP_GET_MEDIA_URL
// Generates signed URL for media access
inputSchema: {
  r2Key: string;
  expiresIn?: number; // Seconds (default: 3600)
}
outputSchema: {
  url: string;
  expiresAt: string;
}

// Tool: WHATSAPP_DELETE_MEDIA
// Deletes media from R2 (for cleanup)
inputSchema: {
  r2Key: string;
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_GET_MEDIA_STATS
// Gets storage statistics
inputSchema: {
  accountId: number;
}
outputSchema: {
  totalSize: number;
  mediaCount: number;
  byType: Record<string, { count: number; size: number }>;
}
```

#### Channel Rules Tools (`rules.ts`)

```typescript
// Tool: WHATSAPP_ADD_CHANNEL_RULE
// Adds allow/block rule for a channel
inputSchema: {
  accountId: number;
  channelType: "contact" | "group";
  channelId: string; // WhatsApp ID
  ruleType: "allow" | "block";
  priority?: number;
  applyToMessages?: boolean;
  applyToMedia?: boolean;
  reason?: string;
}
outputSchema: {
  ruleId: number;
}

// Tool: WHATSAPP_REMOVE_CHANNEL_RULE
// Removes a channel rule
inputSchema: {
  ruleId: number;
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_GET_CHANNEL_RULES
// Gets rules for an account
inputSchema: {
  accountId: number;
  channelType?: "contact" | "group";
  ruleType?: "allow" | "block";
}
outputSchema: {
  rules: ChannelRuleInfo[];
}

// Tool: WHATSAPP_CHECK_CHANNEL_ALLOWED
// Checks if a channel is allowed (considers all rules)
inputSchema: {
  accountId: number;
  channelId: string;
  checkType: "messages" | "media";
}
outputSchema: {
  allowed: boolean;
  appliedRuleId?: number;
  reason?: string;
}

// Tool: WHATSAPP_UPDATE_CHANNEL_RULE
// Updates existing rule
inputSchema: {
  ruleId: number;
  ruleType?: "allow" | "block";
  priority?: number;
  applyToMessages?: boolean;
  applyToMedia?: boolean;
  reason?: string;
}
outputSchema: {
  success: boolean;
}
```

#### Synchronization Tools (`sync.ts`)

```typescript
// Tool: WHATSAPP_START_FULL_SYNC
// Initiates full synchronization process
inputSchema: {
  accountId: number;
  syncContacts?: boolean;
  syncGroups?: boolean;
  syncMessages?: boolean;
  fromDate?: string; // For message sync
}
outputSchema: {
  syncId: string;
  status: "started";
}

// Tool: WHATSAPP_GET_SYNC_STATUS
// Gets synchronization status
inputSchema: {
  accountId: number;
  entityType?: "contacts" | "groups" | "messages";
}
outputSchema: {
  syncs: SyncStatusInfo[];
}

// Tool: WHATSAPP_MARK_SYNC_COMPLETE
// Marks a sync operation as complete
inputSchema: {
  syncId: number;
  status: "completed" | "error";
  errorMessage?: string;
  metadata?: Record<string, any>;
}
outputSchema: {
  success: boolean;
}
```

## Phase 4: Workflows

### 4.1 Synchronization Workflows

```typescript
// Workflow: FULL_WHATSAPP_SYNC
// Orchestrates complete sync of all entities
Steps:
1. Start sync status tracking
2. Sync contacts (bulk)
3. Sync groups (bulk)
4. Sync conversations
5. Sync messages (paginated, respecting date range)
6. Update sync status

// Workflow: MESSAGE_PROCESSING_PIPELINE
// Processes incoming message with all checks
Steps:
1. Check message duplicate (skip if exists)
2. Check channel rules (block if not allowed)
3. Sync sender contact (if not exists)
4. Ensure conversation exists
5. Download and upload media (if applicable)
6. Save message to database
7. Update conversation last message
8. Trigger AI analysis hook (future phase)
```

## Phase 5: Bot Communication Protocol

### 5.1 Local SQLite Access Pattern

**The bot writes directly to SQLite - no HTTP calls for basic operations:**

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
  
  // Download and upload media if present
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

### 5.2 Bot HTTP API (for MCP to call)

The bot exposes HTTP endpoints for the MCP server to call:

```javascript
// whatsapp-bot/src/api/server.js
const express = require('express');
const app = express();

// Send message (for message planner)
app.post('/send-message', async (req, res) => {
  const { to, message } = req.body;
  const chatId = to.includes('@') ? to : `${to}@c.us`;
  
  try {
    await rateLimiter.wait(); // 40 msgs/min
    const sent = await client.sendMessage(chatId, message);
    res.json({ 
      success: true, 
      messageId: sent.id._serialized 
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Query SQLite (for MCP views)
app.post('/query', async (req, res) => {
  const { sql, params } = req.body;
  const results = await db.query(sql, params);
  res.json(results);
});

// Get bot status
app.get('/status', (req, res) => {
  res.json({
    connected: client.info !== null,
    phone: client.info?.wid?.user,
    platform: client.info?.platform
  });
});

app.listen(3001, () => {
  console.log('Bot API running on port 3001');
});
```

### 5.3 Authentication

- Bot API uses API key authentication
- Shared secret between bot and MCP server
- Store in environment variables (both sides)

## Phase 6: Idempotency & Deduplication

### 6.1 Idempotency Strategy

**For all sync operations:**

1. **Unique Identifiers**: Always use WhatsApp IDs as unique keys
2. **Upsert Pattern**: INSERT or UPDATE based on whatsappId
3. **Timestamp Comparison**: Only update if incoming data is newer
4. **Hash Comparison**: For media, use content hash to avoid re-upload

**Implementation Example:**

```typescript
// In WHATSAPP_SYNC_CONTACT tool
execute: async ({ context }) => {
  const db = await getDb(env);
  
  // Check if contact exists
  const existing = await db.select()
    .from(contactsTable)
    .where(
      and(
        eq(contactsTable.whatsappId, context.whatsappId),
        eq(contactsTable.accountId, context.accountId)
      )
    )
    .limit(1);
  
  const now = new Date();
  
  if (existing.length > 0) {
    // Update only if data is different or newer
    const contactData = {
      name: context.name ?? existing[0].name,
      pushName: context.pushName ?? existing[0].pushName,
      number: context.number,
      isMyContact: context.isMyContact ? 1 : 0,
      profilePicUrl: context.profilePicUrl ?? existing[0].profilePicUrl,
      about: context.about ?? existing[0].about,
      updatedAt: now,
    };
    
    await db.update(contactsTable)
      .set(contactData)
      .where(eq(contactsTable.id, existing[0].id));
    
    return { contactId: existing[0].id, isNew: false };
  } else {
    // Create new contact
    const result = await db.insert(contactsTable).values({
      whatsappId: context.whatsappId,
      accountId: context.accountId,
      name: context.name,
      pushName: context.pushName,
      number: context.number,
      isMyContact: context.isMyContact ? 1 : 0,
      isGroup: context.isGroup ? 1 : 0,
      profilePicUrl: context.profilePicUrl,
      about: context.about,
      createdAt: now,
      updatedAt: now,
    }).returning({ id: contactsTable.id });
    
    return { contactId: result[0].id, isNew: true };
  }
}
```

### 6.2 Deduplication Checks

**Before every insert:**

1. Query by whatsappId + accountId
2. If exists, decide: update, skip, or merge
3. Log duplicate detection for monitoring

**For media files:**

1. Generate hash of media content
2. Store hash in database
3. Check if hash exists before R2 upload
4. Reuse existing R2 key if duplicate

## Phase 7: Performance Optimizations

### 7.1 Batch Processing

- Process messages in batches of 100
- Use database transactions for bulk operations
- Parallel processing for independent operations

### 7.2 Caching Strategy

- Cache channel rules in memory (KV store)
- Cache contact lookups for frequently accessed contacts
- Implement LRU cache for media URLs

### 7.3 Indexing

- Index all foreign keys
- Index timestamp fields for date range queries
- Index whatsappId fields for uniqueness checks

## Phase 8: Future AI Analysis Hooks

### 8.1 Analysis Pipeline

```typescript
// Tool: WHATSAPP_ANALYZE_MESSAGE
// Triggers AI analysis on a message
inputSchema: {
  messageId: number;
  analysisTypes: Array<"sentiment" | "entities" | "intent" | "category">;
}
outputSchema: {
  analysisId: number;
  results: AnalysisResults;
}

// Tool: WHATSAPP_GET_CONVERSATION_INSIGHTS
// Gets AI insights for a conversation
inputSchema: {
  conversationId: number;
  fromDate?: string;
  toDate?: string;
}
outputSchema: {
  insights: ConversationInsights;
  summary: string;
  topics: string[];
  sentimentTrend: SentimentData[];
}
```

### 8.2 Analysis Database Tables

```typescript
// Future schema addition
export const messageAnalysisTable = sqliteTable("message_analysis", {
  id: integer("id").primaryKey(),
  messageId: integer("message_id").notNull().references(() => messagesTable.id),
  analysisType: text("analysis_type").notNull(),
  result: text("result").notNull(), // JSON string
  confidence: integer("confidence"), // 0-100
  analyzedAt: integer("analyzed_at", { mode: "timestamp" }).notNull(),
});
```

## Phase 9: Testing & Validation

### 9.1 Unit Tests

- Test each tool independently
- Mock database operations
- Test idempotency logic
- Test deduplication

### 9.2 Integration Tests

- Test bot → MCP server communication
- Test full sync workflow
- Test message persistence with media
- Test channel rules enforcement

### 9.3 Load Testing

- Simulate high message volume
- Test batch processing performance
- Monitor database query performance
- Test R2 storage limits

## Phase 10: Monitoring & Observability

### 10.1 Metrics to Track

- Messages processed per minute
- Sync success/failure rate
- Media upload success rate
- Average message processing time
- Database query performance
- Storage usage (DB + R2)

### 10.2 Logging

- Log all sync operations
- Log channel rule applications
- Log errors with context
- Log performance metrics

### 10.3 Alerts

- Alert on sync failures
- Alert on storage quota approaching limit
- Alert on bot disconnections
- Alert on high error rates

## Implementation Roadmap

### Week 1: Infrastructure
- [x] Create database schema
- [ ] Set up R2 bucket
- [ ] Create domain structure in server/tools/whatsapp/
- [ ] Implement basic account management tools

### Week 2: Bot Development
- [ ] Set up WhatsApp Web.js bot project
- [ ] Implement QR authentication
- [ ] Implement basic message listener
- [ ] Implement HTTP client to MCP server

### Week 3: Core Sync Tools
- [ ] Implement contact sync tools
- [ ] Implement group sync tools
- [ ] Implement message persistence tools
- [ ] Test idempotency logic

### Week 4: Media & Rules
- [ ] Implement media upload to R2
- [ ] Implement thumbnail generation
- [ ] Implement channel rules tools
- [ ] Test allow/block logic

### Week 5: Workflows & Testing
- [ ] Implement full sync workflow
- [ ] Implement message processing pipeline
- [ ] Write unit tests
- [ ] Write integration tests

### Week 6: Polish & Deploy
- [ ] Implement monitoring
- [ ] Add logging
- [ ] Performance optimization
- [ ] Deploy bot and MCP server

## Configuration Requirements

### MCP Server Environment Variables

```bash
# R2 Storage (Cloudflare)
R2_ACCOUNT_ID=your_account_id
R2_ACCESS_KEY_ID=your_access_key
R2_SECRET_ACCESS_KEY=your_secret_key
R2_BUCKET_NAME=whatsapp-media

# Bot Authentication
WHATSAPP_BOT_API_KEY=generated_api_key_for_bot

# Database (handled by Deco)
# No additional config needed
```

### Bot Environment Variables

```bash
# WhatsApp Bot
WHATSAPP_SESSION_PATH=./sessions
CHROME_PATH=/usr/bin/google-chrome

# MCP Server
MCP_SERVER_URL=https://your-app.deco.page
MCP_API_KEY=same_as_WHATSAPP_BOT_API_KEY

# Webhook Settings
WEBHOOK_RETRY_ATTEMPTS=3
WEBHOOK_RETRY_DELAY=1000
```

## Security Considerations

1. **API Key Security**: Never expose bot API key
2. **Session Security**: Encrypt WhatsApp session data
3. **Media Access**: Use signed URLs with expiration
4. **Rate Limiting**: Implement rate limits on all tools
5. **Input Validation**: Validate all incoming data with Zod
6. **SQL Injection**: Use parameterized queries (Drizzle handles this)
7. **XSS Prevention**: Sanitize message content before display

## Scalability Considerations

1. **Message Volume**: Design for 10k+ messages/day
2. **Media Storage**: Plan for TBs of media data
3. **Database Size**: Monitor and implement archival strategy
4. **Concurrent Connections**: Support multiple bot instances
5. **API Rate Limits**: Implement request queuing

## Success Criteria

- ✅ Bot successfully connects to WhatsApp
- ✅ All messages are persisted to database
- ✅ Media files are uploaded to R2
- ✅ No duplicate contacts, groups, or messages
- ✅ Channel rules work correctly (allow/block)
- ✅ Full sync completes without errors
- ✅ System handles 1000+ messages/hour
- ✅ Media retrieval is fast (<500ms)
- ✅ No data loss during bot restarts

## Future Enhancements

1. **Multi-Account Support**: Support multiple WhatsApp accounts
2. **AI-Powered Responses**: Auto-reply based on AI analysis
3. **Smart Categorization**: Auto-categorize conversations
4. **Search Engine**: Full-text search across all messages
5. **Analytics Dashboard**: View with conversation insights
6. **Export Tools**: Export conversations to PDF/CSV
7. **Backup System**: Automated backups to external storage
8. **Message Scheduling**: Schedule messages to be sent later

---

**This plan serves as the foundation for building a comprehensive Life OS with WhatsApp as the primary data source. The architecture is designed to be extensible, performant, and ready for future AI-powered features.**
