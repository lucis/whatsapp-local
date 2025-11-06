# WhatsApp Message Planner - Batch Contextual Messaging

## Overview

This plan extends the WhatsApp Web.js integration (see `mcp-whatsapp.md`) with an intelligent message planning system. It enables users to send contextual, personalized messages to multiple groups/contacts based on sophisticated filtering and AI-generated content.

## Use Case: Leaving Company Groups

**Primary Scenario:**
"I'm leaving the company and need to send farewell messages to 20+ WhatsApp groups. Each message should be personalized based on the group context (team, project, clients) and the people in each group."

**User Journey:**
1. **Filter Groups**: Select groups by members, tags, or manually
2. **Create Message Plan**: Define templates and rules for message generation
3. **AI Generation**: Generate personalized messages for each group
4. **Review & Edit**: Review and customize each message before sending
5. **Schedule & Send**: Send all messages at once or schedule for later
6. **Track Status**: Monitor delivery and acknowledgment

## Architecture Extension

This module extends the existing WhatsApp infrastructure with:
- New database tables for message planning
- AI-powered message generation tools
- Group filtering and selection tools
- Batch sending orchestration
- Review and approval workflow
- Dedicated UI for message planning

## Phase 1: Database Schema Extension

### New Tables

```typescript
// server/schema.ts additions

// Message Plans (containers for batch messaging campaigns)
export const messagePlansTable = sqliteTable("message_plans", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name").notNull(), // e.g., "Farewell Messages - Company Exit"
  description: text("description"),
  status: text("status").notNull().default("draft"), // draft, generating, reviewing, scheduled, sending, completed, cancelled
  totalRecipients: integer("total_recipients").default(0),
  completedRecipients: integer("completed_recipients").default(0),
  scheduledAt: integer("scheduled_at", { mode: "timestamp" }), // When to send (null = send now)
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
  completedAt: integer("completed_at", { mode: "timestamp" }),
});

// Message Templates (base templates for generation)
export const messageTemplatesTable = sqliteTable("message_templates", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name").notNull(), // e.g., "Farewell Template - Team"
  category: text("category"), // farewell, announcement, invitation, reminder, etc.
  basePrompt: text("base_prompt").notNull(), // AI generation prompt
  variables: text("variables"), // JSON: ["groupName", "memberNames", "projectName"]
  exampleOutput: text("example_output"), // Example of generated message
  isActive: integer("is_active").default(1), // Boolean
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Message Plan Recipients (groups/contacts in a plan)
export const messagePlanRecipientsTable = sqliteTable("message_plan_recipients", {
  id: integer("id").primaryKey(),
  planId: integer("plan_id").notNull().references(() => messagePlansTable.id),
  recipientType: text("recipient_type").notNull(), // "group", "contact"
  recipientId: integer("recipient_id").notNull(), // Reference to groups or contacts table
  recipientWhatsappId: text("recipient_whatsapp_id").notNull(), // Denormalized for quick access
  recipientName: text("recipient_name"), // Denormalized display name
  
  // Message content
  generatedMessage: text("generated_message"), // AI-generated message
  finalMessage: text("final_message"), // User-edited final version
  templateId: integer("template_id").references(() => messageTemplatesTable.id),
  generationContext: text("generation_context"), // JSON: context used for generation
  
  // Status tracking
  status: text("status").notNull().default("pending"), // pending, generated, reviewed, approved, sending, sent, failed
  sentAt: integer("sent_at", { mode: "timestamp" }),
  messageWhatsappId: text("message_whatsapp_id"), // ID of sent message (links to messagesTable)
  ack: integer("ack").default(0), // Delivery acknowledgment status
  errorMessage: text("error_message"),
  
  // AI metadata
  aiGeneratedAt: integer("ai_generated_at", { mode: "timestamp" }),
  aiModel: text("ai_model"),
  aiTokens: integer("ai_tokens"),
  
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Group Filters (saved filter configurations)
export const groupFiltersTable = sqliteTable("group_filters", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name").notNull(), // e.g., "Work Groups with Engineering Team"
  description: text("description"),
  filterConfig: text("filter_config").notNull(), // JSON: filter rules
  resultCount: integer("result_count").default(0), // Cached count
  lastAppliedAt: integer("last_applied_at", { mode: "timestamp" }),
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Group Tags (for organizing and filtering groups)
export const groupTagsTable = sqliteTable("group_tags", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name").notNull(), // e.g., "Work", "Family", "Clients", "Projects"
  color: text("color"), // Hex color for UI
  icon: text("icon"), // Icon name for UI
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
});

// Group Tag Assignments (many-to-many)
export const groupTagAssignmentsTable = sqliteTable("group_tag_assignments", {
  id: integer("id").primaryKey(),
  groupId: integer("group_id").notNull().references(() => groupsTable.id),
  tagId: integer("tag_id").notNull().references(() => groupTagsTable.id),
  assignedAt: integer("assigned_at", { mode: "timestamp" }).notNull(),
});

// Generation Rules (rules for contextual message generation)
export const generationRulesTable = sqliteTable("generation_rules", {
  id: integer("id").primaryKey(),
  accountId: integer("account_id").notNull().references(() => whatsappAccountsTable.id),
  name: text("name").notNull(), // e.g., "Formal Tone for Client Groups"
  description: text("description"),
  condition: text("condition").notNull(), // JSON: condition to apply rule
  modification: text("modification").notNull(), // JSON: what to modify in prompt
  priority: integer("priority").default(0), // Higher priority rules apply first
  isActive: integer("is_active").default(1), // Boolean
  createdAt: integer("created_at", { mode: "timestamp" }).notNull(),
  updatedAt: integer("updated_at", { mode: "timestamp" }).notNull(),
});

// Indexes
export const messagePlansAccountIdIndex = index("message_plans_account_id_idx").on(messagePlansTable.accountId);
export const messagePlanRecipientsplanIdIndex = index("message_plan_recipients_plan_id_idx").on(messagePlanRecipientsTable.planId);
export const groupFiltersAccountIdIndex = index("group_filters_account_id_idx").on(groupFiltersTable.accountId);
export const groupTagAssignmentsGroupIdIndex = index("group_tag_assignments_group_id_idx").on(groupTagAssignmentsTable.groupId);
```

## Phase 2: Domain Tools - Message Planning

### New Domain: `server/tools/whatsapp/planner/`

```
server/tools/whatsapp/planner/
  index.ts          - Aggregates all planner tools
  plans.ts          - Message plan management
  templates.ts      - Template management
  filters.ts        - Group filtering and selection
  generation.ts     - AI message generation
  review.ts         - Review and approval
  sending.ts        - Batch sending orchestration
  tags.ts           - Group tagging
```

### Tool Definitions

#### Message Plan Management (`plans.ts`)

```typescript
// Tool: WHATSAPP_CREATE_MESSAGE_PLAN
// Creates a new message plan
inputSchema: {
  accountId: number;
  name: string;
  description?: string;
  scheduledAt?: string; // ISO datetime
}
outputSchema: {
  planId: number;
  status: string;
}

// Tool: WHATSAPP_GET_MESSAGE_PLANS
// Retrieves message plans with filtering
inputSchema: {
  accountId: number;
  status?: string;
  fromDate?: string;
  toDate?: string;
  limit?: number;
  offset?: number;
}
outputSchema: {
  plans: MessagePlanInfo[];
  total: number;
}

// Tool: WHATSAPP_GET_MESSAGE_PLAN_DETAILS
// Gets detailed info about a specific plan
inputSchema: {
  planId: number;
}
outputSchema: {
  plan: MessagePlanInfo;
  recipients: RecipientInfo[];
  stats: {
    total: number;
    pending: number;
    generated: number;
    reviewed: number;
    approved: number;
    sent: number;
    failed: number;
  };
}

// Tool: WHATSAPP_UPDATE_MESSAGE_PLAN
// Updates plan details
inputSchema: {
  planId: number;
  name?: string;
  description?: string;
  status?: string;
  scheduledAt?: string;
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_DELETE_MESSAGE_PLAN
// Deletes a message plan (only if status is draft)
inputSchema: {
  planId: number;
}
outputSchema: {
  success: boolean;
}
```

#### Group Filtering & Selection (`filters.ts`)

```typescript
// Tool: WHATSAPP_FILTER_GROUPS_BY_MEMBERS
// Filters groups by specific members
inputSchema: {
  accountId: number;
  memberWhatsappIds: string[]; // Contact IDs to search for
  matchMode: "any" | "all"; // Match any member or all members
  excludeGroups?: string[]; // Groups to exclude
}
outputSchema: {
  groups: FilteredGroupInfo[];
  total: number;
}

// Tool: WHATSAPP_FILTER_GROUPS_BY_TAGS
// Filters groups by tags
inputSchema: {
  accountId: number;
  tagIds: number[];
  matchMode: "any" | "all";
}
outputSchema: {
  groups: FilteredGroupInfo[];
  total: number;
}

// Tool: WHATSAPP_FILTER_GROUPS_ADVANCED
// Advanced filtering with multiple criteria
inputSchema: {
  accountId: number;
  memberWhatsappIds?: string[];
  tagIds?: number[];
  nameSearch?: string;
  hasMessages?: boolean; // Only groups with messages
  lastMessageAfter?: string; // Date filter
  memberCountMin?: number;
  memberCountMax?: number;
  excludeGroupIds?: number[];
}
outputSchema: {
  groups: FilteredGroupInfo[];
  total: number;
  filterSummary: FilterSummary;
}

// Tool: WHATSAPP_SAVE_GROUP_FILTER
// Saves a filter configuration for reuse
inputSchema: {
  accountId: number;
  name: string;
  description?: string;
  filterConfig: FilterConfig;
}
outputSchema: {
  filterId: number;
}

// Tool: WHATSAPP_APPLY_SAVED_FILTER
// Applies a previously saved filter
inputSchema: {
  filterId: number;
}
outputSchema: {
  groups: FilteredGroupInfo[];
  total: number;
}

// Tool: WHATSAPP_GET_SAVED_FILTERS
// Lists saved filters
inputSchema: {
  accountId: number;
}
outputSchema: {
  filters: GroupFilterInfo[];
}
```

#### Group Tagging (`tags.ts`)

```typescript
// Tool: WHATSAPP_CREATE_GROUP_TAG
// Creates a new tag
inputSchema: {
  accountId: number;
  name: string;
  color?: string;
  icon?: string;
}
outputSchema: {
  tagId: number;
}

// Tool: WHATSAPP_ASSIGN_TAGS_TO_GROUPS
// Assigns tags to multiple groups
inputSchema: {
  groupIds: number[];
  tagIds: number[];
}
outputSchema: {
  assigned: number;
}

// Tool: WHATSAPP_GET_GROUP_TAGS
// Gets all tags for an account
inputSchema: {
  accountId: number;
}
outputSchema: {
  tags: GroupTagInfo[];
}

// Tool: WHATSAPP_GET_GROUPS_BY_TAG
// Gets groups with a specific tag
inputSchema: {
  tagId: number;
}
outputSchema: {
  groups: GroupInfo[];
}

// Tool: WHATSAPP_BULK_TAG_ASSIGNMENT
// Intelligent bulk tagging based on criteria
inputSchema: {
  accountId: number;
  criteria: {
    namePatterns?: string[]; // Regex patterns
    memberPatterns?: string[];
    hasKeywords?: string[];
  };
  tagId: number;
  dryRun?: boolean; // Preview without applying
}
outputSchema: {
  matchedGroups: GroupInfo[];
  totalMatched: number;
  applied: boolean;
}
```

#### Message Templates (`templates.ts`)

```typescript
// Tool: WHATSAPP_CREATE_MESSAGE_TEMPLATE
// Creates a new message template
inputSchema: {
  accountId: number;
  name: string;
  category: string;
  basePrompt: string;
  variables: string[]; // ["groupName", "memberNames", etc.]
  exampleOutput?: string;
}
outputSchema: {
  templateId: number;
}

// Tool: WHATSAPP_GET_MESSAGE_TEMPLATES
// Gets templates with filtering
inputSchema: {
  accountId: number;
  category?: string;
  isActive?: boolean;
}
outputSchema: {
  templates: MessageTemplateInfo[];
}

// Tool: WHATSAPP_UPDATE_MESSAGE_TEMPLATE
// Updates a template
inputSchema: {
  templateId: number;
  name?: string;
  basePrompt?: string;
  variables?: string[];
  exampleOutput?: string;
  isActive?: boolean;
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_TEST_TEMPLATE
// Tests template with sample data
inputSchema: {
  templateId: number;
  sampleData: Record<string, any>;
}
outputSchema: {
  generatedMessage: string;
  tokensUsed: number;
}
```

#### AI Message Generation (`generation.ts`)

```typescript
// Tool: WHATSAPP_GENERATE_MESSAGE_FOR_GROUP
// Generates a personalized message for a specific group
inputSchema: {
  accountId: number;
  groupId: number;
  templateId: number;
  customContext?: Record<string, any>;
  rules?: number[]; // Generation rule IDs to apply
}
outputSchema: {
  generatedMessage: string;
  context: GenerationContext;
  tokensUsed: number;
  model: string;
}

// Tool: WHATSAPP_BULK_GENERATE_MESSAGES
// Generates messages for all groups in a plan
inputSchema: {
  planId: number;
  templateId: number;
  parallelLimit?: number; // Max concurrent generations
}
outputSchema: {
  generated: number;
  failed: number;
  totalTokens: number;
  errors: ErrorInfo[];
}

// Tool: WHATSAPP_GET_GROUP_CONTEXT
// Extracts context from a group for message generation
inputSchema: {
  groupId: number;
  includeMembers?: boolean;
  includeRecentMessages?: number; // Number of recent messages to include
  includeTags?: boolean;
}
outputSchema: {
  groupContext: {
    name: string;
    description: string;
    memberCount: number;
    members?: ContactInfo[];
    recentTopics?: string[];
    tags?: string[];
    lastInteraction?: string;
    conversationTone?: "formal" | "casual" | "mixed";
  };
}

// Tool: WHATSAPP_REGENERATE_MESSAGE
// Regenerates a message with different parameters
inputSchema: {
  recipientId: number; // message_plan_recipients.id
  tone?: "formal" | "casual" | "friendly" | "professional";
  length?: "short" | "medium" | "long";
  customInstructions?: string;
}
outputSchema: {
  generatedMessage: string;
  tokensUsed: number;
}
```

#### Generation Rules (`generation.ts` - continued)

```typescript
// Tool: WHATSAPP_CREATE_GENERATION_RULE
// Creates a conditional rule for message generation
inputSchema: {
  accountId: number;
  name: string;
  description?: string;
  condition: {
    type: "tag" | "member" | "groupName" | "memberCount" | "custom";
    operator: "equals" | "contains" | "greaterThan" | "lessThan" | "matches";
    value: any;
  };
  modification: {
    type: "toneChange" | "addContent" | "removeContent" | "replaceContent";
    value: string;
  };
  priority?: number;
}
outputSchema: {
  ruleId: number;
}

// Tool: WHATSAPP_GET_GENERATION_RULES
// Gets all generation rules
inputSchema: {
  accountId: number;
  isActive?: boolean;
}
outputSchema: {
  rules: GenerationRuleInfo[];
}

// Example Rules:
// - If group has tag "Clients" → Use formal tone
// - If group has < 5 members → Use casual tone
// - If group name contains "Project" → Mention project completion
// - If group has specific member → Add personal note
```

#### Review & Approval (`review.ts`)

```typescript
// Tool: WHATSAPP_ADD_RECIPIENTS_TO_PLAN
// Adds groups/contacts to a message plan
inputSchema: {
  planId: number;
  recipients: Array<{
    type: "group" | "contact";
    id: number;
    whatsappId: string;
  }>;
  templateId?: number; // Default template for all
}
outputSchema: {
  added: number;
  duplicates: number;
}

// Tool: WHATSAPP_GET_PLAN_RECIPIENTS_FOR_REVIEW
// Gets recipients in review status
inputSchema: {
  planId: number;
  status?: string;
  limit?: number;
  offset?: number;
}
outputSchema: {
  recipients: RecipientDetailInfo[];
  total: number;
}

// Tool: WHATSAPP_UPDATE_RECIPIENT_MESSAGE
// Updates the final message for a recipient
inputSchema: {
  recipientId: number;
  finalMessage: string;
  status?: "reviewed" | "approved";
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_APPROVE_RECIPIENT
// Approves a message for sending
inputSchema: {
  recipientId: number;
}
outputSchema: {
  success: boolean;
}

// Tool: WHATSAPP_BULK_APPROVE_RECIPIENTS
// Approves multiple messages at once
inputSchema: {
  recipientIds: number[];
}
outputSchema: {
  approved: number;
}

// Tool: WHATSAPP_REJECT_RECIPIENT
// Rejects/removes a recipient from plan
inputSchema: {
  recipientId: number;
  reason?: string;
}
outputSchema: {
  success: boolean;
}
```

#### Batch Sending (`sending.ts`)

```typescript
// Tool: WHATSAPP_START_SENDING
// Starts sending messages for a plan
inputSchema: {
  planId: number;
  sendDelay?: number; // Milliseconds between messages (default: 2000)
  maxConcurrent?: number; // Max concurrent sends (default: 1)
  skipUnreviewed?: boolean; // Skip messages without review
}
outputSchema: {
  started: boolean;
  recipientsToSend: number;
  estimatedDuration: number; // Seconds
}

// Tool: WHATSAPP_SEND_SINGLE_PLANNED_MESSAGE
// Sends a single message from the plan
inputSchema: {
  recipientId: number;
}
outputSchema: {
  sent: boolean;
  messageWhatsappId?: string;
  error?: string;
}

// Tool: WHATSAPP_GET_SENDING_PROGRESS
// Gets real-time progress of sending
inputSchema: {
  planId: number;
}
outputSchema: {
  status: string;
  total: number;
  sent: number;
  failed: number;
  remaining: number;
  currentRecipient?: RecipientInfo;
  estimatedCompletion?: string;
}

// Tool: WHATSAPP_PAUSE_SENDING
// Pauses the sending process
inputSchema: {
  planId: number;
}
outputSchema: {
  paused: boolean;
  sentCount: number;
  remainingCount: number;
}

// Tool: WHATSAPP_RESUME_SENDING
// Resumes paused sending
inputSchema: {
  planId: number;
}
outputSchema: {
  resumed: boolean;
}

// Tool: WHATSAPP_CANCEL_SENDING
// Cancels the entire plan
inputSchema: {
  planId: number;
}
outputSchema: {
  cancelled: boolean;
  sentCount: number;
  cancelledCount: number;
}

// Tool: WHATSAPP_RETRY_FAILED_MESSAGES
// Retries sending failed messages
inputSchema: {
  planId: number;
}
outputSchema: {
  retried: number;
  successful: number;
  failed: number;
}
```

## Phase 3: Workflows

### Workflow: COMPLETE_MESSAGE_PLANNING_PIPELINE

```typescript
// Orchestrates the entire message planning process
Steps:
1. CREATE_MESSAGE_PLAN
2. FILTER_AND_SELECT_GROUPS
3. ADD_RECIPIENTS_TO_PLAN
4. BULK_GENERATE_MESSAGES (with AI)
5. REVIEW_LOOP (user interaction)
6. BULK_APPROVE_RECIPIENTS
7. START_SENDING (with progress tracking)
8. COMPLETE_PLAN

Input: {
  accountId: number;
  planName: string;
  filterCriteria: FilterConfig;
  templateId: number;
  scheduledAt?: string;
}

Output: {
  planId: number;
  status: string;
  stats: CompletionStats;
}
```

### Workflow: SMART_GROUP_SELECTION

```typescript
// Intelligently selects groups based on complex criteria
Steps:
1. FILTER_GROUPS_BY_MEMBERS (if members specified)
2. FILTER_GROUPS_BY_TAGS (if tags specified)
3. APPLY_ADVANCED_FILTERS
4. DEDUPLICATE_RESULTS
5. RANK_BY_RELEVANCE (optional AI ranking)
6. RETURN_SORTED_GROUPS

Input: {
  accountId: number;
  criteria: SelectionCriteria;
  rankByRelevance?: boolean;
}

Output: {
  groups: GroupInfo[];
  selectionReasoning?: string[];
}
```

### Workflow: CONTEXTUAL_MESSAGE_GENERATION

```typescript
// Generates highly contextual messages
Steps:
1. GET_GROUP_CONTEXT (members, tags, recent messages)
2. APPLY_GENERATION_RULES
3. BUILD_AI_PROMPT (with context)
4. CALL_AI_GENERATE_OBJECT (Deco AI tool)
5. POST_PROCESS_MESSAGE
6. VALIDATE_MESSAGE
7. SAVE_GENERATED_MESSAGE

Input: {
  groupId: number;
  templateId: number;
  customContext?: Record<string, any>;
}

Output: {
  generatedMessage: string;
  context: GenerationContext;
  appliedRules: RuleInfo[];
}
```

## Phase 4: AI Integration

### AI Prompt Engineering

**Base Template Example:**

```typescript
const farewellTemplatePrompt = `
You are helping a user craft a farewell message to a WhatsApp group.

Context:
- Group Name: {groupName}
- Group Type: {groupType}
- Member Count: {memberCount}
- Key Members: {keyMembers}
- Group Tags: {tags}
- Recent Topics: {recentTopics}
- Last Interaction: {lastInteraction}

User's Situation:
- Reason for leaving: {reason}
- Tone preference: {tone}
- Relationship level: {relationship}

Generate a warm, authentic farewell message that:
1. Acknowledges the group's purpose and shared experiences
2. Expresses gratitude appropriately
3. Maintains the specified tone ({tone})
4. Is personal but professional
5. Includes a way to stay connected (if appropriate)
6. Is between {minLength} and {maxLength} words

Generate ONLY the message text, no explanations.
`;
```

**AI Schema for Generation:**

```typescript
const messageGenerationSchema = {
  type: 'object',
  properties: {
    message: {
      type: 'string',
      description: 'The generated farewell message'
    },
    tone: {
      type: 'string',
      enum: ['formal', 'casual', 'friendly', 'professional'],
      description: 'Detected tone of the message'
    },
    reasoning: {
      type: 'string',
      description: 'Brief explanation of personalization choices'
    },
    suggestions: {
      type: 'array',
      items: { type: 'string' },
      description: 'Alternative phrasings or additions'
    }
  },
  required: ['message', 'tone']
};
```

**Using Deco AI Tool:**

```typescript
// In generation.ts tool
execute: async ({ context }) => {
  const groupContext = await getGroupContext(context.groupId);
  const template = await getTemplate(context.templateId);
  
  // Build prompt with context
  const prompt = buildPrompt(template.basePrompt, {
    ...groupContext,
    ...context.customContext
  });
  
  // Call Deco AI via client
  const result = await client.AI_GENERATE_OBJECT({
    messages: [{
      role: 'system',
      content: 'You are an expert at crafting personalized, contextual messages.'
    }, {
      role: 'user',
      content: prompt
    }],
    schema: messageGenerationSchema,
    temperature: 0.7,
    maxTokens: 500
  });
  
  return {
    generatedMessage: result.object.message,
    context: groupContext,
    tokensUsed: result.usage.totalTokens,
    model: 'gpt-4o-mini'
  };
}
```

## Phase 5: Frontend UI - Message Planner View

### View Architecture

```
view/src/routes/whatsapp/
  planner/
    index.tsx              - Main planner dashboard
    new-plan.tsx           - Create new plan wizard
    group-selection.tsx    - Group filtering & selection UI
    message-generation.tsx - AI generation interface
    review.tsx             - Review & edit messages
    sending.tsx            - Send progress monitoring
  
  components/
    GroupFilter.tsx        - Filter builder component
    GroupCard.tsx          - Group display card
    MessageEditor.tsx      - Rich text message editor
    GenerationSettings.tsx - AI generation controls
    SendingProgress.tsx    - Real-time progress tracker
    TemplateSelector.tsx   - Template chooser
```

### View: Message Planner Dashboard

```typescript
// view/src/routes/whatsapp/planner/index.tsx

function MessagePlannerDashboard() {
  const { user } = useUser();
  const { data: plans, isLoading } = useMessagePlans(user?.accountId);
  
  return (
    <div className="container mx-auto p-8">
      <header className="flex justify-between items-center mb-8">
        <div>
          <h1 className="text-3xl font-bold">Message Planner</h1>
          <p className="text-gray-600">
            Send personalized messages to multiple groups
          </p>
        </div>
        <Link 
          to="/whatsapp/planner/new" 
          className="btn-primary"
        >
          <Plus className="w-4 h-4 mr-2" />
          New Message Plan
        </Link>
      </header>
      
      {/* Active Plans */}
      <section className="mb-8">
        <h2 className="text-xl font-semibold mb-4">Active Plans</h2>
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          {plans?.active.map(plan => (
            <PlanCard key={plan.id} plan={plan} />
          ))}
        </div>
      </section>
      
      {/* Completed Plans */}
      <section>
        <h2 className="text-xl font-semibold mb-4">History</h2>
        <PlanHistoryTable plans={plans?.completed} />
      </section>
    </div>
  );
}
```

### View: Group Selection Wizard

```typescript
// view/src/routes/whatsapp/planner/group-selection.tsx

function GroupSelectionWizard() {
  const [filterMode, setFilterMode] = useState<'members' | 'tags' | 'advanced'>('members');
  const [selectedGroups, setSelectedGroups] = useState<Set<number>>(new Set());
  const [filterCriteria, setFilterCriteria] = useState<FilterCriteria>({});
  
  const { data: filteredGroups } = useFilterGroups(filterCriteria);
  
  return (
    <div className="container mx-auto p-8">
      {/* Step Indicator */}
      <StepIndicator currentStep={1} totalSteps={5} />
      
      <div className="grid grid-cols-12 gap-6">
        {/* Filter Panel */}
        <aside className="col-span-3 bg-white rounded-lg shadow p-6">
          <h3 className="font-semibold mb-4">Filter Groups</h3>
          
          {/* Filter Mode Selector */}
          <Tabs value={filterMode} onValueChange={setFilterMode}>
            <TabsList>
              <TabsTrigger value="members">By Members</TabsTrigger>
              <TabsTrigger value="tags">By Tags</TabsTrigger>
              <TabsTrigger value="advanced">Advanced</TabsTrigger>
            </TabsList>
          </Tabs>
          
          {/* Filter Controls */}
          {filterMode === 'members' && (
            <MemberFilterPanel 
              onChange={setFilterCriteria}
            />
          )}
          
          {filterMode === 'tags' && (
            <TagFilterPanel 
              onChange={setFilterCriteria}
            />
          )}
          
          {filterMode === 'advanced' && (
            <AdvancedFilterPanel 
              onChange={setFilterCriteria}
            />
          )}
          
          {/* Saved Filters */}
          <div className="mt-6">
            <h4 className="font-medium mb-2">Saved Filters</h4>
            <SavedFiltersList 
              onApply={setFilterCriteria}
            />
          </div>
        </aside>
        
        {/* Groups Grid */}
        <main className="col-span-9">
          <header className="flex justify-between items-center mb-4">
            <div>
              <h2 className="text-2xl font-bold">Select Groups</h2>
              <p className="text-gray-600">
                {filteredGroups?.total || 0} groups found • 
                {selectedGroups.size} selected
              </p>
            </div>
            
            <div className="flex gap-2">
              <Button 
                variant="outline"
                onClick={() => {
                  filteredGroups?.groups.forEach(g => 
                    selectedGroups.add(g.id)
                  );
                  setSelectedGroups(new Set(selectedGroups));
                }}
              >
                Select All
              </Button>
              <Button 
                variant="outline"
                onClick={() => setSelectedGroups(new Set())}
              >
                Clear Selection
              </Button>
            </div>
          </header>
          
          {/* Groups Grid */}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            {filteredGroups?.groups.map(group => (
              <GroupSelectionCard
                key={group.id}
                group={group}
                isSelected={selectedGroups.has(group.id)}
                onToggle={(id) => {
                  const newSelection = new Set(selectedGroups);
                  if (newSelection.has(id)) {
                    newSelection.delete(id);
                  } else {
                    newSelection.add(id);
                  }
                  setSelectedGroups(newSelection);
                }}
              />
            ))}
          </div>
        </main>
      </div>
      
      {/* Action Bar */}
      <footer className="fixed bottom-0 left-0 right-0 bg-white border-t p-4">
        <div className="container mx-auto flex justify-between items-center">
          <Button variant="outline">
            Back
          </Button>
          <div className="text-sm text-gray-600">
            {selectedGroups.size} groups selected
          </div>
          <Button 
            disabled={selectedGroups.size === 0}
            onClick={() => navigateToNextStep(selectedGroups)}
          >
            Next: Generate Messages
            <ChevronRight className="w-4 h-4 ml-2" />
          </Button>
        </div>
      </footer>
    </div>
  );
}
```

### View: Message Review & Edit

```typescript
// view/src/routes/whatsapp/planner/review.tsx

function MessageReviewView() {
  const { planId } = useParams();
  const { data: plan } = useMessagePlanDetails(planId);
  const { data: recipients } = usePlanRecipients(planId);
  const [currentIndex, setCurrentIndex] = useState(0);
  const updateMutation = useUpdateRecipientMessage();
  
  const currentRecipient = recipients?.[currentIndex];
  
  return (
    <div className="container mx-auto p-8">
      <header className="mb-8">
        <h1 className="text-3xl font-bold">{plan?.name}</h1>
        <p className="text-gray-600">
          Review and edit messages before sending
        </p>
      </header>
      
      <div className="grid grid-cols-12 gap-6">
        {/* Recipient List */}
        <aside className="col-span-3">
          <div className="bg-white rounded-lg shadow p-4">
            <h3 className="font-semibold mb-4">
              Recipients ({recipients?.length})
            </h3>
            <div className="space-y-2">
              {recipients?.map((recipient, index) => (
                <RecipientListItem
                  key={recipient.id}
                  recipient={recipient}
                  isActive={index === currentIndex}
                  onClick={() => setCurrentIndex(index)}
                />
              ))}
            </div>
          </div>
        </aside>
        
        {/* Message Editor */}
        <main className="col-span-9">
          <div className="bg-white rounded-lg shadow p-6">
            {/* Group Info */}
            <div className="mb-6">
              <div className="flex items-center gap-4 mb-4">
                <Avatar src={currentRecipient?.groupPictureUrl} />
                <div>
                  <h2 className="text-xl font-semibold">
                    {currentRecipient?.recipientName}
                  </h2>
                  <p className="text-sm text-gray-600">
                    {currentRecipient?.memberCount} members
                  </p>
                </div>
              </div>
              
              {/* Group Context Tags */}
              <div className="flex flex-wrap gap-2">
                {currentRecipient?.tags?.map(tag => (
                  <Badge key={tag} variant="secondary">
                    {tag}
                  </Badge>
                ))}
              </div>
            </div>
            
            {/* AI Generated Message */}
            <div className="mb-4">
              <label className="block text-sm font-medium mb-2">
                AI Generated Message
              </label>
              <div className="bg-gray-50 rounded-lg p-4 border border-gray-200">
                <p className="whitespace-pre-wrap">
                  {currentRecipient?.generatedMessage}
                </p>
              </div>
              <div className="mt-2 flex gap-2">
                <Button 
                  size="sm" 
                  variant="outline"
                  onClick={() => regenerateMessage(currentRecipient.id)}
                >
                  <RefreshCw className="w-4 h-4 mr-2" />
                  Regenerate
                </Button>
                <Button 
                  size="sm" 
                  variant="outline"
                  onClick={() => copyToEditor(currentRecipient.generatedMessage)}
                >
                  <Copy className="w-4 h-4 mr-2" />
                  Use This
                </Button>
              </div>
            </div>
            
            {/* Editable Message */}
            <div className="mb-6">
              <label className="block text-sm font-medium mb-2">
                Final Message
              </label>
              <Textarea
                value={currentRecipient?.finalMessage || ''}
                onChange={(e) => {
                  updateMutation.mutate({
                    recipientId: currentRecipient.id,
                    finalMessage: e.target.value,
                  });
                }}
                rows={8}
                className="font-sans"
                placeholder="Edit the message here..."
              />
              <div className="mt-2 text-sm text-gray-500">
                {currentRecipient?.finalMessage?.length || 0} characters
              </div>
            </div>
            
            {/* Generation Context (Collapsible) */}
            <Collapsible>
              <CollapsibleTrigger className="flex items-center gap-2 text-sm text-gray-600">
                <ChevronDown className="w-4 h-4" />
                View Generation Context
              </CollapsibleTrigger>
              <CollapsibleContent className="mt-2 text-sm bg-gray-50 rounded p-3">
                <pre className="text-xs">
                  {JSON.stringify(currentRecipient?.generationContext, null, 2)}
                </pre>
              </CollapsibleContent>
            </Collapsible>
            
            {/* Actions */}
            <div className="mt-6 flex justify-between">
              <Button 
                variant="outline"
                disabled={currentIndex === 0}
                onClick={() => setCurrentIndex(currentIndex - 1)}
              >
                <ChevronLeft className="w-4 h-4 mr-2" />
                Previous
              </Button>
              
              <div className="flex gap-2">
                <Button 
                  variant="outline"
                  onClick={() => skipRecipient(currentRecipient.id)}
                >
                  Skip
                </Button>
                <Button 
                  onClick={() => {
                    approveRecipient(currentRecipient.id);
                    if (currentIndex < recipients.length - 1) {
                      setCurrentIndex(currentIndex + 1);
                    }
                  }}
                >
                  <Check className="w-4 h-4 mr-2" />
                  Approve & Next
                </Button>
              </div>
              
              <Button 
                disabled={currentIndex === recipients.length - 1}
                onClick={() => setCurrentIndex(currentIndex + 1)}
              >
                Next
                <ChevronRight className="w-4 h-4 ml-2" />
              </Button>
            </div>
          </div>
        </main>
      </div>
      
      {/* Bulk Actions Bar */}
      <footer className="fixed bottom-0 left-0 right-0 bg-white border-t p-4">
        <div className="container mx-auto flex justify-between items-center">
          <div className="text-sm">
            Progress: {recipients?.filter(r => r.status === 'approved').length} / {recipients?.length} approved
          </div>
          <Button 
            size="lg"
            disabled={!allReviewed}
            onClick={() => navigateToSending()}
          >
            All Reviewed - Proceed to Send
            <Send className="w-4 h-4 ml-2" />
          </Button>
        </div>
      </footer>
    </div>
  );
}
```

### View: Sending Progress

```typescript
// view/src/routes/whatsapp/planner/sending.tsx

function SendingProgressView() {
  const { planId } = useParams();
  const { data: progress } = useSendingProgress(planId, {
    refetchInterval: 2000, // Poll every 2 seconds
  });
  const pauseMutation = usePauseSending();
  const resumeMutation = useResumeSending();
  
  const progressPercent = (progress?.sent / progress?.total) * 100;
  
  return (
    <div className="container max-w-4xl mx-auto p-8">
      <div className="bg-white rounded-lg shadow-lg p-8">
        <header className="text-center mb-8">
          <h1 className="text-3xl font-bold mb-2">Sending Messages</h1>
          <p className="text-gray-600">
            Please keep this page open while messages are being sent
          </p>
        </header>
        
        {/* Progress Ring */}
        <div className="flex justify-center mb-8">
          <CircularProgress 
            value={progressPercent} 
            size={200}
            strokeWidth={12}
          >
            <div className="text-center">
              <div className="text-4xl font-bold">
                {progress?.sent}
              </div>
              <div className="text-gray-600">
                of {progress?.total}
              </div>
            </div>
          </CircularProgress>
        </div>
        
        {/* Stats Grid */}
        <div className="grid grid-cols-4 gap-4 mb-8">
          <StatCard 
            label="Sent" 
            value={progress?.sent}
            icon={<CheckCircle className="text-green-500" />}
          />
          <StatCard 
            label="Failed" 
            value={progress?.failed}
            icon={<XCircle className="text-red-500" />}
          />
          <StatCard 
            label="Remaining" 
            value={progress?.remaining}
            icon={<Clock className="text-blue-500" />}
          />
          <StatCard 
            label="Est. Time" 
            value={formatDuration(progress?.estimatedCompletion)}
            icon={<Timer className="text-purple-500" />}
          />
        </div>
        
        {/* Current Recipient */}
        {progress?.currentRecipient && (
          <div className="bg-blue-50 border border-blue-200 rounded-lg p-4 mb-6">
            <div className="flex items-center gap-3">
              <Loader className="w-5 h-5 animate-spin text-blue-600" />
              <div>
                <div className="font-medium">
                  Sending to {progress.currentRecipient.name}...
                </div>
                <div className="text-sm text-gray-600">
                  {progress.currentRecipient.preview}
                </div>
              </div>
            </div>
          </div>
        )}
        
        {/* Recent Activity */}
        <div className="mb-6">
          <h3 className="font-semibold mb-3">Recent Activity</h3>
          <div className="space-y-2 max-h-60 overflow-y-auto">
            {progress?.recentActivity?.map((activity, index) => (
              <ActivityItem key={index} activity={activity} />
            ))}
          </div>
        </div>
        
        {/* Controls */}
        <div className="flex justify-center gap-4">
          {progress?.status === 'sending' ? (
            <Button 
              variant="outline"
              onClick={() => pauseMutation.mutate(planId)}
            >
              <Pause className="w-4 h-4 mr-2" />
              Pause Sending
            </Button>
          ) : progress?.status === 'paused' ? (
            <Button 
              onClick={() => resumeMutation.mutate(planId)}
            >
              <Play className="w-4 h-4 mr-2" />
              Resume Sending
            </Button>
          ) : null}
          
          <Button 
            variant="destructive"
            onClick={() => {
              if (confirm('Cancel sending? Messages already sent cannot be recalled.')) {
                cancelSending(planId);
              }
            }}
          >
            <XCircle className="w-4 h-4 mr-2" />
            Cancel
          </Button>
        </div>
        
        {/* Completion */}
        {progress?.status === 'completed' && (
          <div className="mt-8 bg-green-50 border border-green-200 rounded-lg p-6 text-center">
            <CheckCircle className="w-12 h-12 text-green-600 mx-auto mb-3" />
            <h2 className="text-2xl font-bold text-green-900 mb-2">
              All Messages Sent!
            </h2>
            <p className="text-green-700 mb-4">
              {progress.sent} messages were successfully delivered
            </p>
            <div className="flex justify-center gap-4">
              <Button 
                variant="outline"
                onClick={() => navigate(`/whatsapp/planner/${planId}/report`)}
              >
                View Report
              </Button>
              <Button onClick={() => navigate('/whatsapp/planner')}>
                Back to Dashboard
              </Button>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Phase 6: Integration with Bot

### Bot Sending Interface

The bot needs a new endpoint to send planned messages:

```javascript
// whatsapp-bot/src/handlers/planner.js

async function sendPlannedMessage(recipientWhatsappId, message) {
  try {
    const chat = await client.getChatById(recipientWhatsappId);
    const sentMessage = await chat.sendMessage(message);
    
    return {
      success: true,
      messageWhatsappId: sentMessage.id._serialized,
      timestamp: sentMessage.timestamp,
    };
  } catch (error) {
    return {
      success: false,
      error: error.message,
    };
  }
}

// Webhook endpoint for MCP server to call
app.post('/send-message', async (req, res) => {
  const { recipientWhatsappId, message, recipientId } = req.body;
  
  const result = await sendPlannedMessage(recipientWhatsappId, message);
  
  // Report back to MCP server
  await mcpClient.WHATSAPP_UPDATE_SENDING_STATUS({
    recipientId,
    ...result,
  });
  
  res.json(result);
});
```

## Phase 7: Testing Strategy

### Unit Tests

```typescript
// Test message generation with context
describe('WHATSAPP_GENERATE_MESSAGE_FOR_GROUP', () => {
  it('should generate personalized message based on group context', async () => {
    const result = await generateMessage({
      groupId: 1,
      templateId: 1,
    });
    
    expect(result.generatedMessage).toContain('group name');
    expect(result.tokensUsed).toBeGreaterThan(0);
  });
  
  it('should apply generation rules correctly', async () => {
    const result = await generateMessage({
      groupId: 1, // Group with "Clients" tag
      templateId: 1,
    });
    
    // Should apply formal tone rule
    expect(result.generatedMessage).toMatch(/formal language patterns/);
  });
});

// Test filtering logic
describe('WHATSAPP_FILTER_GROUPS_BY_MEMBERS', () => {
  it('should filter groups containing specific members', async () => {
    const result = await filterGroups({
      memberWhatsappIds: ['5511999999999@c.us'],
      matchMode: 'any',
    });
    
    expect(result.groups.length).toBeGreaterThan(0);
    expect(result.groups.every(g => 
      g.members.includes('5511999999999@c.us')
    )).toBe(true);
  });
});
```

### Integration Tests

```typescript
// Test full workflow
describe('Complete Message Planning Workflow', () => {
  it('should complete full cycle from creation to sending', async () => {
    // 1. Create plan
    const plan = await createPlan({
      accountId: 1,
      name: 'Test Plan',
    });
    
    // 2. Add recipients
    const added = await addRecipients({
      planId: plan.planId,
      recipients: testGroups,
    });
    
    // 3. Generate messages
    const generated = await bulkGenerate({
      planId: plan.planId,
      templateId: 1,
    });
    
    expect(generated.generated).toBe(testGroups.length);
    
    // 4. Approve all
    const approved = await bulkApprove({
      recipientIds: generated.recipientIds,
    });
    
    // 5. Start sending (mock)
    const sending = await startSending({
      planId: plan.planId,
    });
    
    expect(sending.started).toBe(true);
  });
});
```

## Phase 8: Example Use Cases

### Use Case 1: Company Exit Farewell

```typescript
// Template for company farewell
{
  name: "Company Exit Farewell",
  category: "farewell",
  basePrompt: `
    Generate a warm farewell message for leaving a company WhatsApp group.
    
    Group Context: {groupName}, {memberCount} members
    Key People: {keyMembers}
    Group Purpose: {groupPurpose}
    
    Message should:
    - Thank the team for collaboration
    - Acknowledge specific projects or achievements if relevant
    - Wish them continued success
    - Provide LinkedIn for staying connected
    - Maintain professional yet warm tone
    - Be 50-100 words
  `,
  variables: ["groupName", "memberCount", "keyMembers", "groupPurpose"]
}

// Generation Rules
[
  {
    name: "Formal for Client Groups",
    condition: { type: "tag", operator: "equals", value: "Clients" },
    modification: { type: "toneChange", value: "formal" }
  },
  {
    name: "Casual for Social Groups",
    condition: { type: "tag", operator: "equals", value: "Social" },
    modification: { type: "toneChange", value: "casual" }
  },
  {
    name: "Mention Projects",
    condition: { type: "tag", operator: "equals", value: "Projects" },
    modification: { 
      type: "addContent", 
      value: "Mention completed projects and deliverables" 
    }
  }
]
```

### Use Case 2: Event Invitation

```typescript
{
  name: "Event Invitation",
  category: "invitation",
  basePrompt: `
    Generate an event invitation message for a WhatsApp group.
    
    Event: {eventName}
    Date: {eventDate}
    Location: {eventLocation}
    Group: {groupName}
    
    Message should:
    - Announce the event clearly
    - Include date, time, location
    - Explain relevance to this specific group
    - Include RSVP instructions
    - Be enthusiastic and inviting
    - Be 75-150 words
  `,
  variables: ["eventName", "eventDate", "eventLocation", "groupName"]
}
```

### Use Case 3: Project Completion Announcement

```typescript
{
  name: "Project Completion",
  category: "announcement",
  basePrompt: `
    Generate a project completion announcement for a team group.
    
    Project: {projectName}
    Team: {groupName} ({memberCount} members)
    Key Contributors: {keyMembers}
    Achievements: {achievements}
    
    Message should:
    - Announce successful completion
    - Thank team members
    - Highlight key achievements
    - Celebrate the team's effort
    - Maintain professional yet celebratory tone
    - Be 100-200 words
  `,
  variables: ["projectName", "groupName", "memberCount", "keyMembers", "achievements"]
}
```

## Phase 9: Future Enhancements

### 9.1 Advanced AI Features

```typescript
// Tool: WHATSAPP_ANALYZE_GROUP_SENTIMENT
// Analyzes recent messages to determine group mood
inputSchema: {
  groupId: number;
  messageCount?: number;
}
outputSchema: {
  sentiment: "positive" | "neutral" | "negative" | "mixed";
  tone: "formal" | "casual" | "professional";
  topics: string[];
  suggestedMessageTone: string;
}

// Tool: WHATSAPP_SUGGEST_MESSAGE_IMPROVEMENTS
// AI suggests improvements to user's draft
inputSchema: {
  message: string;
  groupContext: GroupContext;
  targetTone?: string;
}
outputSchema: {
  suggestions: Array<{
    type: "grammar" | "tone" | "length" | "clarity";
    original: string;
    suggested: string;
    reason: string;
  }>;
}
```

### 9.2 Smart Scheduling

```typescript
// Tool: WHATSAPP_SUGGEST_BEST_SEND_TIME
// AI suggests optimal time to send based on group activity
inputSchema: {
  groupId: number;
}
outputSchema: {
  suggestedTime: string;
  reasoning: string;
  alternativeTimes: string[];
  activityPattern: ActivityPattern;
}
```

### 9.3 A/B Testing

```typescript
// Tool: WHATSAPP_CREATE_AB_TEST
// Creates multiple message variants for testing
inputSchema: {
  planId: number;
  variants: number; // Number of variants to generate
}
outputSchema: {
  variants: MessageVariant[];
}
```

### 9.4 Analytics & Insights

```typescript
// Tool: WHATSAPP_GET_PLAN_ANALYTICS
// Comprehensive analytics for sent plans
inputSchema: {
  planId: number;
}
outputSchema: {
  deliveryRate: number;
  readRate: number;
  responseRate: number;
  avgResponseTime: number;
  sentimentAnalysis: SentimentData;
  topResponses: string[];
}
```

## Phase 10: Implementation Roadmap

### Week 1: Database & Core Tools
- [ ] Create all database tables and migrations
- [ ] Implement group tagging tools
- [ ] Implement filter tools (members, tags, advanced)
- [ ] Test filtering logic thoroughly

### Week 2: Message Planning
- [ ] Implement plan management tools
- [ ] Implement template management tools
- [ ] Implement recipient management tools
- [ ] Test CRUD operations

### Week 3: AI Generation
- [ ] Implement context extraction
- [ ] Implement generation rules engine
- [ ] Implement AI message generation
- [ ] Test with various templates and contexts
- [ ] Fine-tune prompts

### Week 4: Review & Sending
- [ ] Implement review tools
- [ ] Implement batch sending orchestration
- [ ] Integrate with WhatsApp bot
- [ ] Test sending with delays and error handling

### Week 5: Frontend UI
- [ ] Create planner dashboard view
- [ ] Create group selection wizard
- [ ] Create message review interface
- [ ] Create sending progress monitor
- [ ] Add views to server/views.ts

### Week 6: Testing & Polish
- [ ] Write comprehensive unit tests
- [ ] Write integration tests
- [ ] Performance testing with large plans
- [ ] UI/UX refinements
- [ ] Documentation

## Success Criteria

- ✅ Can filter groups by members with high accuracy
- ✅ Can filter groups by tags and advanced criteria
- ✅ AI generates contextual, personalized messages
- ✅ Generation rules apply correctly
- ✅ Review interface is intuitive and fast
- ✅ Batch sending handles 100+ messages reliably
- ✅ Progress tracking is real-time and accurate
- ✅ Error handling is robust (retries, logging)
- ✅ UI is responsive and user-friendly
- ✅ Complete workflow takes < 10 minutes for 20 groups

## Configuration & Environment

### Additional Environment Variables

```bash
# AI Generation
AI_GENERATION_MODEL=gpt-4o-mini
AI_MAX_TOKENS=500
AI_TEMPERATURE=0.7

# Sending Configuration
SENDING_DELAY_MS=2000
SENDING_MAX_CONCURRENT=1
SENDING_RETRY_ATTEMPTS=3
```

## Security Considerations

1. **Message Content**: Never log full message content
2. **Rate Limiting**: Implement rate limits on AI generation
3. **User Permissions**: Verify user owns account before operations
4. **Message Approval**: Require explicit approval before sending
5. **Audit Trail**: Log all plan operations for accountability

## Monitoring & Alerts

- Alert on high AI generation costs
- Alert on sending failures (>10%)
- Monitor average generation time
- Track user engagement with feature
- Log all sent messages for accountability

---

**This complementary plan adds sophisticated message planning capabilities to the WhatsApp Life OS, enabling users to send personalized, contextual messages at scale with AI assistance and full control over the process.**
