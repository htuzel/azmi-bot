# Azmi Bot - AI-Powered Code Automation Router

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Issues](https://img.shields.io/github/issues/htuzel/router)](https://github.com/htuzel/router/issues)
[![GitHub Stars](https://img.shields.io/github/stars/htuzel/router)](https://github.com/htuzel/router/stargazers)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

> **Inspired by Azmi Mengü's vision** about building an agent that takes Jira tickets, writes code, runs tests, deploys, and closes the ticket automatically.

Azmi Bot is a GitHub Actions-based automation system that orchestrates AI agents (**Claude Opus 4.5** for planning & **Gemini 3 Pro** for coding/review) to transform Jira tickets into production code through a quality-gated, multi-repository workflow.

## ⚡ Quick Start

**New to Azmi Bot?** Start here:

- 🚀 **[Quick Start Guide (30 min)](QUICKSTART.md)** - Step-by-step setup for beginners
- 📖 **[Full Documentation](#-quick-start-1)** - Detailed guide below

---

## 🎯 What Does It Do?

```
Jira Ticket (+ Images) → Quality Gate (Router) → Target Repo → TSD → Implementation → PR → Review → Merge → Deploy
```

1. **Quality Gate**: Gemini 3 Pro scores Jira tickets (0-100) for clarity, completeness, and testability
   - ✨ **Vision Support**: AI can see and analyze attached images (mockups, screenshots, diagrams)
2. **Routing**: Passes high-quality tickets to the appropriate repository
3. **Planning**: Claude Opus 4.5 creates detailed implementation plans (TSD)
4. **Implementation**: Gemini 3 Pro implements code based on the plan
5. **Review**: Gemini 3 Pro reviews PRs and auto-fixes issues
6. **Automation**: PRs are auto-created, reviewed, and merged

## 🏗️ Architecture

### Three Repository Types

#### 1. Router Repository (This Repo)
- Receives Jira webhook triggers
- Scores ticket quality with **Gemini 3 Pro** (Structured Outputs)
- Updates Jira custom fields (score, verdict, comments)
- Routes approved tickets to target repositories via `repository_dispatch`

#### 2. Target Repositories (Your Service Repos)
- Receive `ai-coding-task` events
- Run 2-stage AI workflow:
  - **Claude Opus 4.5**: Creates Technical Specification Document (TSD) with full code examples
  - **Gemini 3 Pro**: Implements code based on TSD, opens PR
- Auto-merge after **Gemini 3 Pro** review

#### 3. Jira Integration
- Custom fields: `AI Quality Score`, `AI Quality Verdict`, `AI Target Repo`
- Automation rule: "Assignee is Azmi Bot" → triggers router
- Smart routing based on repo selection field

## 🖼️ Vision Support (Image Processing)

Azmi Bot can now **see and analyze images** attached to Jira tickets! This dramatically improves implementation accuracy for UI/UX tasks, bug reports with screenshots, and architectural diagrams.

### Supported Use Cases

#### 1. **UI/UX Implementation**
Attach mockups, wireframes, or design screenshots:
- ✅ AI reads exact colors, spacing, layouts from images
- ✅ Matches visual specifications in implementation
- ✅ Reduces back-and-forth for visual requirements

**Example:**
```
Title: "Implement new dashboard layout"
Description: "See attached mockup for the new dashboard design"
Attachments: dashboard-mockup.png
```
→ AI sees the mockup and implements matching the exact visual design

#### 2. **Bug Reports with Screenshots**
Attach error screenshots or visual bugs:
- ✅ AI identifies issues from error messages in images
- ✅ Understands visual bugs (layout issues, missing elements)
- ✅ Creates targeted fixes based on visual evidence

**Example:**
```
Title: "Login button not visible on mobile"
Description: "Button is cut off on iPhone 13"
Attachments: mobile-bug-screenshot.png
```
→ AI sees the screenshot and fixes the responsive layout issue

#### 3. **Architecture Diagrams**
Attach system architecture, flowcharts, or database schemas:
- ✅ Understands system relationships from diagrams
- ✅ Implements according to architectural patterns shown
- ✅ Follows data flow illustrated in images

**Example:**
```
Title: "Implement user authentication flow"
Description: "Follow the authentication flow in the attached diagram"
Attachments: auth-flow-diagram.png
```
→ AI understands the flow and implements each step correctly

#### 4. **Revision Requests with Images**
Comment with `#AZMI` + screenshot showing the issue:
- ✅ AI sees before/after comparisons in comment attachments
- ✅ Identifies visual differences or bugs from screenshots
- ✅ Implements fixes matching visual expectations
- ✅ Optimized images passed efficiently via dispatch payload
- ✅ Claude Code receives full visual context for accurate fixes

**How Revision Images Work:**
1. Add `#AZMI` comment to Jira issue
2. Attach screenshot(s) showing the problem
3. Router fetches & optimizes images (800x600 JPEG)
4. Passes optimized images in dispatch payload to target repo
5. Claude Code implements fix with full visual context

**Example:**
```
Comment: "#AZMI The spacing is wrong, see attached comparison"
Attachments: before-after-comparison.png
```
→ AI sees both images, identifies exact spacing issue, implements fix

**Example with Bootstrap classes:**
```
Comment: "#AZMI Use Bootstrap btn btn-outline-danger class instead of custom CSS"
Attachments: design-mockup.png
```
→ AI sees the design, uses exact Bootstrap class as requested (not custom CSS)

### Supported Image Formats

- ✅ **PNG, JPG, JPEG**: Standard image formats
- ✅ **GIF, WebP**: Modern image formats
- ✅ **PDF**: Automatically converted to PNG (first page only)
- ❌ **SVG**: Not supported (attach PNG export instead)

### Technical Details

**Image Processing Pipeline:**
```
Jira Attachment → Download → Optimize (800x600, JPEG quality 75) → Encode (Base64) → Embed in Prompt → AI Vision API
```

**Aggressive Image Optimization:**
- **Resolution**: Resized to max 800x600 (preserves aspect ratio)
- **Format**: PNG converted to JPEG (quality 75)
- **Metadata**: Stripped for smaller file size
- **Compression**: 90-95% size reduction (2-3MB → 50-100KB)
- **PDFs**: Converted to PNG (first page only, 200 DPI), then optimized
- **Large files** (> 10MB) are skipped with warning
- **Benefits**:
  - Fast AI processing (smaller tokens)
  - Fits in GitHub Actions dispatch payloads
  - Single Jira API fetch (efficient)
  - Still maintains quality for AI vision

**Quality Gate Scoring with Images:**
- If images clearly show WHAT needs to be done → **+20-30 bonus points**
- Visual specifications can substitute for detailed text descriptions
- AI analyzes images BEFORE evaluating ticket quality
- Optimized images analyzed by GPT-5 Vision for quality scoring

**Implementation with Images:**
- **Router**: Fetches and optimizes images once, passes in dispatch payload
- **Codex Planning**: Receives only image count notification (doesn't need full images for planning)
- **Claude Code Implementation**: Receives full optimized images for visual-accurate coding
- **Revision Flow**: Same optimization applied, images passed via dispatch payload

### How to Use

**Initial Ticket with Images:**
1. Create Jira ticket with clear title
2. Add description (can be brief if image is clear)
3. **Attach images** (mockups, screenshots, diagrams)
4. Assign to Azmi Bot
5. Select target repo

→ Router fetches images, AI evaluates with visual context, routes if approved

**Revision with Images:**
1. Comment on Jira ticket: `#AZMI fix the layout issue`
2. **Attach screenshot** showing the problem
3. AZMI evaluates revision request with visual context
4. If approved → Creates revision plan with image context → Implements fix

### Best Practices

**Do:**
- ✅ Use high-quality screenshots (clear, not blurry)
- ✅ Annotate images with arrows/highlights if helpful
- ✅ Include before/after comparisons for changes
- ✅ Attach multiple angles for complex UI
- ✅ Use design tool exports (Figma screenshots, etc.)

**Don't:**
- ❌ Attach huge files (> 10MB will be skipped)
- ❌ Use low-resolution images (hard for AI to read text)
- ❌ Attach unrelated images (confuses AI)
- ❌ Rely solely on images without any text description

### Limitations

- **Max 3 images for revisions** (configurable in scripts, prevents payload bloat)
- **Max file size: 10MB** per image before optimization (larger files skipped)
- **After optimization: ~50-100KB** per image (90-95% size reduction)
- **Total payload size: Safe for dispatch** (~300KB for 3 images)
- **PDF support: First page only** (multi-page PDFs truncated, then optimized)
- **No SVG support** (export to PNG instead)
- **Image quality: 800x600 JPEG** (sufficient for AI vision, optimized for speed)

### Troubleshooting

**Images not appearing in AI context:**
```bash
# Check workflow logs for:
"🖼️  Building structured input with X image(s)"
```

**Empty or corrupt images:**
```bash
# Check logs for:
"❌ Downloaded file is empty (0 bytes), skipping..."
```

**Large files skipped:**
```bash
# Check logs for:
"⚠️  File too large (XXX MB), skipping..."
```

**Image optimization working:**
```bash
# Check logs for successful optimization:
"✅ Optimized: 2.3MB → 78KB (-96%)"
"📸 Including optimized image context in payload (3 images)"
```

**"Argument list too long" errors:**
This was a common issue before optimization. If you see this error:
```bash
/usr/bin/jq: Argument list too long
```
- Ensure `fetch-jira-attachments.sh` has image optimization enabled
- Check that images are being resized to 800x600 max
- Verify PNG → JPEG conversion is working
- Should see 90-95% size reduction in logs

## 🚀 Quick Start

### Prerequisites

- Jira Cloud instance
- GitHub organization with repos
- Google AI API key (Gemini 3 Pro)
- Anthropic API key (Claude Opus 4.5)

### 1. Setup Router Repository

#### Clone and Configure

```bash
git clone https://github.com/YOUR_ORG/router.git
cd router
```

#### Add GitHub Secrets

Go to `Settings → Secrets and variables → Actions` and add:

| Secret | Description | How to Get |
|--------|-------------|------------|
| `GEMINI_API_KEY` | Google AI API key | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
| `JIRA_BASE` | Your Jira URL | `https://yourcompany.atlassian.net` |
| `JIRA_EMAIL` | Bot account email | Create a service account in Jira |
| `JIRA_API_TOKEN` | Jira API token | [id.atlassian.com → Security → API tokens](https://id.atlassian.com/manage-profile/security/api-tokens) |
| `DISPATCH_PAT` | GitHub Personal Access Token | See below ⬇️ |

#### How to Create DISPATCH_PAT

This token allows the router to trigger workflows in target repos:

1. Go to GitHub → **Settings** (your profile, not repo)
2. **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
3. **Generate new token**
   - Name: `azmi-bot-router`
   - Expiration: 90 days (or your preference)
   - Repository access: **Only select repositories**
     - Select all target repos (e.g., `org/win-room`, `org/api-service`)
   - Permissions:
     - ✅ **Contents**: Read and write
     - ✅ **Pull requests**: Read and write
     - ✅ **Metadata**: Read-only (automatic)
4. **Generate token** → Copy it (you won't see it again!)
5. Add as `DISPATCH_PAT` secret in router repo

### 2. Setup Jira

#### Create Custom Fields

In Jira → **Settings** → **Issues** → **Custom fields**:

1. **AI Quality Score**
   - Type: Number
   - Add to all screens where "AI Coding" issues appear

2. **AI Quality Verdict**
   - Type: Select List (single choice)
   - Options: `pass`, `fail`

3. **AI Target Repo**
   - Type: Select List (single choice)
   - Options:
     - `htuzel/win-room`
     - `htuzel/api-service`
     - `htuzel/mobile-backend`
     - *(add all your repos)*

**Find Custom Field IDs**: After creating fields, go to any issue → Edit → Inspect element on the field → find `customfield_XXXXX` in HTML. You'll need these IDs for the workflow.

#### Create Jira Bot User

1. Create a user: **azmi-bot@yourcompany.com**
2. Give it project access
3. Generate API token for this user (link above)

#### Setup Jira Automation

**Project Settings** → **Automation** → **Create rule**:

**Trigger:**
```
When: Work item transitioned
Condition applied: Assignee is "Azmi Bot"
```

**Action:** Send web request
```
URL: https://api.github.com/repos/YOUR_ORG/router/dispatches
Method: POST
Headers:
  Accept: application/vnd.github+json
  Authorization: Bearer YOUR_GITHUB_PAT
  Content-Type: application/json

Body (Custom data):
{
  "event_type": "ai-coding-request",
  "client_payload": {
    "issue_key": "{{issue.key}}",
    "title": "{{issue.summary}}",
    "description": "{{issue.description}}",
    "target_repo": "{{issue.AI Target Repo}}",
    "url": "{{issue.url}}"
  }
}
```

Replace `YOUR_ORG/router` and `YOUR_GITHUB_PAT` with your values.

**Note:** You can use the same PAT as `DISPATCH_PAT` or create a separate one just for Jira automation.

### 3. Update Router Workflow

Edit `.github/workflows/router.yml`:

**Line 131-136**: Update custom field IDs
```yaml
# Change customfield_12345 and 12346 to your actual field IDs
curl -sS -X PUT "$JIRA_BASE/rest/api/3/issue/$ISSUE" \
  -H "Authorization: Basic $AUTH" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  --data '{"fields":{"customfield_12345":'"$SCORE"',"customfield_12346":"'"$VERDICT"'"}}'
```

To find your field IDs:
```bash
curl -H "Authorization: Basic $(echo -n 'your-email:api-token' | base64)" \
  https://yourcompany.atlassian.net/rest/api/3/field | jq '.[] | select(.name | contains("AI"))'
```

### 4. Setup Target Repositories

For each target repo (e.g., `win-room`, `api-service`):

#### Add Secrets

`Settings → Secrets and variables → Actions`:

| Secret | Value | Description |
|--------|-------|-------------|
| `GEMINI_API_KEY` | Same as router | For Gemini 3 Pro coding and review |
| `ANTHROPIC_API_KEY` | From [console.anthropic.com](https://console.anthropic.com) | For Claude Opus 4.5 planning |
| `JIRA_BASE` | Same as router | `https://yourcompany.atlassian.net` |
| `JIRA_EMAIL` | Same as router | Bot account email |
| `JIRA_API_TOKEN` | Same as router | Jira API token |
| `GH_PAT` | GitHub Personal Access Token | For creating PRs and triggering workflows (see below) |

#### How to Create GH_PAT for Target Repos

This token is needed because `secrets.GITHUB_TOKEN` doesn't trigger other workflows. Create it same way as `DISPATCH_PAT`:

1. Go to GitHub → **Settings** (your profile)
2. **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
3. **Generate new token**
   - Name: `azmi-bot-target-repo`
   - Repository access: **Only select repositories** → Select this target repo
   - Permissions:
     - ✅ **Contents**: Read and write
     - ✅ **Pull requests**: Read and write
     - ✅ **Workflows**: Read and write (to trigger other workflows)
4. **Generate token** → Add as `GH_PAT` secret in target repo

**Note:** You can reuse the same `DISPATCH_PAT` token as `GH_PAT` if it has access to the target repo.

#### Set Permissions

`Settings → Actions → General → Workflow permissions`:
- ✅ **Read and write permissions**
- ✅ **Allow GitHub Actions to create and approve pull requests**

#### Add Workflows

Create these files:

**`.github/workflows/ai-coding.yml`**
```yaml
name: AI Coding

on:
  repository_dispatch:
    types: [ai-coding-task]

permissions:
  contents: write
  pull-requests: write
  issues: write

env:
  BASE_BRANCH: main

jobs:
  codex-plan:
    runs-on: ubuntu-latest
    env:
      JIRA_BASE: ${{ secrets.JIRA_BASE }}
      JIRA_EMAIL: ${{ secrets.JIRA_EMAIL }}
      JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
    steps:
      - uses: actions/checkout@v4

      - name: Extract issue_key from payload
        id: payload
        run: |
          PAYLOAD='${{ toJSON(github.event.client_payload) }}'
          echo "issue_key=$(jq -r '.issue_key' <<< "$PAYLOAD")" >> $GITHUB_OUTPUT

      - name: Fetch issue from Jira
        id: jira
        shell: bash
        run: |
          set -euo pipefail

          ISSUE=$(jq -r '.issue_key' <<< '${{ toJSON(github.event.client_payload) }}')

          if [ -z "$ISSUE" ] || [ "$ISSUE" = "null" ]; then
            echo "ERROR: Issue key is missing from payload"
            exit 1
          fi

          echo "Fetching issue: $ISSUE from Jira..."
          echo "JIRA_BASE: $JIRA_BASE"
          echo "JIRA_EMAIL: $JIRA_EMAIL"
          echo "Full URL: $JIRA_BASE/rest/api/3/issue/$ISSUE?expand=renderedFields"

          AUTH=$(printf "%s:%s" "$JIRA_EMAIL" "$JIRA_API_TOKEN" | base64 | tr -d '\n')

          mkdir -p .artifacts
          curl -sS -H "Authorization: Basic $AUTH" \
            -H "Accept: application/json" \
            "$JIRA_BASE/rest/api/3/issue/$ISSUE?expand=renderedFields" \
            -o .artifacts/jira-issue.json

          echo "Issue fetched successfully"

          TITLE=$(jq -r '.fields.summary' .artifacts/jira-issue.json)
          # Use renderedFields.description (HTML string) instead of fields.description (ADF JSON)
          DESCRIPTION=$(jq -r '.renderedFields.description // "No description"' .artifacts/jira-issue.json)

          echo "Title: $TITLE"
          echo "Description preview: ${DESCRIPTION:0:100}..."

          # Multi-line output requires EOF delimiter in GitHub Actions
          echo "title=$TITLE" >> $GITHUB_OUTPUT
          echo "description<<EOF" >> $GITHUB_OUTPUT
          echo "$DESCRIPTION" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Codex - produce plan (TSD) as text
        id: codex
        uses: openai/codex-action@v1
        with:
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
          prompt: |
            You are a senior  software engineer and a senior planner. Given the Jira task, output a clear, concise implementation plan as Markdown.
            The plan is for Claude Code to implement.

            Task ID: ${{ steps.payload.outputs.issue_key }}
            Title: ${{ steps.jira.outputs.title }}
            Description:
            ${{ steps.jira.outputs.description }}

            Output sections:
            1) goal
            2) scope - list of files or modules to touch
            3) acceptance_criteria - 3 to 6 bullet points
            4) test_plan - unit and integration bullets
            5) risks_and_mitigations - brief list
            6) runbook - how to run & rollback
            7) code - code to be implemented
            8) Output should be a detailed tsd.md file.

            Constraints:
            - Keep it specific to this repo.
            - Do not write any code here. Only the plan.
          effort: "high"

      - name: Save plan to artifact
        run: |
          mkdir -p .artifacts
          cat > .artifacts/PLAN.md << 'EOF'
          ${{ steps.codex.outputs.final-message }}
          EOF
      - name: Upload plan artifact
        uses: actions/upload-artifact@v4
        with:
          name: ai-plan
          path: .artifacts/PLAN.md

  claude-implement:
    runs-on: ubuntu-latest
    needs: codex-plan
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          name: ai-plan
          path: .artifacts

      - name: Read plan from artifact
        id: read-plan
        run: |
          PLAN=$(cat .artifacts/PLAN.md)
          echo "plan<<EOF" >> $GITHUB_OUTPUT
          echo "$PLAN" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Claude Code - review plan and implement, then open PR
        uses: anthropics/claude-code-action@v1
        env:
          GH_TOKEN: ${{ secrets.GH_PAT }}
          GITHUB_TOKEN: ${{ secrets.GH_PAT }}
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GH_PAT }}
          show_full_output: true
          prompt: |
            You are Claude Code working on ${{ github.repository }}.

            CONTEXT:
            - Repository: Win Room v2.0 - Next.js 14 + PostgreSQL + Socket.IO sales platform
            - Architecture documented in TSD.md
            - Task: ${{ github.event.client_payload.issue_key || 'Task' }}

            IMPLEMENTATION PLAN FROM CODEX:
            ---
            ${{ steps.read-plan.outputs.plan }}
            ---

            YOUR WORKFLOW:

            1. VALIDATE THE PLAN:
               - Read the plan carefully
               - Check if requirements are already implemented in the codebase
               - If already implemented: document what exists and suggest only verification/testing steps
               - If gaps exist: refine the plan with specific file paths and implementation details
               - Create your "Final Implementation Plan" (can be same as Codex plan if it's good)

            2. IMPLEMENT THE CHANGES:
               - Follow the Final Implementation Plan step by step
               - Make minimal, safe changes
               - Ensure code follows existing patterns in the repo
               - Add appropriate error handling
               - Update types/interfaces as needed

            3. ADD TESTS:
               - Follow the test_plan from the implementation plan
               - Add unit tests for new functions/components
               - Add integration tests for API endpoints
               - Ensure tests pass before creating PR

            4. COMMIT AND PUSH:
               - Create feature branch: `git checkout -b feature/${{ github.event.client_payload.issue_key || 'ai-task' }}`
               - Stage changes: `git add -A`
               - Commit with message: `git commit -m "Implement ${{ github.event.client_payload.issue_key || 'task' }}"`
               - Push to remote: `git push -u origin feature/${{ github.event.client_payload.issue_key || 'ai-task' }}`

            5. CREATE PULL REQUEST:
               - Use GitHub CLI to create PR: `gh pr create --title "..." --body "..." --base ${{ env.BASE_BRANCH }}`
               - PR Title: "AI Coding - ${{ github.event.client_payload.issue_key || 'Task' }}"
               - PR Body must include:
                 * **Final Implementation Plan** (your validated/refined plan)
                 * **Changes Made** (bullet list of files changed and why)
                 * **Testing Done** (what tests were added/run)
                 * **Notes** (any important considerations for reviewers)
               - CRITICAL: You MUST create the PR. Use gh pr create command.
               - Do not ask for approval. Execute the gh pr create command automatically.

            IMPORTANT RULES:
            - If the feature is already 90%+ implemented, create a PR that just adds tests/documentation
            - Always create a PR even if changes are minimal - the PR is the deliverable
            - Include clear commit messages
            - Make sure all file paths in commits are correct
            - Run any relevant build/lint commands before pushing

            CRITICAL: You are running in CI/CD environment. DO NOT ASK FOR APPROVAL.
            Automatically proceed with all file operations, git commands, and implementations.
            No human is available to approve - execute all changes autonomously.

            BEGIN IMPLEMENTATION NOW.
          claude_args: |
            --dangerously-skip-permissions
            --max-turns 100
            --model claude-sonnet-4-5-20250929
```

**Note:** See "Add Secrets" section above for `GH_PAT` setup instructions.

**`.github/workflows/codex-review.yml`**
```yaml
name: Codex Review and Merge

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: write
  pull-requests: write

jobs:
  codex-review:
    runs-on: ubuntu-latest
    env:
      JIRA_BASE: ${{ secrets.JIRA_BASE }}
      JIRA_EMAIL: ${{ secrets.JIRA_EMAIL }}
      JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
    steps:
      - uses: actions/checkout@v4
        with:
          ref: refs/pull/${{ github.event.pull_request.number }}/merge

      - name: Fetch PR heads
        run: |
          git fetch --no-tags origin \
            ${{ github.event.pull_request.base.ref }} \
            +refs/pull/${{ github.event.pull_request.number }}/head

      - name: Extract Jira issue key from PR
        id: jira-key
        run: |
          # Try to extract from PR title first (format: "AI Coding - GET-1234")
          TITLE="${{ github.event.pull_request.title }}"
          ISSUE_KEY=$(echo "$TITLE" | grep -oE '[A-Z]+-[0-9]+' | head -1 || echo "")

          if [ -z "$ISSUE_KEY" ]; then
            # Try from PR body
            ISSUE_KEY=$(echo "${{ github.event.pull_request.body }}" | grep -oE '[A-Z]+-[0-9]+' | head -1 || echo "")
          fi

          echo "issue_key=$ISSUE_KEY" >> $GITHUB_OUTPUT
          echo "Found Jira issue: $ISSUE_KEY"

      - name: Codex - review PR and, if needed, output a unified diff patch
        id: codex
        uses: openai/codex-action@v1
        with:
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
          prompt: |
            You are a senior reviewer. Review ONLY the changes in this PR for ${{ github.repository }}.

            1) If there are blocking issues, output a unified diff patch that fixes them.
               The patch must be between markers:
               ```patch
               *** BEGIN PATCH
               <unified diff here>
               *** END PATCH
               ```
            2) If there are no blocking issues, output a short approval message.

            Also verify that the PR body contains a "Final TSD" and that acceptance criteria are met by tests.
          effort: "high"

      - name: Try to extract patch
        id: patch
        run: |
          set -e
          echo "${{ steps.codex.outputs.final-message }}" > .codex_msg.txt
          if grep -q "\*\*\* BEGIN PATCH" .codex_msg.txt; then
            awk '/\*\*\* BEGIN PATCH/{flag=1; next} /\*\*\* END PATCH/{flag=0} flag' .codex_msg.txt > codex.patch
            echo "has_patch=true" >> $GITHUB_OUTPUT
          else
            echo "has_patch=false" >> $GITHUB_OUTPUT
          fi

      - name: Apply patch if exists
        if: steps.patch.outputs.has_patch == 'true'
        run: |
          git config user.name "codex-bot"
          git config user.email "codex-bot@users.noreply.github.com"
          git checkout ${{ github.event.pull_request.head.ref }}
          git apply --whitespace=fix codex.patch
          git commit -am "Codex fix patch"
          git push origin HEAD

      - name: Claude Code - secondary review
        id: claude
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          prompt: |
            You are a senior code reviewer performing a secondary review of this PR.
            Codex has already done an initial review. Your role is to provide additional insights.

            Review this PR for ${{ github.repository }} against the TSD.md architecture.

            Focus on:
            1. **Security**: SQL injection, XSS, authentication bypass, sensitive data exposure
            2. **Performance**: N+1 queries, unnecessary loops, memory leaks, inefficient algorithms
            3. **Correctness**: Logic errors, edge cases, null handling, race conditions
            4. **Code Quality**: Consistent patterns, proper error handling, code duplication
            5. **Testing**: Coverage of acceptance criteria, edge case tests, integration tests
            6. **Architecture**: Follows Win Room patterns (TSD.md), proper separation of concerns

            Output format:
            - If you find issues: List them with severity (CRITICAL/HIGH/MEDIUM/LOW)
            - If all looks good: "LGTM - Claude Code secondary review passed"

            Keep your review concise but specific. Reference file paths and line numbers.

      - name: Post Claude review as comment
        uses: actions/github-script@v7
        with:
          script: |
            const claudeReview = `${{ steps.claude.outputs.response }}`;
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.payload.pull_request.number,
              body: `## 🤖 Claude Code Secondary Review\n\n${claudeReview}`
            });

      - name: Approve PR if no blockers
        if: steps.patch.outputs.has_patch == 'false'
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.pulls.createReview({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.payload.pull_request.number,
              event: "APPROVE",
              body: "LGTM by Codex."
            });

      - name: Post review summary to Jira
        if: steps.jira-key.outputs.issue_key != ''
        run: |
          ISSUE_KEY="${{ steps.jira-key.outputs.issue_key }}"

          if [ -z "$ISSUE_KEY" ]; then
            echo "No Jira issue found, skipping Jira update"
            exit 0
          fi

          AUTH=$(printf "%s:%s" "$JIRA_EMAIL" "$JIRA_API_TOKEN" | base64 | tr -d '\n')
          PR_URL="${{ github.event.pull_request.html_url }}"

          # Determine review status
          if [ "${{ steps.patch.outputs.has_patch }}" = "true" ]; then
            REVIEW_STATUS="⚠️ Issues Found - Auto-fixed by Codex"
          else
            REVIEW_STATUS="✅ Approved"
          fi

          # Create ADF (Atlassian Document Format) comment
          COMMENT_BODY=$(jq -n \
            --arg status "$REVIEW_STATUS" \
            --arg pr_url "$PR_URL" \
            --arg codex_msg "${{ steps.codex.outputs.final-message }}" \
            '{
              body: {
                type: "doc",
                version: 1,
                content: [
                  {
                    type: "heading",
                    attrs: { level: 2 },
                    content: [{ type: "text", text: "🤖 AI Code Review Completed" }]
                  },
                  {
                    type: "paragraph",
                    content: [
                      { type: "text", text: "Status: ", marks: [{ type: "strong" }] },
                      { type: "text", text: $status }
                    ]
                  },
                  {
                    type: "paragraph",
                    content: [
                      { type: "text", text: "Pull Request: ", marks: [{ type: "strong" }] },
                      { type: "text", text: $pr_url, marks: [{ type: "link", attrs: { href: $pr_url } }] }
                    ]
                  },
                  {
                    type: "heading",
                    attrs: { level: 3 },
                    content: [{ type: "text", text: "Codex Review:" }]
                  },
                  {
                    type: "codeBlock",
                    content: [{ type: "text", text: $codex_msg }]
                  },
                  {
                    type: "paragraph",
                    content: [
                      { type: "text", text: "Claude Code secondary review also completed.", marks: [{ type: "em" }] }
                    ]
                  }
                ]
              }
            }')

          curl -sS -X POST "$JIRA_BASE/rest/api/3/issue/$ISSUE_KEY/comment" \
            -H "Authorization: Basic $AUTH" \
            -H "Accept: application/json" \
            -H "Content-Type: application/json" \
            --data "$COMMENT_BODY"

          echo "Posted review summary to Jira issue $ISSUE_KEY"

      - name: Move Jira issue to Review status
        if: steps.jira-key.outputs.issue_key != ''
        run: |
          ISSUE_KEY="${{ steps.jira-key.outputs.issue_key }}"

          if [ -z "$ISSUE_KEY" ]; then
            echo "No Jira issue found, skipping status transition"
            exit 0
          fi

          AUTH=$(printf "%s:%s" "$JIRA_EMAIL" "$JIRA_API_TOKEN" | base64 | tr -d '\n')

          # Get available transitions for this issue
          TRANSITIONS=$(curl -sS -X GET "$JIRA_BASE/rest/api/3/issue/$ISSUE_KEY/transitions" \
            -H "Authorization: Basic $AUTH" \
            -H "Accept: application/json")

          echo "Available transitions:"
          echo "$TRANSITIONS" | jq '.transitions[] | {id: .id, name: .name}'

          # Find transition ID for "Review" status (try common names)
          TRANSITION_ID=$(echo "$TRANSITIONS" | jq -r '.transitions[] | select(.name | test("Review|In Review|Code Review"; "i")) | .id' | head -1)

          if [ -z "$TRANSITION_ID" ] || [ "$TRANSITION_ID" = "null" ]; then
            echo "⚠️ No 'Review' transition found for issue $ISSUE_KEY"
            echo "Available transitions are listed above. Please configure Jira workflow."
            exit 0
          fi

          # Perform transition
          TRANSITION_BODY=$(jq -n --arg id "$TRANSITION_ID" '{transition: {id: $id}}')

          curl -sS -X POST "$JIRA_BASE/rest/api/3/issue/$ISSUE_KEY/transitions" \
            -H "Authorization: Basic $AUTH" \
            -H "Accept: application/json" \
            -H "Content-Type: application/json" \
            --data "$TRANSITION_BODY"

          echo "✅ Moved Jira issue $ISSUE_KEY to Review status (transition ID: $TRANSITION_ID)"

      - name: Merge PR if approved and checks pass
        uses: actions/github-script@v7
        with:
          script: |
            // try merge; if not ready, do nothing
            try {
              await github.rest.pulls.merge({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: context.payload.pull_request.number,
                merge_method: "squash"
              });
              core.info("Merged successfully.");
            } catch (e) {
              core.info("Merge not possible yet. Waiting for checks or approvals.");
            }
```

#### Add CLAUDE.md (Optional but Recommended)

Create `CLAUDE.md` in the root of each target repo:

```markdown
# Development Guidelines for Azmi Bot

## Code Standards
- Follow existing patterns in the repository
- Use TypeScript with strict mode
- Write self-documenting code with clear variable names
- Add JSDoc comments for complex functions

## Testing Requirements
- Unit tests for all new functions (>= 80% coverage)
- Integration tests for API endpoints
- Use existing test framework (Jest/Vitest)

## PR Requirements
- Include "Final TSD" section in PR body
- List all changed files with brief explanations
- Add "Testing Done" section
- Keep PRs focused and minimal

## Architecture
- Next.js App Router structure
- API routes under `app/api/`
- Components in `components/`
- Shared utilities in `lib/`
- Database operations in `lib/db/`

## Never Do
- Break existing API contracts
- Remove existing tests
- Deploy without testing
- Commit secrets or API keys
```

## 📊 End-to-End Flow

```
1. Create Jira Ticket
   └─ Assign to "Azmi Bot"
   └─ Select "AI Target Repo"

2. Jira Automation Triggers
   └─ Sends webhook to Router repo

3. Router Repository
   ├─ Gemini 3 Pro scores ticket (0-100)
   ├─ Updates Jira fields
   ├─ Posts suggestions as comment
   └─ If score >= 80:
       └─ Sends repository_dispatch to target repo

4. Target Repository
   ├─ Claude Opus 4.5 creates TSD (Technical Spec + Code Examples)
   ├─ Gemini 3 Pro implements code from TSD
   ├─ Gemini 3 Pro opens PR
   └─ Triggers Gemini Review workflow

5. Code Review Workflow
   ├─ Gemini 3 Pro reviews PR
   ├─ If issues: Creates patch → Applies → Commits
   ├─ Posts to Jira with PR link
   ├─ Moves Jira to "Review" status
   └─ If approved: Auto-merges PR

6. Deploy (Your CI/CD)
   └─ After merge, your deploy pipeline runs
```

## 🔧 Troubleshooting

### Router not triggering

**Check:**
- Jira automation is active
- GitHub PAT has correct permissions
- Webhook payload matches expected format

**Debug:**
```bash
# Manually trigger router
curl -X POST \
  -H "Authorization: Bearer YOUR_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/YOUR_ORG/router/dispatches \
  -d '{"event_type":"ai-coding-request","client_payload":{"issue_key":"TEST-1","title":"Test","description":"Test description","target_repo":"YOUR_ORG/YOUR_REPO"}}'
```

### Target repo not receiving dispatch

**Check:**
- `DISPATCH_PAT` has access to target repo
- Target repo has `repository_dispatch` trigger for `ai-coding-task`
- Jira score >= 80 (check router logs)

### PR not created

**Check:**
- `GH_PAT` secret exists in target repo
- `GH_TOKEN` environment variable set in Claude Code step
- Workflow permissions: "Read and write" + "Allow PR creation"

### PR not auto-merging

**Check:**
- Branch protection rules (may require approvals)
- CI checks must pass first
- Codex review completed without blockers

## 🔄 Post-Merge Revisions with #AZMI

After code is merged and deployed to staging, you might discover bugs or need adjustments. Instead of creating a new Jira ticket, you can **request revisions directly in the original ticket** by commenting with `#AZMI`.

### How It Works

```
Merged PR → Staging Testing → Find Issue → Comment "#AZMI fix XYZ" → AZMI Implements Fix → New PR
```

1. **Test in staging** and find an issue
2. **Comment on the Jira ticket** with `#AZMI` followed by revision request
3. **Router evaluates** the revision request (quality check)
4. **If approved**: Task moves to "In Progress", AZMI creates revision plan
5. **Claude Code implements** the fix with context from previous PR
6. **New PR is created** for review and merge

### Setup Revision Flow

#### 1. Add Revision Router Workflow

The router repo already has `revision-router.yml` which handles revision requests.

#### 2. Add Revision Workflow to Target Repos

Copy `target_repo/ai-revision.yml` to your target repository's `.github/workflows/` folder.

This workflow:
- Receives `ai-revision-task` events
- Fetches previous PR changes for context
- Creates revision plan with Codex
- Implements fixes with Claude Code
- Opens new PR with revision

#### 3. Setup Jira Automation for #AZMI Comments

**Project Settings** → **Automation** → **Create rule**:

**Trigger:**
```
When: Comment is added
Condition: Comment contains "#AZMI" (case insensitive)
```

**Action:** Send web request
```
URL: https://api.github.com/repos/YOUR_ORG/router/dispatches
Method: POST
Headers:
  Accept: application/vnd.github+json
  Authorization: Bearer YOUR_GITHUB_PAT
  Content-Type: application/json

Body (Custom data):
{
  "event_type": "ai-revision-request",
  "client_payload": {
    "issue_key": "{{issue.key}}",
    "comment_body": "{{comment.body}}",
    "target_repo": "{{issue.AI Target Repo}}",
    "previous_pr_url": ""
  }
}
```

**Note:** `previous_pr_url` can be extracted from Jira comments if PRs post their links. Otherwise, AZMI will search through Jira comments to find the PR.

#### 4. How to Use

After a PR is merged and you test in staging:

**Example 1: Bug Fix**
```
#AZMI The login button on mobile devices is not working.
Error in console: "Cannot read property 'email' of undefined"
Fix the null check in src/components/LoginForm.tsx line 45
```

**Example 2: UI Adjustment**
```
#AZMI The dashboard cards need more spacing.
Increase the gap between cards from 16px to 24px in src/app/dashboard/page.tsx
```

**Example 3: Performance Issue**
```
#AZMI The user list is loading slowly (5+ seconds)
Add pagination to the /api/users endpoint, limit to 50 users per page
```

### Quality Criteria for Revision Requests

AZMI evaluates revision comments on:

1. **Clarity (30 points)**: What needs to be changed?
2. **Specificity (30 points)**: File paths, function names, error messages?
3. **Actionability (40 points)**: Can AI implement this directly?

**Passing score**: 70+

**Good revision request:**
```
#AZMI Fix the authentication bug in production:
- File: src/lib/auth.ts line 67
- Issue: Token expiry check is using wrong timezone (UTC vs local)
- Fix: Use UTC consistently for token expiry comparison
- Test: Verify users can stay logged in across timezone boundaries
```

**Poor revision request:**
```
#AZMI fix the bug
```

### Revision Flow Diagram

```
1. Comment on Jira: "#AZMI fix the login bug..."
   └─ Jira automation triggers

2. Router Repository (revision-router.yml)
   ├─ AI quality check on revision request (0-100)
   ├─ Posts assessment to Jira
   ├─ Moves task to "In Progress" if approved
   └─ Dispatches to target repo with previous PR context

3. Target Repository (ai-revision.yml)
   ├─ Fetches previous PR changes
   ├─ Codex creates revision plan
   ├─ Claude Code implements fixes
   └─ Opens new PR: "AZMI Revision - TASK-123"

4. Review & Merge (codex-review.yml)
   ├─ Same review process as initial implementation
   ├─ Codex reviews, Claude Code validates
   └─ Auto-merge if approved
```

### Benefits of Revision Flow

- **Fast iterations**: No need to create new tickets
- **Context preservation**: AZMI sees previous implementation
- **Quality gated**: Poor revision requests are rejected with suggestions
- **Audit trail**: All revisions linked to original task in Jira
- **Minimal overhead**: Just comment "#AZMI" + description

## 🎨 Customization

### Change Quality Threshold

Edit `router/.github/workflows/router.yml` line 228:
```yaml
if: steps.jira.outputs.verdict == 'pass'  # Change logic here
```

### Add More Repos

1. Add repo option in Jira "AI Target Repo" field
2. Give `DISPATCH_PAT` access to new repo
3. Add workflows to new repo

### Customize AI Prompts

- **Quality scoring**: `router.yml` line 81-91
- **TSD generation**: `ai-coding.yml` line 77-98
- **Implementation**: `ai-coding.yml` line 217-266
- **Code review**: `codex-review.yml` line 49-61

### Change Review Behavior

Edit `codex-review.yml`:
- Auto-fix: Keep patch application logic
- Review only: Remove patch application, keep comment
- Block merge: Add `exit 1` if issues found

## 📚 Documentation Index

### Getting Started
- [Quick Start Guide (30 min)](QUICKSTART.md) - Beginner-friendly setup guide
- [Setup Checklist](SETUP_CHECKLIST.md) - Track your configuration progress
- [Full README](#-quick-start-1) - Comprehensive documentation (this file)

### Community & Contribution
- [Contributing Guidelines](CONTRIBUTING.md) - How to contribute
- [Code of Conduct](CODE_OF_CONDUCT.md) - Community standards
- [Security Policy](SECURITY.md) - Security best practices
- [Changelog](CHANGELOG.md) - Version history

### Technical Articles
- [ARTICLE.md](ARTICLE.md) - Building Azmi Bot: The full story and technical deep dive

### External References
- [OpenAI Codex Action](https://github.com/openai/codex-action)
- [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- [Jira REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Jira Automation](https://support.atlassian.com/cloud-automation/docs/jira-automation-actions/)

## 🤝 Contributing

This is an open-source implementation inspired by Azmi Mengü's vision of autonomous code agents. Feel free to:

- Fork and adapt for your organization
- Submit improvements via PR
- Share your experience

## 📝 License

MIT License - See LICENSE file

## 🙏 Credits

- Inspired by Azmi Mengü's vision of AI agents that code, test, and deploy
- Planning powered by [Claude Opus 4.5](https://www.anthropic.com/claude)
- Coding & Review powered by [Google Gemini 3 Pro](https://ai.google.dev/)
- Orchestrated with [GitHub Actions](https://github.com/features/actions)

---

**Star ⭐ if this helped you!** | **[Report Issues](https://github.com/htuzel/router/issues)** | **[Contribute](CONTRIBUTING.md)**
