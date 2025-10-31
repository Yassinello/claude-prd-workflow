---
name: create-prd
description: Create new PRD with interactive refinement and AI assistance
category: PRD Management
---

# Create PRD Command

Create a new Product Requirements Document with interactive questions to refine scope before generation.

## Purpose

Generate well-scoped PRDs through:
- Interactive clarifying questions
- AI-assisted content generation
- Standardized structure
- Automatic ID assignment
- Draft mode by default (review later on feature branch)

## Workflow

### Step 0: Check for Template Usage (NEW in v2.6)

**Check for --template flag**:

```bash
/create-prd --template=ecommerce "Add shopping cart feature"
```

**If template specified**:
1. List available templates if `--list-templates`
2. Load template YAML file
3. Pre-fill PRD sections from template
4. Skip to Step 3 (generation)

**Available Templates** (7 total):
- 🛒 `ecommerce` - Online store with cart, checkout, payments
- 💼 `saas` - SaaS app with auth, billing, dashboard
- 📱 `mobile-app` - Cross-platform mobile with offline support
- 🔌 `api-service` - REST/GraphQL API with documentation
- ⚙️ `admin-panel` - Admin dashboard with CRUD operations
- 📊 `analytics-dashboard` - Real-time analytics and reporting
- 🔗 `integration` - Third-party API integration with webhooks

**List templates**:
```bash
/create-prd --list-templates

📋 Available PRD Templates:

1. 🛒 ecommerce - E-Commerce Store
   Online store with product catalog, cart, checkout, and payments

2. 💼 saas - SaaS Application
   SaaS app with authentication, billing, and user dashboard

3. 📱 mobile-app - Mobile Application
   Cross-platform mobile app for iOS and Android

4. 🔌 api-service - API Service
   RESTful API or GraphQL service with documentation

5. ⚙️ admin-panel - Admin Dashboard
   Internal admin panel with CRUD and user management

6. 📊 analytics-dashboard - Analytics Dashboard
   Real-time analytics and reporting dashboard

7. 🔗 integration - Third-Party Integration
   Integration with third-party API and webhooks

Usage: /create-prd --template=<name> "Your feature description"
Example: /create-prd --template=ecommerce "Add shopping cart"
```

**Template Benefits**:
- Pre-filled acceptance criteria (P0, P1, P2)
- Tech stack recommendations
- Out-of-scope examples (prevent creep)
- Success metrics
- Risk analysis
- 50% faster PRD creation


### Step 0.5: Auto-Detect Feature Type (NEW in v0.3)

**Before generating PRD, analyze feature description for type hints**:

```bash
FEATURE_DESC="$1"

# Detection logic
if echo "$FEATURE_DESC" | grep -iE "experiment|poc|prototype|test|trial|pilot" > /dev/null; then
  FEATURE_TYPE="experiment"
  TEMPLATE_FILE="product/templates/experiment.md"
  EMOJI="🧪"
elif echo "$FEATURE_DESC" | grep -iE "fix|bug|hotfix|patch|update|tweak|small|minor|quick" > /dev/null; then
  FEATURE_TYPE="quick"
  TEMPLATE_FILE="product/templates/quick-feature.md"
  EMOJI="⚡"
else
  FEATURE_TYPE="full"
  TEMPLATE_FILE="product/templates/full-feature.md"
  EMOJI="🎯"
fi
```

**Display detection result**:

```markdown
🔍 **Feature Type Auto-Detected**: ${EMOJI} ${FEATURE_TYPE^}

Based on keywords in: "${FEATURE_DESC}"

Detected as: **${FEATURE_TYPE^} Feature**
- Template: ${TEMPLATE_FILE}
- Estimated: ${ESTIMATE}

Is this correct? (y/n/change)
> _
```

**Type characteristics**:

| Type | Keywords | Estimate | Template |
|------|----------|----------|----------|
| 🎯 Full | (default) | 5+ days | full-feature.md |
| ⚡ Quick | fix, bug, update, small | 1-3 days | quick-feature.md |
| 🧪 Experiment | experiment, poc, prototype | 1-2 weeks | experiment.md |

**If user says "change"**:
```markdown
Which type should this be?
  [1] 🎯 Full Feature (5+ days)
  [2] ⚡ Quick Ship (1-3 days)
  [3] 🧪 Experiment/POC (1-2 weeks)

Choose: _
```

**Why this matters**:
- **Different templates** for different feature sizes
- **Right level of detail** - Quick features don't need extensive docs
- **Clearer expectations** - Team knows scope upfront


### Step 1: Gather Feature Description

Ask user for brief feature description:
```markdown
What feature do you want to build?

Example: "Add OAuth2 authentication with Google and GitHub"

💡 Tip: Use --template flag for faster PRD creation!
   /create-prd --list-templates

> [User input]
```

### Step 2: Detect Feature Type & Ask Contextual Questions

**NEW**: AI detects feature type and asks **type-specific questions** instead of generic ones.

#### Step 2a: Classify Feature Type

AI analyzes description and classifies into one of these types:
- 🔐 **Authentication/Security** (login, OAuth, permissions, encryption)
- 💳 **Payment/Financial** (billing, subscriptions, invoices, PCI)
- 🎨 **UI/UX** (design, themes, layouts, components)
- 🔌 **API/Backend** (endpoints, services, data processing)
- 🗄️ **Database** (schema, migrations, data modeling)
- 🔗 **Integration** (third-party APIs, webhooks, SDKs)
- 🏗️ **Infrastructure** (CI/CD, deployment, monitoring)
- 📊 **Analytics/Reporting** (dashboards, metrics, exports)
- 🧪 **Testing/QA** (test frameworks, coverage, automation)
- 📝 **Simple/Generic** (unclear type or very simple)

#### Step 2b: Ask Type-Specific Questions (4-6 questions)

**For simple/well-defined features**: Skip questions, go straight to generation
**For complex features**: Ask targeted questions based on type

#### Example: Payment/Financial Feature

User input: "Add payment processing with Stripe"

```markdown
Great! I've detected this is a Payment/Financial feature. Let me ask a few specific questions...

❓ Question 1: Compliance & Security
Do you need PCI-DSS Level 1 compliance?
- Yes (we handle card data directly)
- No (use Stripe.js tokenization - recommended)

> [User: No, use tokenization]

❓ Question 2: Payment Methods
Which payment methods for v1?
- Credit/Debit cards only
- Cards + Apple Pay/Google Pay
- Cards + Bank transfers (ACH)
- Full suite (cards + wallets + buy-now-pay-later)

> [User: Cards + Apple Pay]

❓ Question 3: Billing Model
One-time payments or recurring?
- One-time only
- Recurring subscriptions (monthly/yearly)
- Both
- Usage-based billing

> [User: Recurring subscriptions]

❓ Question 4: Multi-Currency
Single currency or international?
- USD only
- Multiple currencies (which regions: US, EU, Asia?)
- Dynamic currency based on user location

> [User: USD only for v1]

❓ Question 5: Failed Payment Handling
How to handle failed recurring payments?
- Retry logic (how many attempts?)
- Dunning emails (automated reminders)
- Grace period before cancellation
- All of the above

> [User: Retry 3x, then 7-day grace period]

❓ Question 6: Out of Scope
What are you explicitly NOT doing in v1?
(Prevents scope creep!)

> [User: No refunds UI, no invoice PDFs, no tax calculations]
```

#### Example: UI/UX Feature

User input: "Add dark mode to the app"

```markdown
Got it! This is a UI/UX feature. A few quick questions...

❓ Question 1: Scope
Which parts get dark mode in v1?
- Entire app (all pages)
- Specific pages (dashboard, settings only)
- Just the main layout (header, sidebar)

> [User: Entire app]

❓ Question 2: User Preference
How should we remember the preference?
- localStorage (per-device)
- User account setting (synced across devices)
- Auto-detect system preference only (no toggle)

> [User: User account + system preference fallback]

❓ Question 3: Color Palette
How many themes?
- Light + Dark (2 themes)
- Light + Dark + High Contrast (3 themes)
- Custom theme builder (let users pick colors)

> [User: Just Light + Dark]

❓ Question 4: Transition
Switch between themes how?
- Instant (no animation)
- Smooth fade (300ms transition)
- Page reload required

> [User: Smooth fade]

❓ Question 5: Out of Scope
What's NOT included in v1?

> [User: No custom themes, no scheduled auto-switch, no theme previews]
```

#### Example: API/Backend Feature

User input: "Build a REST API for product catalog"

```markdown
This is an API/Backend feature. Let's define the scope...

❓ Question 1: API Design
REST or GraphQL?
- REST (standard HTTP methods)
- GraphQL (flexible queries)
- Both (REST for simple, GraphQL for complex)

> [User: REST]

❓ Question 2: Authentication
Who can access this API?
- Public (no auth required)
- Internal only (service-to-service)
- API keys (for third-party integrations)
- OAuth 2.0 (for user-specific data)

> [User: API keys for partners]

❓ Question 3: Rate Limiting
How many requests per minute?
- None (trusted users only)
- 60 req/min per API key
- 1000 req/min (high volume)
- Tiered (different limits per plan)

> [User: 1000 req/min, tiered later]

❓ Question 4: Data Format
Response format?
- JSON only
- JSON + XML
- JSON + CSV export

> [User: JSON only]

❓ Question 5: Endpoints (v1 scope)
Which operations for v1?
- CRUD (Create, Read, Update, Delete)
- Read-only (GET endpoints)
- Create + Read only (no updates/deletes yet)

> [User: Read-only for v1]

❓ Question 6: Out of Scope
What's deferred to v2?

> [User: No webhooks, no bulk import, no API versioning]
```

#### Example: Simple Feature (Skip Questions)

User input: "Add a loading spinner to the button"

```markdown
This is a simple, well-defined UI change. No need for questions!

Generating PRD directly...
```

**Question Selection Strategy** (Enhanced)
```markdown
Great! Let me ask a few questions to refine the scope...

Question 1: Target Users
Who will use this feature?
Options:
- All users (B2C)
- Enterprise/B2B users only
- Internal team/developers
- Specific user segment

> [User answers]

Question 2: Core Requirements
What are the 2-3 must-have capabilities for v1?
(Focus on minimum viable scope)

> [User lists core requirements]

Question 3: Out of Scope
What are you explicitly NOT doing in v1?
(This prevents scope creep - be specific!)

> [User defines boundaries]

Question 4: Integration Points (if applicable)
Does this integrate with existing systems?
- Which systems/services?
- Data flow between systems?

> [User answers]

Question 5: Success Criteria
How will you measure success?
Pick 1-2 key metrics.

> [User defines metrics]

Question 6: Technical Constraints (if applicable)
Any technical decisions already made?
- Tech stack?
- Performance targets?

> [User answers or "none"]
```

### Step 3: Generate PRD ID

Read PRD ID configuration from .claude/config.json or use defaults (PRD-XXX format)

Scan existing PRDs to find next number

Generate new ID: PRD-007

### Step 4: Generate PRD Content

Using answers from Step 2, generate comprehensive PRD with all required sections

### Step 5: Quick Self-Review

Automatically perform lightweight review and assign grade

### Step 6: Create File with New Naming Format

NEW NAMING FORMAT: PRD-{ID}-{feature-slug}.md

Examples:
- PRD-003-oauth2-integration.md
- PRD-007-dark-mode-support.md

Benefits:
- ID directly in filename (grep-able)
- No date confusion (Git tracks history)
- Cleaner, more professional

Save to: product/prds/01-draft/PRD-{ID}-{feature-slug}.md

### Step 7: Update WORK_PLAN.md

Add new entry to PRD pipeline table

### Step 8: Create Draft PR (Lock PRD↔PR Alignment)

**Purpose**: Reserve PR number immediately to guarantee PRD-025 ↔ PR #25 alignment

**Pre-flight checks**:
```bash
# 1. Ensure gh CLI is authenticated
if ! gh auth status >/dev/null 2>&1; then
  echo "⚠️  GitHub CLI not authenticated"
  echo "Run: gh auth login"
  echo ""
  echo "Skipping PR creation (will be created in /setup-prd)"
  SKIP_PR_CREATION=true
fi

# 2. Detect expected PR number
NEXT_PRD_NUMBER=$(basename "$PRD_FILE" | grep -oP 'PRD-\K\d+')
NEXT_PR_NUMBER=$(gh pr list --json number --limit 1 | jq -r '.[0].number + 1 // 1')

# 3. Check for desync
if [ "$NEXT_PRD_NUMBER" != "$NEXT_PR_NUMBER" ]; then
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "⚠️  PRD-PR ALIGNMENT WARNING"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""
  echo "Expected: PRD-$NEXT_PRD_NUMBER ↔ PR #$NEXT_PRD_NUMBER"
  echo "Actual:   PRD-$NEXT_PRD_NUMBER ↔ PR #$NEXT_PR_NUMBER"
  echo ""
  echo "Possible causes:"
  echo "  • Manual PR created (policy violation)"
  echo "  • PRD numbers were reset"
  echo "  • PR was deleted from GitHub"
  echo ""
  echo "Options:"
  echo "  1. Continue anyway (breaks alignment)"
  echo "  2. Skip PR creation (fix manually later)"
  echo "  3. Cancel and investigate"
  echo ""
  read -p "Choose (1-3): " choice

  case $choice in
    1) echo "⚠️  Proceeding with misalignment" ;;
    2) SKIP_PR_CREATION=true ;;
    3) echo "❌ Cancelled"; exit 1 ;;
  esac
fi
```

**Create feature branch**:
```bash
# Generate branch name from config
BRANCH_PREFIX=$(jq -r '.prd_workflow.branch_naming.prefix // "feat"' .claude/config.json)
FEATURE_SLUG=$(basename "$PRD_FILE" .md | sed 's/^PRD-[0-9]*-//' | tr '_' '-')
BRANCH_NAME="${BRANCH_PREFIX}/PRD-${NEXT_PRD_NUMBER}-${FEATURE_SLUG}"

# Create branch (don't checkout - will be done in /setup-prd)
git branch "$BRANCH_NAME" main

echo "✅ Branch created: $BRANCH_NAME"
```

**Create draft PR**:
```bash
if [ "$SKIP_PR_CREATION" != "true" ]; then
  echo "🔄 Creating draft PR to lock alignment..."

  # Extract feature name from PRD
  FEATURE_NAME=$(grep -m1 '^# ' "$PRD_FILE" | sed 's/^# //' | sed 's/^PRD-[0-9]*: //')

  # Create PR body
  PR_BODY="📋 **PRD**: \`$PRD_FILE\`

🚧 **Status**: Draft (not ready for development)

## PRD Details
- **Priority**: $(grep -m1 '^\*\*Priority\*\*:' "$PRD_FILE" | sed 's/.*: //' || echo 'Not set')
- **Estimated Effort**: $(grep -m1 '^\*\*Estimated Effort\*\*:' "$PRD_FILE" | sed 's/.*: //' || echo 'TBD')
- **Grade**: $(grep -m1 '^\*\*Grade\*\*:' "$PRD_FILE" | sed 's/.*: //' || echo 'Not reviewed')

## Next Steps
- [ ] PRD review and approval (\`/review-prd PRD-${NEXT_PRD_NUMBER}\`)
- [ ] Run \`/setup-prd PRD-${NEXT_PRD_NUMBER}\` to start development
- [ ] Worktree will be created automatically

---
⚠️ **Draft PR created by \`/create-prd\` to reserve PR number**

**Purpose**: Ensures PRD-${NEXT_PRD_NUMBER} ↔ PR #${NEXT_PR_NUMBER} alignment for the team

Do not close manually - will be auto-managed by PRD lifecycle

*Auto-created: $(date +%Y-%m-%d\ %H:%M:%S)*"

  # Create draft PR
  gh pr create --draft \
    --title "PRD-${NEXT_PRD_NUMBER}: ${FEATURE_NAME}" \
    --body "$PR_BODY" \
    --head "$BRANCH_NAME" \
    --base main 2>&1 | tee /tmp/pr_create_output.txt

  # Capture PR number
  PR_NUMBER=$(gh pr list --head "$BRANCH_NAME" --json number --jq '.[0].number' 2>/dev/null)

  if [ -n "$PR_NUMBER" ]; then
    echo "✅ Draft PR #$PR_NUMBER created"
    echo ""
    echo "🔗 PRD-${NEXT_PRD_NUMBER} ↔ PR #$PR_NUMBER locked"

    # Update PRD with PR link
    if ! grep -q "^\*\*PR\*\*:" "$PRD_FILE"; then
      # Add PR field after Status
      sed -i "/^\*\*Status\*\*:/a **PR**: #$PR_NUMBER" "$PRD_FILE"
    fi

    # Update WORK_PLAN with PR link
    if [ -f "product/WORK_PLAN.md" ]; then
      sed -i "s/PRD-${NEXT_PRD_NUMBER}/PRD-${NEXT_PRD_NUMBER} ([#$PR_NUMBER](https:\/\/github.com\/$(gh repo view --json nameWithOwner -q .nameWithOwner)\/pull\/$PR_NUMBER))/" product/WORK_PLAN.md
    fi
  else
    echo "⚠️  PR creation failed - check /tmp/pr_create_output.txt"
    echo "PR will be created in /setup-prd instead"
  fi
else
  echo "ℹ️  PR creation skipped - will be created in /setup-prd"
fi
```

**Success output**:
```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRD-PR ALIGNMENT LOCKED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 PRD: PRD-025-context-management.md
🌿 Branch: feat/PRD-025-context-management
🔗 PR: #25 (draft)

✅ PRD-025 ↔ PR #25 guaranteed

Next: /setup-prd PRD-025 (will reuse this PR)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 9: Provide Next Steps with Options

```markdown
PRD Created: PRD-007 - OAuth2 Integration

File: product/prds/01-draft/PRD-007-oauth2-integration.md
Status: Draft
Quick Review: B+ (Good scope)

Next Steps - Choose Your Path:

1. Approve Now (Quick path)
   Move to approved, start coding
   
2. Review Later (Recommended for parallel workflow)
   Keep in draft, create feature branch now
   Allows Main branch to stay free
   
3. Refine First
   Stay on Main, improve PRD now

What would you like to do? (1/2/3)
```

Handle user choice appropriately

## Configuration

Respects prd_workflow configuration in .claude/config.json

## Options

```bash
/create-prd
/create-prd "Add OAuth2 authentication"
/create-prd "Simple bug fix" --quick
/create-prd "New feature" --auto-approve
```

## Best Practices

- Ask clarifying questions - Don't assume!
- Define OUT OF SCOPE - Critical to prevent creep
- Stay in draft by default - Review on feature branch
- One feature per PRD
- Be specific
- Include success metrics

---

Plugin: claude-prd-workflow
Category: PRD Management
Version: 2.2.0
Requires: Git 2.0+
