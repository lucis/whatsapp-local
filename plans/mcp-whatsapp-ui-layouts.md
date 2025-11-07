# WhatsApp Life OS - UI Layouts & Wireframes

## Overview

This document provides detailed wireframes and UI specifications for all views in the WhatsApp Life OS system. Each layout includes component breakdowns, user flows, and interaction patterns.

---

## 1. Dashboard / Home View

**Route**: `/` or `/dashboard`

**Purpose**: Central hub showing overview of WhatsApp activity and quick actions

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [☰ Menu]  WhatsApp Life OS                    [🔔 Notifications] [👤 User]│
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────┐  ┌─────────────────────────┐                 │
│  │  📱 Bot Status          │  │  📊 Today's Stats       │                 │
│  │                         │  │                         │                 │
│  │  🟢 Connected           │  │  Messages: 247          │                 │
│  │  Phone: +55 11 9xxxx   │  │  New Contacts: 3        │                 │
│  │  Last Seen: 2 min ago  │  │  Active Chats: 12       │                 │
│  │                         │  │  Storage: 2.4 GB        │                 │
│  │  [⚙️ Configure]         │  │                         │                 │
│  └─────────────────────────┘  └─────────────────────────┘                 │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  📈 Activity Timeline                                                 │ │
│  │                                                                       │ │
│  │  ┌──────────────────────────────────────────────────────────────┐   │ │
│  │  │                                                              │   │ │
│  │  │  ███░░░░█████░░██████░░░░███ (24h message volume chart)     │   │ │
│  │  │                                                              │   │ │
│  │  └──────────────────────────────────────────────────────────────┘   │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  Quick Actions:                                                            │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐                │
│  │ 📋 View        │ │ 👥 Manage      │ │ 📨 Message     │                │
│  │    Messages    │ │    Contacts    │ │    Planner     │                │
│  └────────────────┘ └────────────────┘ └────────────────┘                │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐                │
│  │ 📁 Groups      │ │ 🎯 Channel     │ │ 📊 Analytics   │                │
│  │                │ │    Rules       │ │    (Coming)    │                │
│  └────────────────┘ └────────────────┘ └────────────────┘                │
│                                                                             │
│  Recent Activity:                                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ 🔵 New message from João Silva • 2 min ago                           │ │
│  │ 🟢 Contact sync completed • 5 min ago • 234 contacts                 │ │
│  │ 🟡 Maria Santos joined "Project Alpha" group • 12 min ago            │ │
│  │ 🔵 New message in "Team Meeting" • 18 min ago                        │ │
│  │ 🟣 Message plan "Company Exit" sent • 1 hour ago • 23/23 successful │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Components:
- **Bot Status Card**: Real-time connection status
- **Stats Cards**: Key metrics for quick glance
- **Activity Chart**: 24h message volume
- **Quick Action Grid**: Navigation shortcuts
- **Activity Feed**: Recent events stream

---

## 2. Messages View

**Route**: `/messages`

**Purpose**: Browse, search, and view all WhatsApp messages

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Messages                                     [🔍 Search] [⚙️]   │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Filters: [All ▼] [📅 Date Range] [👤 Contact] [💬 Type] [Clear]          │
│                                                                             │
│  ┌────────────────────┬──────────────────────────────────────────────────┐ │
│  │  Conversations     │  Messages                                        │ │
│  │  (Left Sidebar)    │  (Main Content)                                  │ │
│  ├────────────────────┼──────────────────────────────────────────────────┤ │
│  │                    │                                                  │ │
│  │  🔍 [Search...]    │  Conversation with: João Silva                   │ │
│  │                    │  Phone: +55 11 99999-9999                        │ │
│  │  ━━━━━━━━━━━━━━━   │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │ │
│  │                    │                                                  │ │
│  │  [🟢] João Silva   │  ┌────────────────────────────────────────────┐ │ │
│  │  Hey, how are you? │  │ [Them] Oi, tudo bem?                      │ │ │
│  │  2 min ago         │  │        10:23 AM  ✓✓                        │ │ │
│  │                    │  └────────────────────────────────────────────┘ │ │
│  │  [📁] Team Alpha   │                                                  │ │
│  │  New budget        │  ┌────────────────────────────────────────────┐ │ │
│  │  15 min ago        │  │                                   [You] Oi! │ │ │
│  │                    │  │                     Tudo ótimo, e você?    │ │ │
│  │  [👥] Family       │  │                        10:25 AM  ✓✓        │ │ │
│  │  Mom: Dinner at 8? │  └────────────────────────────────────────────┘ │ │
│  │  1 hour ago        │                                                  │ │
│  │                    │  ┌────────────────────────────────────────────┐ │ │
│  │  [🟢] Maria Santos │  │ [Them] 📸 [Image]                         │ │ │
│  │  Thanks!           │  │        Look at this!                       │ │ │
│  │  2 hours ago       │  │        10:28 AM  ✓✓                        │ │ │
│  │                    │  │        [📥 Download] [👁️ View]             │ │ │
│  │  [Show More...]    │  └────────────────────────────────────────────┘ │ │
│  │                    │                                                  │ │
│  │                    │  ┌────────────────────────────────────────────┐ │ │
│  │                    │  │                              [You] 😊 Nice! │ │ │
│  │                    │  │                        10:29 AM  ✓✓        │ │ │
│  │                    │  └────────────────────────────────────────────┘ │ │
│  │                    │                                                  │ │
│  │                    │  [Load Older Messages]                          │ │
│  │                    │                                                  │ │
│  └────────────────────┴──────────────────────────────────────────────────┘ │
│                                                                             │
│  💡 Tip: Click on any message to see details and metadata                  │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Features:
- **Conversation List**: Left sidebar with recent chats
- **Message Thread**: WhatsApp-style message bubbles
- **Media Preview**: Images, documents, videos
- **Search & Filters**: Date range, contact, message type
- **Delivery Status**: Check marks (✓✓ = delivered, ✓✓✓ = read)
- **Pagination**: Load older messages on scroll

### Message Detail Modal (on click):
```
┌──────────────────────────────────────────────────┐
│  Message Details                         [✕]    │
├──────────────────────────────────────────────────┤
│                                                  │
│  From: João Silva (+55 11 99999-9999)           │
│  To: Me                                          │
│  Date: 2025-11-06 10:23:45                      │
│  Message ID: 3A1234567890ABCDEF                 │
│  Type: Text                                      │
│  Status: Read (✓✓✓)                             │
│                                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                  │
│  Content:                                        │
│  "Oi, tudo bem?"                                 │
│                                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                  │
│  Timestamps:                                     │
│  • Sent: 10:23:12                               │
│  • Delivered: 10:23:15                          │
│  • Read: 10:24:03                               │
│                                                  │
│  [📋 Copy Text] [🔗 Copy ID] [📤 Export]        │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 3. Contacts View

**Route**: `/contacts`

**Purpose**: Manage WhatsApp contacts with full CRUD operations

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Contacts                   [🔍 Search] [+ Add Manual] [⚙️]      │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Filters: [All ▼] [My Contacts] [Groups] [Blocked] [Recent]               │
│                                                                             │
│  Showing 234 contacts                                [📊 Grid] [📋 List]   │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │ [👤]  João Silva                                      [🟢 Online] │   │
│  │       +55 11 99999-9999                                            │   │
│  │       About: "Living my best life 🌟"                             │   │
│  │       Tags: Work, Project Alpha                                    │   │
│  │       Last seen: Active now                                        │   │
│  │       [💬 Message] [👁️ View] [✏️ Edit] [•••]                      │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │ [👤]  Maria Santos                                    [⚪ Offline] │   │
│  │       +55 11 98888-8888                                            │   │
│  │       About: "Designer @ Company XYZ"                              │   │
│  │       Tags: Work, Clients                                          │   │
│  │       Last seen: Today at 14:32                                    │   │
│  │       [💬 Message] [👁️ View] [✏️ Edit] [•••]                      │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │ [👤]  Pedro Costa                                     [⚪ Offline] │   │
│  │       +55 11 97777-7777                                            │   │
│  │       About: Not set                                               │   │
│  │       Tags: Family                                                 │   │
│  │       Last seen: Yesterday at 20:15                                │   │
│  │       [💬 Message] [👁️ View] [✏️ Edit] [•••]                      │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  [Load More...]                                            [1] 2 3 ... 10  │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Contact Detail View (Slide-in Panel):
```
┌─────────────────────────────────────────────┐
│  Contact Details                    [✕]    │
├─────────────────────────────────────────────┤
│                                             │
│       ┌───────────┐                         │
│       │   [👤]    │  João Silva             │
│       │  Photo    │  +55 11 99999-9999      │
│       └───────────┘                         │
│                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                             │
│  📱 WhatsApp Info:                          │
│  • WhatsApp ID: 5511999999999@c.us          │
│  • Push Name: João                          │
│  • In My Contacts: ✅ Yes                   │
│  • Blocked: ❌ No                           │
│                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                             │
│  📝 About:                                  │
│  "Living my best life 🌟"                  │
│                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                             │
│  🏷️ Tags:                                   │
│  [Work] [Project Alpha]  [+ Add Tag]       │
│                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                             │
│  📊 Statistics:                             │
│  • Total Messages: 1,234                    │
│  • Last Message: 2 min ago                  │
│  • Groups in Common: 5                      │
│  • First Contact: 2023-05-12                │
│                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                             │
│  Actions:                                   │
│  [💬 Send Message]                          │
│  [📋 View Messages]                         │
│  [👥 View Groups]                           │
│  [🚫 Block Contact]                         │
│  [🗑️ Delete Contact]                        │
│                                             │
└─────────────────────────────────────────────┘
```

### Add/Edit Contact Modal:
```
┌─────────────────────────────────────────────┐
│  Edit Contact                       [✕]    │
├─────────────────────────────────────────────┤
│                                             │
│  Name:                                      │
│  [João Silva                          ]    │
│                                             │
│  Phone Number:                              │
│  [+55 11 99999-9999                   ]    │
│  (Format: +[country][area][number])         │
│                                             │
│  Tags:                                      │
│  [Work] [Project Alpha] [+ Add]            │
│                                             │
│  Custom Notes:                              │
│  ┌───────────────────────────────────────┐ │
│  │ Met at conference 2023.               │ │
│  │ Interested in our product.            │ │
│  │                                       │ │
│  └───────────────────────────────────────┘ │
│                                             │
│  Status:                                    │
│  [✓] Active                                 │
│  [ ] Blocked                                │
│                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                             │
│  [Cancel]              [💾 Save Changes]   │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 4. Groups View

**Route**: `/groups`

**Purpose**: Browse and manage WhatsApp groups

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Groups                        [🔍 Search] [+ Create] [⚙️]       │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Filters: [All ▼] [Active] [Muted] [My Groups] [Tags ▼]                   │
│                                                                             │
│  Showing 47 groups                                [📊 Grid] [📋 List]      │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ [📁] Team Alpha Project                                              │ │
│  │      Description: Main project coordination group                    │ │
│  │      Members: 12 • Messages: 3,456 • Tags: Work, Projects           │ │
│  │      Last Activity: 5 min ago                                        │ │
│  │      [💬 View] [👥 Members] [✏️ Edit] [🏷️ Tags] [•••]               │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ [📁] Family                                                   [🔇]   │ │
│  │      Description: Family group chat                                  │ │
│  │      Members: 8 • Messages: 12,345 • Tags: Family, Personal         │ │
│  │      Last Activity: 1 hour ago                                       │ │
│  │      [💬 View] [👥 Members] [✏️ Edit] [🏷️ Tags] [•••]               │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ [📁] Client - Acme Corp                                              │ │
│  │      Description: Communication with Acme Corp team                  │ │
│  │      Members: 5 • Messages: 234 • Tags: Clients, Business           │ │
│  │      Last Activity: Yesterday at 18:30                               │ │
│  │      [💬 View] [👥 Members] [✏️ Edit] [🏷️ Tags] [•••]               │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  [Load More...]                                            [1] 2 3 ... 8   │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Group Detail View:
```
┌─────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Team Alpha Project                              [✏️] [•••]   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────┐  Team Alpha Project                                     │
│  │   [📁]     │  12 members • Created: 2024-03-15                       │
│  │ Group Pic  │  Admin: João Silva                                      │
│  └────────────┘  Tags: [Work] [Projects] [Active]                      │
│                                                                          │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                                          │
│  📝 Description:                                                         │
│  Main project coordination group for Team Alpha initiative.             │
│  All project-related discussions should happen here.                    │
│                                                                          │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                                          │
│  👥 Members (12):                                   [+ Invite] [⚙️]     │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐    │
│  │ [👤] João Silva (Admin)                         [🟢 Online]    │    │
│  │ [👤] Maria Santos (Admin)                       [⚪ Offline]   │    │
│  │ [👤] Pedro Costa                                [🟢 Online]    │    │
│  │ [👤] Ana Oliveira                               [⚪ Offline]   │    │
│  │ [👤] Carlos Mendes                              [🟢 Online]    │    │
│  │                                                                │    │
│  │ [Show All Members...]                                          │    │
│  └────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                                          │
│  📊 Statistics:                                                          │
│  • Total Messages: 3,456                                                │
│  • Messages Today: 47                                                   │
│  • Most Active: João Silva (234 msgs)                                  │
│  • Last Activity: 5 min ago                                             │
│  • Average Response Time: 12 minutes                                    │
│                                                                          │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                                          │
│  Quick Actions:                                                          │
│  [💬 View Messages] [📤 Send Message] [📥 Export Chat]                 │
│  [🔇 Mute/Unmute] [📌 Pin/Unpin] [🗑️ Leave Group]                     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Channel Rules View

**Route**: `/rules`

**Purpose**: Configure allow/block rules for channels (contacts and groups)

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Channel Rules                      [+ Add Rule] [📥 Import]    │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Channel rules control which contacts/groups are tracked and stored.       │
│  Rules are applied in priority order (higher number = higher priority).    │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Active Rules (12):                            [Filter: All ▼] [Sort ▼]   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ ⚪ BLOCK | Priority: 100                                  [✏️] [🗑️] │ │
│  │                                                                      │ │
│  │ Type: Group                                                          │ │
│  │ Target: "Spam Group ABC"                                             │ │
│  │ WhatsApp ID: 123456789@g.us                                          │ │
│  │ Applies to: [✓] Messages [✓] Media                                  │ │
│  │ Reason: Spam/unwanted content                                        │ │
│  │ Created: 2025-10-15 • Last Modified: 2025-10-15                     │ │
│  │                                                                      │ │
│  │ [Toggle Off/On]                                                      │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ 🟢 ALLOW | Priority: 90                                   [✏️] [🗑️] │ │
│  │                                                                      │ │
│  │ Type: Contact                                                        │ │
│  │ Target: João Silva (+55 11 99999-9999)                              │ │
│  │ WhatsApp ID: 5511999999999@c.us                                      │ │
│  │ Applies to: [✓] Messages [✓] Media                                  │ │
│  │ Reason: Important contact - always track                            │ │
│  │ Created: 2025-09-20 • Last Modified: 2025-09-20                     │ │
│  │                                                                      │ │
│  │ [Toggle Off/On]                                                      │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ 🟢 ALLOW | Priority: 80                                   [✏️] [🗑️] │ │
│  │                                                                      │ │
│  │ Type: Group                                                          │ │
│  │ Target: "Team Alpha Project"                                         │ │
│  │ WhatsApp ID: 987654321@g.us                                          │ │
│  │ Applies to: [✓] Messages [ ] Media (media blocked)                  │ │
│  │ Reason: Work group - important but limit media                      │ │
│  │ Created: 2025-08-10 • Last Modified: 2025-10-01                     │ │
│  │                                                                      │ │
│  │ [Toggle Off/On]                                                      │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  [Load More...]                                                [1] 2       │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Add/Edit Rule Modal:
```
┌─────────────────────────────────────────────────────┐
│  Create Channel Rule                        [✕]    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Rule Type:                                         │
│  ( ) Allow   (•) Block                              │
│                                                     │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                     │
│  Channel Type:                                      │
│  (•) Contact   ( ) Group                            │
│                                                     │
│  Select Channel:                                    │
│  [🔍 Search contacts...          ] [Browse]        │
│                                                     │
│  Selected: João Silva (+55 11 99999-9999)          │
│                                                     │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                     │
│  Priority:                                          │
│  [90                ] (0-100, higher = first)      │
│                                                     │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                     │
│  Apply Rule To:                                     │
│  [✓] Messages                                       │
│  [✓] Media                                          │
│                                                     │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                     │
│  Reason (optional):                                 │
│  ┌─────────────────────────────────────────────┐   │
│  │ Important contact - always track all        │   │
│  │ communication                                │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                     │
│  [Cancel]                    [✅ Create Rule]      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 6. Message Planner - Dashboard

**Route**: `/planner`

**Purpose**: Main dashboard for message planning campaigns

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Message Planner                          [+ New Plan] [⚙️]      │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Plan and send personalized messages to multiple groups at once            │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Active Plans (2):                                                          │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ 📨 Company Exit - Farewell Messages                  [Status: Draft] │ │
│  │                                                                      │ │
│  │ Recipients: 23 groups                                                │ │
│  │ Progress: ██████░░░░ 12/23 reviewed (52%)                           │ │
│  │ Created: Today at 09:15 • Last edited: 10 minutes ago               │ │
│  │                                                                      │ │
│  │ [✏️ Continue Editing] [👁️ Preview] [•••]                            │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │ 📨 Event Invitation - Company Party               [Status: Sending] │ │
│  │                                                                      │ │
│  │ Recipients: 15 groups                                                │ │
│  │ Progress: ████████░░ 8/15 sent (53%)                                │ │
│  │ Started: Today at 11:00 • Est. completion: 11:12                    │ │
│  │                                                                      │ │
│  │ [⏸️ Pause] [👁️ View Progress] [•••]                                 │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Completed Plans (8):                                    [View All →]      │
│                                                                             │
│  ┌────────────────────┬────────────────────┬──────────────┬─────────────┐ │
│  │ Plan Name          │ Recipients         │ Success Rate │ Completed   │ │
│  ├────────────────────┼────────────────────┼──────────────┼─────────────┤ │
│  │ Holiday Greetings  │ 45 groups          │ 100% (45/45) │ 2 days ago  │ │
│  │ Project Update     │ 12 contacts        │ 100% (12/12) │ 1 week ago  │ │
│  │ Meeting Reminder   │ 8 groups           │ 87% (7/8)    │ 2 weeks ago │ │
│  └────────────────────┴────────────────────┴──────────────┴─────────────┘ │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Quick Stats:                                                               │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐           │
│  │ Total Plans  │ Messages     │ Success Rate │ Avg. Time    │           │
│  │     12       │    234       │    96.2%     │  8.5 min     │           │
│  └──────────────┴──────────────┴──────────────┴──────────────┘           │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Message Planner - Group Selection Wizard

**Route**: `/planner/new/select-groups`

**Purpose**: Step 1 - Filter and select target groups

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  New Message Plan: Select Groups                  Step 1 of 5    │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │ 1⃣ Select   2⃣ Template   3⃣ Generate   4⃣ Review   5⃣ Send         │  │
│  │ Groups                                                              │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────┬────────────────────────────────────────────────────────┐ │
│  │ Filters      │  Groups (24 found)                   [Grid] [List]    │ │
│  │              │                                                        │ │
│  │ [By Members] │  Selected: 8 groups                                   │ │
│  │  By Tags     │                                                        │ │
│  │  Advanced    │  [Select All] [Clear Selection]                       │ │
│  │              │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │ ━━━━━━━━━━━ │                                                        │ │
│  │              │  ┌──────────────────────────────────────────────┐    │ │
│  │ Filter by    │  │ ☑️ Team Alpha Project                        │    │ │
│  │ Members:     │  │    Members: 12 • Tags: Work, Projects        │    │ │
│  │              │  │    Includes: João, Maria, Pedro              │    │ │
│  │ [+] Add      │  │    Last Active: 5 min ago                    │    │ │
│  │ Person       │  └──────────────────────────────────────────────┘    │ │
│  │              │                                                        │ │
│  │ ┌──────────┐ │  ┌──────────────────────────────────────────────┐    │ │
│  │ │ [👤]     │ │  │ ☑️ Project Beta Team                         │    │ │
│  │ │ João     │ │  │    Members: 8 • Tags: Work, Projects         │    │ │
│  │ │ Silva [X]│ │  │    Includes: João, Ana, Carlos               │    │ │
│  │ └──────────┘ │  │    Last Active: 2 hours ago                  │    │ │
│  │              │  └──────────────────────────────────────────────┘    │ │
│  │ ┌──────────┐ │                                                        │ │
│  │ │ [👤]     │ │  ┌──────────────────────────────────────────────┐    │ │
│  │ │ Maria    │ │  │ ☐ Family Group                               │    │ │
│  │ │ Santos[X]│ │  │    Members: 8 • Tags: Family, Personal       │    │ │
│  │ └──────────┘ │  │    Includes: Maria                           │    │ │
│  │              │  │    Last Active: Yesterday                    │    │ │
│  │ Match Mode:  │  └──────────────────────────────────────────────┘    │ │
│  │ (•) Any      │                                                        │ │
│  │ ( ) All      │  ┌──────────────────────────────────────────────┐    │ │
│  │              │  │ ☑️ Client - Acme Corp                        │    │ │
│  │ ━━━━━━━━━━━ │  │    Members: 5 • Tags: Clients, Business      │    │ │
│  │              │  │    Includes: João                            │    │ │
│  │ Exclude:     │  │    Last Active: Yesterday                    │    │ │
│  │              │  └──────────────────────────────────────────────┘    │ │
│  │ ┌──────────┐ │                                                        │ │
│  │ │ [📁]     │ │  [Show More Groups...]                                │ │
│  │ │ Spam  [X]│ │                                                        │ │
│  │ └──────────┘ │                                                        │ │
│  │              │                                                        │ │
│  │ [💾 Save    │                                                        │ │
│  │  Filter]    │                                                        │ │
│  │              │                                                        │ │
│  └──────────────┴────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ [Cancel]                8 groups selected      [Next: Choose Template →]│ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Message Planner - Review & Edit

**Route**: `/planner/:planId/review`

**Purpose**: Step 4 - Review and edit AI-generated messages before sending

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Company Exit Farewell: Review Messages          Step 4 of 5     │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Review each message before sending. You can edit, regenerate, or skip.    │
│                                                                             │
│  ┌───────────┬──────────────────────────────────────────────────────────┐ │
│  │Recipients │  Message Editor                          Group 12 of 23  │ │
│  │ (23)      │                                                          │ │
│  │           │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │ [🔍]      │                                                          │ │
│  │           │  Group: Team Alpha Project                              │ │
│  │ ✅ Group 1│  Members: 12 • Tags: [Work] [Projects]                  │ │
│  │ ✅ Group 2│                                                          │ │
│  │ ✅ Group 3│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │ ...       │                                                          │ │
│  │ ✅ Grp 11 │  🤖 AI Generated Message:                               │ │
│  │ ➡️ Grp 12 │  ┌────────────────────────────────────────────────────┐ │ │
│  │ ⏸️ Grp 13 │  │ Olá, equipe do Team Alpha!                        │ │ │
│  │ ⏸️ Grp 14 │  │                                                    │ │ │
│  │ ...       │  │ Queria compartilhar que estou saindo da empresa   │ │ │
│  │           │  │ na próxima semana. Foi um prazer imenso trabalhar │ │ │
│  │ Progress: │  │ com vocês no projeto Alpha - conseguimos entregar │ │ │
│  │ ████░░░░░ │  │ resultados incríveis juntos!                      │ │ │
│  │ 11/23     │  │                                                    │ │ │
│  │ (48%)     │  │ Agradeço toda a colaboração e parceria. Estarei   │ │ │
│  │           │  │ disponível no LinkedIn se quiserem manter contato.│ │ │
│  │           │  │                                                    │ │ │
│  │           │  │ Desejo muito sucesso para o projeto e para todos! │ │ │
│  │           │  │                                                    │ │ │
│  │           │  │ Um abraço,                                        │ │ │
│  │           │  │ [Seu nome]                                        │ │ │
│  │           │  └────────────────────────────────────────────────────┘ │ │
│  │           │                                                          │ │
│  │           │  [🔄 Regenerate] [📋 Copy]                              │ │
│  │           │                                                          │ │
│  │           │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │           │                                                          │ │
│  │           │  ✏️ Your Final Message:                                 │ │
│  │           │  ┌────────────────────────────────────────────────────┐ │ │
│  │           │  │ Olá, equipe do Team Alpha!                        │ │ │
│  │           │  │                                                    │ │ │
│  │           │  │ Queria compartilhar que estou saindo da empresa   │ │ │
│  │           │  │ na próxima semana. Foi um prazer imenso trabalhar │ │ │
│  │           │  │ com vocês no projeto Alpha - conseguimos entregar │ │ │
│  │           │  │ resultados incríveis juntos!                      │ │ │
│  │           │  │                                                    │ │ │
│  │           │  │ Agradeço toda a colaboração e parceria. Estarei   │ │ │
│  │           │  │ disponível no LinkedIn se quiserem manter contato.│ │ │
│  │           │  │                                                    │ │ │
│  │           │  │ Desejo muito sucesso para o projeto e para todos! │ │ │
│  │           │  │                                                    │ │ │
│  │           │  │ Um abraço,                                        │ │ │
│  │           │  │ [Seu nome]                                        │ │ │
│  │           │  │                                                    │ │ │
│  │           │  └────────────────────────────────────────────────────┘ │ │
│  │           │                                                          │ │
│  │           │  Characters: 358 • Words: 67                            │ │
│  │           │                                                          │ │
│  │           │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │           │                                                          │ │
│  │           │  💡 Context Used:                    [View Details ▼]   │ │
│  │           │  • Group name, members, project tag                     │ │
│  │           │  • Rule applied: "Professional tone for work groups"    │ │
│  │           │                                                          │ │
│  └───────────┴──────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ [← Previous]  [Skip]  [✅ Approve & Next →]      [Bulk Actions ▼]     │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Message Planner - Sending Progress

**Route**: `/planner/:planId/sending`

**Purpose**: Step 5 - Monitor real-time sending progress

```
┌────────────────────────────────────────────────────────────────────────────┐
│  Sending Messages...                                    [⏸️ Pause] [✕ Stop]│
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│                      ┌──────────────────┐                                  │
│                      │                  │                                  │
│                      │                  │                                  │
│                      │       17         │                                  │
│                      │    ────────      │   Large circular progress        │
│                      │       23         │   indicator showing 17/23        │
│                      │                  │                                  │
│                      │   (74% sent)     │                                  │
│                      │                  │                                  │
│                      └──────────────────┘                                  │
│                                                                             │
│                                                                             │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐           │
│  │   Sent       │   Failed     │  Remaining   │   Est. Time  │           │
│  │     17       │      0       │      6       │   0:42 min   │           │
│  └──────────────┴──────────────┴──────────────┴──────────────┘           │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  📤 Currently sending to:                                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Team Marketing Group                                                 │ │
│  │  Members: 8 • Sending...  [⏳]                                        │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Recent Activity:                                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ ✅ Team Alpha Project • Sent successfully • 10:35:42                  │ │
│  │ ✅ Client - Acme Corp • Sent successfully • 10:35:40                  │ │
│  │ ✅ Project Beta Team • Sent successfully • 10:35:38                   │ │
│  │ ✅ Design Team • Sent successfully • 10:35:36                         │ │
│  │ ✅ Product Team • Sent successfully • 10:35:34                        │ │
│  │ ✅ Engineering Team • Sent successfully • 10:35:32                    │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  ⚙️ Settings:                                                              │
│  • Delay between messages: 2 seconds                                       │
│  • Max concurrent sends: 1                                                 │
│  • Retry failed: Enabled (3 attempts)                                      │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘

[After completion:]

┌────────────────────────────────────────────────────────────────────────────┐
│  All Messages Sent! 🎉                                                     │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                      ┌──────────────────┐                                  │
│                      │                  │                                  │
│                      │       ✓          │                                  │
│                      │                  │                                  │
│                      │   23 messages    │                                  │
│                      │   successfully   │                                  │
│                      │    delivered     │                                  │
│                      │                  │                                  │
│                      └──────────────────┘                                  │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Summary:                                                                   │
│  • Total Recipients: 23 groups                                             │
│  • Successfully Sent: 23 (100%)                                            │
│  • Failed: 0 (0%)                                                          │
│  • Total Time: 0:48 minutes                                                │
│  • Started: 10:35:00                                                       │
│  • Completed: 10:35:48                                                     │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  What's next?                                                               │
│  ┌──────────────────────┐ ┌──────────────────────┐                        │
│  │ 📊 View Full Report  │ │ 📋 Back to Dashboard │                        │
│  └──────────────────────┘ └──────────────────────┘                        │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Bot Configuration View

**Route**: `/config`

**Purpose**: Configure bot settings and connection

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Bot Configuration                                [💾 Save]      │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────┬───────────────────────────────────────────────────┐ │
│  │ Sections          │  Connection Settings                              │ │
│  │                   │                                                   │ │
│  │ [🔌] Connection   │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │  📊  Statistics   │                                                   │ │
│  │  ⚙️  Preferences  │  Bot Status:                                      │ │
│  │  🗄️  Database     │  🟢 Connected                                     │ │
│  │  🔐  Security     │                                                   │ │
│  │  📱  Account      │  Connected Account:                               │ │
│  │                   │  Phone: +55 11 99999-9999                         │ │
│  │                   │  Name: Your Name                                  │ │
│  │                   │  Platform: Chrome/Desktop                         │ │
│  │                   │  Connected Since: 2025-11-06 08:00:00            │ │
│  │                   │                                                   │ │
│  │                   │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │                   │                                                   │ │
│  │                   │  QR Code Authentication:                          │ │
│  │                   │                                                   │ │
│  │                   │  [🔄 Reconnect]  [🗑️ Clear Session]              │ │
│  │                   │                                                   │ │
│  │                   │  ⚠️ Warning: Clearing session will require        │ │
│  │                   │  re-scanning QR code on your phone.               │ │
│  │                   │                                                   │ │
│  │                   │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │                   │                                                   │ │
│  │                   │  Bot API Settings:                                │ │
│  │                   │                                                   │ │
│  │                   │  Local API URL:                                   │ │
│  │                   │  [http://localhost:3001              ]           │ │
│  │                   │                                                   │ │
│  │                   │  API Key:                                         │ │
│  │                   │  [•••••••••••••••••••••••••••••      ] [Show]    │ │
│  │                   │                                                   │ │
│  │                   │  Connection Test:                                 │ │
│  │                   │  [🧪 Test Connection]                             │ │
│  │                   │                                                   │ │
│  │                   │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │ │
│  │                   │                                                   │ │
│  │                   │  Rate Limiting:                                   │ │
│  │                   │                                                   │ │
│  │                   │  Max Messages per Minute:                         │ │
│  │                   │  [40              ] (WhatsApp limit: 40)         │ │
│  │                   │                                                   │ │
│  │                   │  Delay Between Messages (ms):                     │ │
│  │                   │  [2000            ] (2 seconds)                   │ │
│  │                   │                                                   │ │
│  │                   │  [✓] Enable automatic rate limiting               │ │
│  │                   │                                                   │ │
│  └───────────────────┴───────────────────────────────────────────────────┘ │
│                                                                             │
│  [Cancel]                                                [💾 Save Changes] │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Database Settings Tab:
```
┌────────────────────────────────────────────────────────────────────────────┐
│  Database Settings                                                          │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SQLite Database:                                                           │
│                                                                             │
│  Database File: ./data/whatsapp.db                                         │
│  Size: 245.7 MB                                                            │
│  Last Modified: 2 minutes ago                                              │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Statistics:                                                                │
│  • Total Messages: 12,345                                                  │
│  • Total Contacts: 234                                                     │
│  • Total Groups: 47                                                        │
│  • Total Conversations: 89                                                 │
│  • Media Files: 1,234 (2.4 GB in R2)                                       │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Maintenance:                                                               │
│                                                                             │
│  [🧹 Vacuum Database] - Optimize and reclaim space                         │
│  [📊 Analyze Database] - Update statistics for better performance          │
│  [💾 Backup Database] - Create backup file                                 │
│  [📥 Export Data] - Export to CSV/JSON                                     │
│                                                                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                             │
│  Automatic Cleanup:                                                         │
│                                                                             │
│  [✓] Auto-delete messages older than [90 ] days                           │
│  [✓] Auto-delete media older than [180] days                              │
│  [ ] Keep starred messages forever                                         │
│                                                                             │
│  Last Cleanup: 1 week ago                                                  │
│  [▶️ Run Cleanup Now]                                                      │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. Analytics View (Future)

**Route**: `/analytics`

**Purpose**: AI-powered insights and analytics

```
┌────────────────────────────────────────────────────────────────────────────┐
│  [← Back]  Analytics & Insights                          📅 Last 30 Days   │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🚧 Coming Soon: AI-Powered WhatsApp Analytics                             │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Planned Features:                                                    │ │
│  │                                                                       │ │
│  │  📊 Message Volume Trends                                            │ │
│  │  👥 Top Contacts & Groups                                            │ │
│  │  🕐 Activity Heatmaps                                                │ │
│  │  💬 Conversation Analysis                                            │ │
│  │  🎯 Sentiment Tracking                                               │ │
│  │  📈 Response Time Analytics                                          │ │
│  │  🔍 Topic Extraction & Trends                                        │ │
│  │  🤖 AI-Generated Insights                                            │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  [📧 Notify Me When Available]                                             │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## Common UI Patterns & Components

### 1. Navigation Menu (Sidebar)
```
┌──────────────────┐
│ [☰] WhatsApp OS  │
├──────────────────┤
│ 🏠 Dashboard     │
│ 💬 Messages      │
│ 👥 Contacts      │
│ 📁 Groups        │
│ 🎯 Channel Rules │
│ 📨 Planner       │
│ 📊 Analytics     │
│ ⚙️ Config        │
├──────────────────┤
│ 👤 Your Name     │
│ 🟢 Connected     │
└──────────────────┘
```

### 2. Status Indicators
- 🟢 Online/Connected
- 🟡 Connecting
- 🔴 Disconnected/Error
- ⚪ Offline
- ✓ Sent
- ✓✓ Delivered
- ✓✓✓ Read

### 3. Action Buttons
- Primary: Blue/purple background, white text
- Secondary: White background, gray border
- Danger: Red text/background for delete/cancel
- Success: Green for approve/confirm

### 4. Cards
- White background
- Subtle shadow
- Rounded corners
- Clear hierarchy (title, content, actions)

### 5. Color Palette
- Primary: #7C3AED (Purple)
- Success: #10B981 (Green)
- Warning: #F59E0B (Amber)
- Danger: #EF4444 (Red)
- Info: #3B82F6 (Blue)
- Gray: #6B7280 (Neutral)

### 6. Typography
- Headings: Bold, 24-32px
- Body: Regular, 14-16px
- Small: 12-14px
- Monospace: For IDs, phone numbers

---

## Responsive Design Notes

### Mobile (< 768px):
- Stack sidebar and content vertically
- Hide sidebar by default, show with hamburger menu
- Reduce padding/margins
- Full-width cards
- Simplify tables to lists

### Tablet (768px - 1024px):
- Narrow sidebar
- 2-column grid layouts
- Moderate spacing

### Desktop (> 1024px):
- Full sidebar
- 3+ column grids where appropriate
- Maximum content width: 1400px
- Generous spacing

---

## Accessibility Considerations

1. **Keyboard Navigation**: All interactive elements accessible via Tab
2. **ARIA Labels**: Proper labels for screen readers
3. **Color Contrast**: WCAG AA compliance minimum
4. **Focus Indicators**: Clear visual focus states
5. **Alt Text**: All images and icons have descriptions
6. **Loading States**: Clear loading indicators with text
7. **Error Messages**: Clear, actionable error descriptions

---

## Performance Optimizations

1. **Virtual Scrolling**: For long message/contact lists
2. **Lazy Loading**: Images and media load on demand
3. **Pagination**: Server-side pagination for large datasets
4. **Debounced Search**: 300ms debounce on search inputs
5. **Cached Queries**: TanStack Query caching for repeated data
6. **Optimistic Updates**: Instant UI feedback before server confirmation

---

This completes the UI layout specifications for the WhatsApp Life OS system!
