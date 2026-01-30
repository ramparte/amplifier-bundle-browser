# Usage Instructions

## For Amplifier Users

This bundle provides token-efficient browser automation capabilities using agent-browser.

### Prerequisites

**Install agent-browser globally** (one-time setup):

```bash
npm install -g agent-browser
agent-browser install
```

Linux users also need:
```bash
agent-browser install --with-deps
```

### Using the Bundle in Amplifier

#### Method 1: Load from GitHub (Recommended)

Once this bundle is published, you can reference it directly:

```bash
# In an Amplifier session, just ask naturally:
"Load the browser automation bundle from GitHub"

# Or load it directly:
amplifier run --bundle git+https://github.com/samschillace/amplifier-bundle-browser "Test my app's login flow"
```

#### Method 2: Local Directory

```bash
# Use local bundle path
amplifier run --bundle /home/samschillace/dev/ANext/amplifier-bundle-browser/bundle.md "Test the login flow"
```

#### Method 3: Add to Settings (Persistent)

Add to `~/.amplifier/settings.yaml`:

```yaml
bundles:
  - source: git+https://github.com/samschillace/amplifier-bundle-browser
    name: browser
```

Then in any session:
```bash
amplifier run --bundle browser "Automate this web task"
```

---

## For Amplifier Sessions (Agent Instructions)

If you are an AI agent in an Amplifier session and this bundle is loaded, you now have access to token-efficient browser automation.

### Quick Context

- **Tool**: Use `agent-browser` CLI via the bash tool
- **Key Innovation**: Snapshot + Refs system (93% token reduction)
- **Command Pattern**: `snapshot -i --json` → parse refs → interact using `@e1, @e2, etc.`
- **Always use `-i` flag**: Interactive elements only (700 tokens vs 10,000 tokens)

### Basic Workflow

```bash
# 1. Navigate
agent-browser open https://example.com

# 2. Get interactive elements with refs
agent-browser snapshot -i --json
# Parse JSON to identify elements: @e1 (button), @e2 (input), etc.

# 3. Interact using refs
agent-browser fill @e1 "user@example.com"
agent-browser click @e2

# 4. Capture results
agent-browser screenshot result.png
agent-browser get text @e3 --json

# 5. Close
agent-browser close
```

### Token Efficiency Rules

1. **Always use `-i` flag**: `snapshot -i --json` (not just `snapshot --json`)
2. **Combine filters**: `snapshot -i -c --json` for even more reduction
3. **Scope when possible**: `snapshot -i -s "#main" --json`
4. **Re-snapshot only after page changes**: After clicks, navigation, or AJAX

### Common Patterns Available

The bundle includes complete patterns for:
- UX component testing
- Form automation
- Login flows
- Web scraping
- Visual regression testing
- Multi-account testing

See `context/patterns.md` for detailed examples.

### Example: Login Flow

```bash
# Open login page
agent-browser open https://app.example.com/login

# Get form elements
agent-browser snapshot -i --json
# Parse JSON to find: @e1 (email), @e2 (password), @e3 (submit)

# Fill credentials
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"

# Submit
agent-browser click @e3

# Wait for redirect
agent-browser wait --url "**/dashboard"

# Verify success
agent-browser get title --json
```

### Session Isolation

Test multiple accounts simultaneously:

```bash
# User 1
agent-browser --session user1 open app.com/login
agent-browser --session user1 fill @e1 "user1@example.com"
agent-browser --session user1 click @e3

# User 2
agent-browser --session user2 open app.com/login
agent-browser --session user2 fill @e1 "user2@example.com"
agent-browser --session user2 click @e3
```

### Persistent Authentication

Avoid repeated logins:

```bash
# Login once with profile
agent-browser --profile ~/.myapp-profile open myapp.com
# ... login flow ...

# Reuse authenticated session later
agent-browser --profile ~/.myapp-profile open myapp.com/dashboard
```

### Error Handling

If `@e5` element not found:
```bash
# Re-snapshot to get fresh refs
agent-browser snapshot -i --json
# Use new ref
agent-browser click @e3  # Updated ref
```

### Debugging

```bash
# Show browser window
agent-browser --headed open https://example.com

# Check console errors
agent-browser console --json
agent-browser errors --json

# Take screenshots at each step
agent-browser screenshot step1.png
```

### Documentation References

When loaded, you have access to:
- **browser-automation-guide.md** - Complete command reference
- **patterns.md** - 10+ workflow examples
- **TROUBLESHOOTING.md** - Common issues and solutions

### Key Constraints

- **JSON output required**: Always use `--json` flag for parsing
- **Check success field**: `{"success": true/false, "data": {...}}`
- **Re-snapshot after changes**: Page updates invalidate old refs
- **Wait appropriately**: Use `wait --text` or `wait --url`, not arbitrary timeouts

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Navigate | `agent-browser open <url>` |
| Get elements | `agent-browser snapshot -i --json` |
| Click | `agent-browser click @e1` |
| Fill input | `agent-browser fill @e2 "text"` |
| Get text | `agent-browser get text @e3 --json` |
| Screenshot | `agent-browser screenshot page.png` |
| Wait for element | `agent-browser wait @e4` |
| Wait for text | `agent-browser wait --text "Success"` |
| Check visibility | `agent-browser is visible @e5 --json` |
| Close | `agent-browser close` |

## Support

- **Bundle Issues**: File issue at the bundle repository
- **agent-browser Issues**: https://github.com/vercel-labs/agent-browser/issues
- **Documentation**: See README.md and context/ files in this bundle

## License

Apache-2.0
