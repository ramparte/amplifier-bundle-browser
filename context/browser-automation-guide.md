# Browser Automation with agent-browser

This guide provides comprehensive instructions for using agent-browser, a token-efficient browser automation CLI designed specifically for AI agents.

## Core Concept: Snapshot + Refs

agent-browser uses a **deterministic reference system** that eliminates token bloat from traditional browser automation.

### Traditional Approach (Playwright MCP)
```
[... 10,000 lines of DOM tree ...]
<div id="main">
  <form class="login-form">
    <input type="email" name="email" placeholder="Email" />
    <input type="password" name="password" placeholder="Password" />
    <button type="submit">Sign In</button>
  </form>
</div>
[... thousands more lines ...]
```

**Problem**: 10,000+ tokens per page interaction. Context window exhaustion on complex sites.

### agent-browser Approach
```bash
agent-browser snapshot -i --json
```

Returns:
```json
{
  "success": true,
  "data": {
    "snapshot": "...",
    "refs": {
      "e1": {"role": "textbox", "name": "Email"},
      "e2": {"role": "textbox", "name": "Password"},
      "e3": {"role": "button", "name": "Sign In"}
    }
  }
}
```

**Result**: ~700 tokens. Then interact using refs:
```bash
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3
```

## Installation & Setup

```bash
# Install CLI globally
npm install -g agent-browser

# Download Chromium browser
agent-browser install

# Linux only: Install system dependencies
agent-browser install --with-deps
```

**Verification:**
```bash
agent-browser --help
# Should show command list
```

## Basic Workflow

### 1. Navigate to Page
```bash
agent-browser open https://example.com
```

**Options:**
- `--headed` - Show browser window for debugging
- `--session <name>` - Use isolated session
- `--profile <path>` - Persistent auth/cookies

### 2. Get Page Snapshot
```bash
agent-browser snapshot -i --json
```

**Snapshot Options:**
- `-i` / `--interactive` - Only interactive elements (buttons, inputs, links) - **recommended**
- `-c` / `--compact` - Remove empty structural elements
- `-d <n>` / `--depth <n>` - Limit tree depth
- `-s <sel>` / `--selector <sel>` - Scope to CSS selector
- `--json` - Structured JSON output (required for parsing)

**When to snapshot:**
- After navigation
- After page changes (clicks, form submissions)
- When you need to find new elements

### 3. Interact Using Refs
```bash
agent-browser click @e2           # Click element
agent-browser fill @e3 "text"     # Fill input
agent-browser get text @e1        # Extract text
agent-browser hover @e4           # Hover element
```

### 4. Capture Results
```bash
agent-browser screenshot result.png
agent-browser pdf report.pdf
agent-browser get url  # Current URL
agent-browser get title  # Page title
```

### 5. Close Browser
```bash
agent-browser close
```

## Command Reference

### Navigation
```bash
agent-browser open <url>          # Navigate to URL
agent-browser back                # Go back
agent-browser forward             # Go forward
agent-browser reload              # Reload page
agent-browser wait 2000           # Wait 2 seconds
agent-browser wait @e5            # Wait for element
```

### Element Interaction
```bash
agent-browser click @e1           # Click element
agent-browser dblclick @e1        # Double-click
agent-browser fill @e2 "text"     # Clear and fill input
agent-browser type @e2 "text"     # Type into input (no clear)
agent-browser press Enter         # Press key
agent-browser hover @e3           # Hover element
agent-browser check @e4           # Check checkbox
agent-browser uncheck @e4         # Uncheck checkbox
agent-browser select @e5 "value"  # Select dropdown option
agent-browser upload @e6 "file.pdf"  # Upload file
```

### Information Extraction
```bash
agent-browser get text @e1        # Get text content
agent-browser get value @e2       # Get input value
agent-browser get html @e3        # Get innerHTML
agent-browser get attr @e4 href   # Get attribute
agent-browser get title           # Page title
agent-browser get url             # Current URL
agent-browser is visible @e5      # Check visibility
agent-browser is enabled @e6      # Check if enabled
agent-browser is checked @e7      # Check if checked
```

### Screenshots & PDFs
```bash
agent-browser screenshot page.png             # Viewport screenshot
agent-browser screenshot page.png --full      # Full page screenshot
agent-browser pdf report.pdf                  # Save as PDF
```

### Alternative Selectors (CSS, Text, Semantic)

While refs are recommended, agent-browser also supports traditional selectors:

```bash
# CSS selectors
agent-browser click "#submit-button"
agent-browser fill ".email-input" "user@example.com"

# Text selectors
agent-browser click "text=Sign In"
agent-browser find text "Submit" click

# Semantic locators (ARIA roles)
agent-browser find role button click --name "Submit"
agent-browser find label "Email" fill "user@example.com"
agent-browser find placeholder "Search..." type "query"
```

## Session Management

Run multiple isolated browser instances:

```bash
# Start different sessions
agent-browser --session user1 open site-a.com
agent-browser --session user2 open site-b.com

# Interact with specific session
agent-browser --session user1 click @e2
agent-browser --session user2 fill @e3 "data"

# Or use environment variable
export AGENT_BROWSER_SESSION=user1
agent-browser click @e2

# List active sessions
agent-browser session list
```

Each session has:
- Separate browser instance
- Isolated cookies and storage
- Independent navigation history
- Separate authentication state

## Persistent Profiles

By default, browser state (cookies, localStorage, logins) is ephemeral. Use `--profile` to persist across sessions:

```bash
# Login once with profile
agent-browser --profile ~/.myapp-profile open myapp.com
# ... login flow ...

# Reuse authenticated session later
agent-browser --profile ~/.myapp-profile open myapp.com/dashboard
# Already logged in!
```

**Profile stores:**
- Cookies and localStorage
- IndexedDB data
- Service workers
- Browser cache
- Login sessions

**Tip**: Use different profile paths for different accounts/projects.

## JSON Output for Structured Parsing

Always use `--json` flag for programmatic parsing:

```bash
agent-browser snapshot -i --json
agent-browser get text @e1 --json
agent-browser is visible @e2 --json
```

**Example JSON response:**
```json
{
  "success": true,
  "data": {
    "text": "Welcome, User!"
  }
}
```

**Error response:**
```json
{
  "success": false,
  "error": "Element not found: @e999"
}
```

**Always check `success` field before using `data`.**

## Token Efficiency Best Practices

### 1. Use Snapshot Filters

```bash
# GOOD: Only interactive elements (700 tokens)
agent-browser snapshot -i --json

# BETTER: Interactive + compact (500 tokens)
agent-browser snapshot -i -c --json

# BAD: Full tree (10,000+ tokens)
agent-browser snapshot --json
```

### 2. Scope Snapshots When Possible

```bash
# Only snapshot the form area
agent-browser snapshot -i -s "#login-form" --json
```

### 3. Re-snapshot Only When Needed

```bash
# Initial snapshot
agent-browser snapshot -i --json

# Multiple interactions without re-snapshot
agent-browser fill @e2 "user@example.com"
agent-browser fill @e3 "password"
agent-browser click @e4

# Re-snapshot only after page changes
agent-browser wait 2000
agent-browser snapshot -i --json
```

### 4. Use Deterministic Refs Over CSS Selectors

```bash
# GOOD: Deterministic from snapshot
agent-browser click @e2

# BAD: Fragile, may break if page changes
agent-browser click "#submit-btn-primary-login-form-v2"
```

## Common Patterns

### Pattern: Login Flow

```bash
# 1. Open login page
agent-browser open https://app.example.com/login

# 2. Get form elements
agent-browser snapshot -i --json
# Parse JSON to find email (@e1), password (@e2), submit (@e3)

# 3. Fill credentials
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"

# 4. Submit
agent-browser click @e3

# 5. Wait for redirect
agent-browser wait --url "**/dashboard"

# 6. Verify success
agent-browser get title --json
# Parse: {"success": true, "data": {"title": "Dashboard"}}
```

### Pattern: Form Automation

```bash
# Open form
agent-browser open https://forms.example.com

# Get snapshot
agent-browser snapshot -i --json

# Fill all fields
agent-browser fill @e1 "John Doe"
agent-browser fill @e2 "john@example.com"
agent-browser select @e3 "option-1"
agent-browser check @e4
agent-browser upload @e5 "/path/to/file.pdf"

# Submit
agent-browser click @e6

# Wait for success message
agent-browser wait --text "Thank you"
```

### Pattern: Web Scraping

```bash
# Navigate to page
agent-browser open https://docs.example.com/article

# Get snapshot to find content area
agent-browser snapshot -i -s "article" --json

# Extract text from specific elements
agent-browser get text @e1 --json  # Title
agent-browser get text @e2 --json  # Body

# Screenshot for reference
agent-browser screenshot article.png

# Navigate to next page
agent-browser click @e5  # "Next" link
```

### Pattern: Visual Regression Testing

```bash
# Take baseline screenshot
agent-browser open https://myapp.com/dashboard
agent-browser screenshot baseline.png --full

# Make changes (deploy, etc.)

# Take comparison screenshot
agent-browser reload
agent-browser screenshot current.png --full

# AI can now compare baseline.png vs current.png
```

## Debugging

### Show Browser Window

```bash
agent-browser --headed open https://example.com
```

This opens a visible browser window so you can see what's happening.

### Check Console Messages

```bash
agent-browser console --json
# Shows: log, error, warn, info messages

agent-browser errors --json
# Shows: uncaught JavaScript exceptions
```

### Highlight Elements

```bash
agent-browser highlight @e2
# Visually highlights element in browser (use with --headed)
```

### Save Browser Trace

```bash
agent-browser trace start trace.zip
# ... perform actions ...
agent-browser trace stop

# Open trace.zip in Playwright Trace Viewer
# npx playwright show-trace trace.zip
```

## Advanced Features

### Custom Browser Executable

```bash
# Use specific Chrome/Chromium
agent-browser --executable-path "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" open example.com

# Or via environment variable
export AGENT_BROWSER_EXECUTABLE_PATH="/path/to/chromium"
agent-browser open example.com
```

### Connect to Existing Browser (CDP)

```bash
# Start Chrome with remote debugging:
# google-chrome --remote-debugging-port=9222

# Connect
agent-browser connect 9222
agent-browser snapshot -i --json
agent-browser close
```

### Cloud Browser Providers

For serverless/CI environments:

```bash
# Browserbase
export BROWSERBASE_API_KEY="key"
export BROWSERBASE_PROJECT_ID="id"
agent-browser -p browserbase open https://example.com

# Browser Use
export BROWSER_USE_API_KEY="key"
agent-browser -p browseruse open https://example.com

# Kernel (with stealth mode)
export KERNEL_API_KEY="key"
export KERNEL_STEALTH=true
agent-browser -p kernel open https://example.com
```

## Error Handling

### Common Errors

**"Element not found: @e5"**
- Element no longer exists (page changed)
- Re-run `snapshot` to get fresh refs

**"Browser not running"**
- No active browser session
- Run `open` command first

**"Timeout waiting for element"**
- Element didn't appear in time
- Increase timeout: `agent-browser wait @e5 --timeout 30000`

**"Command failed with exit code 1"**
- Check `agent-browser console --json` for JavaScript errors
- Use `--headed` to visually debug

## Performance Tips

1. **Reuse sessions** - The daemon persists, so subsequent commands are instant
2. **Batch commands** - Use bash loops/scripts to run multiple commands efficiently
3. **Minimize snapshots** - Only snapshot after page changes
4. **Use compact mode** - `snapshot -i -c` removes noise
5. **Scope selectors** - `snapshot -i -s "#main"` reduces tree size

## Token Usage Comparison

| Task | Playwright MCP | agent-browser | Savings |
|------|----------------|---------------|---------|
| Single page load | 10,000 tokens | 700 tokens | 93% |
| Form fill (3 fields) | 30,000 tokens | 2,100 tokens | 93% |
| 5-step navigation | 50,000 tokens | 3,500 tokens | 93% |
| 10-step automation | 100,000 tokens | 7,000 tokens | 93% |

**Result**: Browser automation that doesn't blow your context window.

## When NOT to Use agent-browser

- **Immediate feedback needed** - For rapid iteration, use headed browser manually
- **Complex JavaScript execution** - If you need to run custom JS extensively, consider Playwright directly
- **Non-browser automation** - Use appropriate tools (selenium, requests, etc.)

## Further Reading

- [GitHub Repository](https://github.com/vercel-labs/agent-browser) - Full API documentation
- [Common Patterns](patterns.md) - Workflow examples
- [Troubleshooting](../docs/TROUBLESHOOTING.md) - Solutions to common issues
