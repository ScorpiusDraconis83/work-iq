# JSON Development Workflow

This document provides step-by-step instructions for developing M365 Copilot agents using JSON manifest files.

## Prerequisites

- Project must be scaffolded first (use scaffolding workflow if needed)
- Project contains `m365agents.yml` at the root
- Project uses JSON manifest files (`.json`)

---

## ✅ APP NAME & DESCRIPTION REQUIREMENT ✅

When developing an agent, you MUST ALWAYS update the app name and description in `manifest.json` to something **meaningful and descriptive** that reflects the agent's purpose. Never leave default/placeholder names like "My Agent" or generic descriptions.

**NEVER use "(local)" suffix in app names.** Always remove any "(local)" suffix from the app name.

---

## 🚨 CRITICAL VALIDATION AND DEPLOYMENT RULE 🚨

After editing an agent, validate it with `wiqd agent validate` before returning to the user.
Provision only when the user explicitly asks to deploy, provision, test, share, or publish.

**You must NEVER:**
- Skip validation after changing manifests, instructions, capabilities, or plugins
- Provision merely because files changed
- Deploy when validation found errors — not even "to test" or "to demonstrate"
- Deploy "to show the user what happens" when there are errors — just report the errors
- Run `wiqd agent provision` "for educational purposes" to demonstrate failure — errors = STOP, not a teaching moment

---

---

## 💬 CONVERSATION STARTERS REQUIREMENT 💬

Every agent needs meaningful conversation starters that help users understand what the agent can do. Review the agent's capabilities and add/update conversation starters that showcase the agent's primary functions. Never leave an agent without conversation starters.

---

## 📝 ALWAYS UPDATE INSTRUCTIONS & STARTERS AFTER CHANGES — MANDATORY 📝

**This is NOT optional.** Adding a capability without updating instructions is incomplete work.

When you add, remove, or modify ANY capability or plugin, you MUST complete ALL of these steps
before validation and any user-requested deployment:

1. **Update `instructions`** — Add a section describing what the new capability/plugin enables. For removals, delete all references to the removed capability.
2. **Add conversation starters** — Add at least 1 new conversation starter per added capability or plugin. Each starter should demonstrate the new functionality.
3. **Remove stale starters** — Delete conversation starters that reference removed capabilities.
4. **Update description** in `manifest.json` if the agent's purpose has expanded.
5. **Review existing instructions** for stale references to removed capabilities.

**This applies to EVERY edit operation:** adding capabilities, removing capabilities, adding API plugins, adding MCP servers, modifying scoping, or any other manifest change.

---

## Instructions

### Step 1: Understand the Requirements

**Action:** Gather and analyze the agent requirements:
- Identify the agent's primary purpose and target users
- Determine required data sources (M365 services, external APIs)
- List necessary actions the agent must perform
- Identify security and compliance requirements

### Step 2: Design the Agent Architecture

**Action:** Create a comprehensive architectural design:
- Select deployment model (personal or shared)
- Choose appropriate M365 capabilities with scoping
- Design API plugin integrations if needed
- Plan authentication and authorization strategy
- Design conversation flow and instructions

### Step 3: Edit JSON Manifest Files

**⚠️ PRE-EDIT CHECK — Before making ANY edits, do ALL of these:**

1. **Check for malformed JSON**: Read `declarativeAgent.json` and verify it parses correctly. If it has syntax errors (missing commas, unclosed brackets, trailing commas, etc.):
   - **STOP** — do NOT proceed with your edit
   - **INFORM** the user: list every syntax issue with line numbers
   - **ASK** the user if you should fix the syntax errors first
   - Only after the user confirms, fix with surgical edits, then re-read the file
   - Then continue with the user's original request as a separate step

2. **Check the schema version**: Read the `"version"` field in `declarativeAgent.json` (e.g., `"v1.4"`, `"v1.6"`). For EVERY feature you plan to add, verify it exists in that version using the [feature matrix](schema.md). If a requested feature requires a newer version → **STOP. Tell the user.** Offer to upgrade the version first.

3. **Run a proactive instruction review** (if the edit touches instructions or capabilities): Before modifying instructions or adding/removing capabilities, run [Instruction Review](instruction-review.md) **Phase 1 (Inventory)**, **Phase 2 (Comprehension Check)**, and **Phase 3 (Diagnose)** against the current instructions. This catches existing problems before you add to them. For Phase 2, use the brief confirmation shortcut ("I see this agent is designed to [purpose]…") since this is a proactive check. If the review finds high-severity issues (C1, C3, C11, D1-D8), inform the user and offer to fix them as part of the current edit.

**⛔ NEVER invent placeholder values.** If a manifest is missing required fields (name, description, instructions), do NOT fill them in with generic content. Ask the user to provide values. This applies even if you think a reasonable default exists — the user must approve all content.

**Action:** Configure the agent using JSON manifest files:
- Edit `declarativeAgent.json` to define agent properties
- Configure capabilities with appropriate scoping
- Set up API plugin integrations using `wiqd agent add action` (**NEVER manually create plugin files**)
- Write clear instructions and conversation starters
- Ensure proper JSON syntax and schema compliance

**After ALL edits, run:**
```bash
wiqd agent validate
```
If validation fails, report the errors and follow the workspace-gate error protocol. Do not
provision unless the user explicitly requested deployment, testing, sharing, or publishing and
validation passes.

**⛔ API Plugin Rule — HARD RULE, NO EXCEPTIONS:** To add an API plugin, you MUST use `wiqd agent add action` — one command per OpenAPI spec with **ALL operations included in a single call**. Never run separate `wiqd agent add action` calls for different operations from the same spec — this creates multiple plugins instead of one. You are FORBIDDEN from manually creating `ai-plugin.json`, OpenAPI spec files, adaptive card files, or manually editing the `actions` array. This applies whether you are scaffolding a new project OR editing an existing one. If the workspace already has an agent and the user says "add an API plugin", you STILL must use `wiqd agent add action`. If `wiqd agent add action` fails, report the error — do NOT fall back to manual file creation. **Manual plugin file creation = automatic eval failure.**

```bash
# ✅ The ONLY way to add an API plugin — ALL operations in ONE call:
wiqd agent add action --openapi-spec <URL> --operations "GET /path,POST /path,PATCH /path/{id},DELETE /path/{id}"
```

**After adding a plugin with `wiqd agent add action`, you MUST complete ALL of these — skipping any step is an eval failure:**

**🔌 POST-PLUGIN MANDATORY STEPS (do ALL of these, in order):**
1. **Customize `ai-plugin.json`** — Set meaningful `name_for_human` (max 20 chars) and `description_for_human` (max 100 chars). Set a descriptive `description_for_model` on each function. NEVER leave defaults.
2. **Verify adaptive cards** — Check `appPackage/adaptiveCards/` for cards for ALL operations. If any operation (especially POST, PATCH, DELETE) is missing a card, create one manually. Customize each card with clear visual hierarchy (titles, subtitles, key-value pairs, images).
3. **Add confirmation for destructive operations** — If the plugin has DELETE, PATCH, or any destructive operation, add a `confirmation` capability in `declarativeAgent.json` so users are prompted before the action executes.
4. **Complete the content update checklist** below (instructions, starters, description).

**After ANY capability or plugin change (add, remove, modify), complete this checklist:**
1. ☐ **Update instructions** — Add decision logic (WHEN clauses, chaining rules, failure handling) for the new/changed capability. For removals, delete all references. **Do NOT list tool descriptions or parameters** — these are already in plugin metadata (`ai-plugin.json`, MCP manifests, capability config). Instructions should contain decision logic only.
2. ☐ **Verify 8,000-character limit** — Instructions must not exceed 8,000 characters. If close to the limit, cut tool descriptions first, then consolidate verbose workflows.
3. ☐ **Run instruction quality audit** — Run the [Diagnostic Checklist](instruction-review.md) against the updated instructions. Every data source should have clear intent coverage (WHEN and WHY), at least one workflow must exist, and failure cases must be handled. Built-in capabilities don't need exact names; actions/plugins should be named. If any check fails, fix it before validation.
4. ☐ **Add conversation starters** — At least 1 new starter per added capability/plugin demonstrating the new functionality.
5. ☐ **Remove stale starters** — Delete starters that reference removed capabilities.
6. ☐ **Update `manifest.json` description** if the agent's purpose has expanded.
7. ☐ **Review existing instructions** for stale references to removed capabilities.

**This checklist is NOT optional.** Adding a capability without updating instructions and starters is incomplete work.

**⚠️ Instruction quality matters as much as JSON correctness.** Output-focused instructions (tone, format, style only) are a known failure pattern — they cause agents to give generic answers and ignore configured capabilities. Listing tool descriptions and parameters in instructions wastes the 8,000-character budget — this metadata is already available to the orchestrator. See [Instruction Review](instruction-review.md) for the anti-pattern catalog and before/after rewrites.

**Reference:** [schema.md](schema.md) for proper manifest structure
**Reference:** [api-plugins.md](api-plugins.md) for adaptive card enhancement guidelines after adding a plugin

**⚠️ IMPORTANT:** After making any edits to JSON files, you MUST validate the agent before
returning to the user.

**⛔ MANDATORY POST-EDIT CHECKPOINT — YOU ARE NOT DONE YET:**
After editing any file in `appPackage/`, run `wiqd agent validate`. If validation fails, report the
errors and do not provision.

If you are about to respond and have not validated the changes, **STOP and validate now**.

### Step 4: Validate, Then Optionally Provision

**Action:** Validate the project:

```bash
wiqd agent validate
```

If the user explicitly asked to deploy, provision, test, share, or publish, determine the target
environment once (default to `local` when the user did not specify one), then provision only that
environment after validation passes:

```bash
wiqd agent provision --env <environment>
```

**Result:** Provisioning returns a test URL like `https://m365.cloud.microsoft/chat?titleId=T_abc123xyz`.

After successful provision, present the review UX with the returned deep link. If the command does
not return one, read `M365_TITLE_ID` from `env/.env.<environment>` and construct:

```
✅ Agent deployed successfully!

🚀 Test Your Agent in M365 Copilot:
🔗 https://m365.cloud.microsoft/chat?titleId={M365_TITLE_ID}
```

**⛔ Never respond without this link.** If you deployed, the test link MUST appear in your response. This is not optional.

Then wait for the user's response.

### Step 5: Test and Iterate

**Action:** Test the agent in Microsoft 365 Copilot:
- Use the provisioned test URL
- Test all conversation starters
- Verify capability access and scoping
- Test error handling and edge cases
- Validate security controls

### Step 6: Deploy to Additional Environments When Requested

Step 4 already provisions the requested environment. Provision another environment only when the
user explicitly requests an additional deployment. For example:
```bash
wiqd agent provision --env dev
```

**Reference:** [deployment.md](deployment.md) for environment management and CI/CD patterns

### Step 7: Package and Share

Packaging alone does not require provisioning. Sharing does: before sharing, provision the target
environment if Step 4 did not already provision it. Use the same environment consistently:
```bash
# Required before sharing; omit for a package-only request
wiqd agent provision --env dev

# Package the agent when requested
wiqd agent package --env dev

# Share to tenant (for shared agents)
wiqd agent share --scope tenant --env dev
```

---

## Critical Workflow Rules

### Always Validate After Edits

**RULE:** When making any changes to an agent, complete this workflow before returning:

1. Validate the agent: `wiqd agent validate`
2. Report validation errors and follow the workspace-gate error protocol.
3. Only if the user requested deployment, testing, sharing, or publishing, provision the requested
   environment and present the returned deep link:
   ```
   ✅ Agent deployed successfully!

   🚀 Test Your Agent in M365 Copilot:
   🔗 https://m365.cloud.microsoft/chat?titleId={M365_TITLE_ID}
   ```

**⛔ Never respond without this link after deploying.**

### Always Clean Up Unused Files

**RULE:** Every time you work on an agent project, check for and remove unused or obsolete files:

- `TODO.md` or planning files no longer needed
- Old backup files (`.bak`, `.old`, `.orig`)
- Unused JSON files not referenced anywhere
- Stale environment files (`.env.old`, `.env.backup`)
- Empty or placeholder files
- Outdated manifest versions
- Unused API plugin definitions

---

## ⛔ FINAL GATE — Before Responding to the User

**STOP.** Before writing your response to the user, verify ALL of the following:

- [ ] I ran `wiqd agent validate` and it succeeded
- [ ] If the user requested deployment, testing, sharing, or publishing, I provisioned the requested environment
- [ ] If I provisioned, I presented the returned deep link (or constructed it from `M365_TITLE_ID`)

**If you cannot check ALL boxes, you are NOT done.** Go back and complete the missing steps.

Validation applies after every edit. Provisioning remains an explicit user-requested operation.
