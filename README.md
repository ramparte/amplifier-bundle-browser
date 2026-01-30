# Browser Automation Bundle

Token-efficient web automation for AI agents using [agent-browser](https://github.com/vercel-labs/agent-browser).

## Overview

This Amplifier bundle provides browser automation capabilities optimized for AI agents. By using agent-browser's **Snapshot + Refs** system, it achieves **93% token reduction** compared to traditional Playwright MCP approaches.

## Key Features

- **Token Efficient**: Uses refs (@e1, @e2) instead of full DOM trees
- **Zero Config**: No MCP server setup required
- **Deterministic**: Refs provide stable element targeting
- **AI-Friendly**: Designed specifically for LLM interaction
- **Session Isolation**: Multiple browser instances with separate state
- **Persistent Profiles**: Reuse auth across sessions
- **Cloud Ready**: Supports Browserbase, Browser Use, Kernel

## Installation

### 1. Install agent-browser

```bash
npm install -g agent-browser
agent-browser install
```

Linux users:
```bash
agent-browser install --with-deps
```

### 2. Use in Amplifier

This bundle provides context and patterns - no additional Amplifier setup needed. AI agents can use agent-browser directly through the bash tool.

## Quick Start

```bash
# Open page
agent-browser open https://example.com

# Get interactive elements with refs
agent-browser snapshot -i --json

# Returns:
# {
#   "success": true,
#   "data": {
#     "refs": {
#       "e1": {"role": "button", "name": "Sign In"},
#       "e2": {"role": "textbox", "name": "Email"},
#       "e3": {"role": "textbox", "name": "Password"}
#     }
#   }
# }

# Interact using refs
agent-browser fill @e2 "user@example.com"
agent-browser fill @e3 "password"
agent-browser click @e1

# Capture results
agent-browser screenshot result.png
agent-browser close
```

## Token Savings Example

### Playwright MCP (Traditional)
```
[... 10,000 lines of DOM tree ...]
<div id="app">
  <header>
    <nav>
      <ul>
        <li><a href="/home">Home</a></li>
        <li><a href="/about">About</a></li>
        ...
      </ul>
    </nav>
  </header>
  <main>
    <form class="login-form">
      <input type="email" name="email" placeholder="Email" />
      ...
[... thousands more lines ...]
```
**Cost**: ~10,000 tokens per page

### agent-browser
```bash
agent-browser snapshot -i --json
```
```json
{
  "refs": {
    "e1": {"role": "textbox", "name": "Email"},
    "e2": {"role": "textbox", "name": "Password"},
    "e3": {"role": "button", "name": "Sign In"}
  }
}
```
**Cost**: ~700 tokens per page

**Result**: 93% reduction. A 10-step automation uses ~7k tokens instead of ~100k tokens.

## Use Cases

### UX Testing
Validate UI components and interaction flows:
```bash
agent-browser open https://myapp.com/component
agent-browser snapshot -i --json
agent-browser click @e1  # Trigger modal
agent-browser wait --text "Modal Title"
agent-browser screenshot modal-open.png
```

### Form Automation
Fill and submit forms efficiently:
```bash
agent-browser open https://app.example.com/form
agent-browser snapshot -i --json
agent-browser fill @e1 "John Doe"
agent-browser fill @e2 "john@example.com"
agent-browser select @e3 "option-1"
agent-browser click @e4  # Submit
```

### Web Scraping
Extract content with minimal tokens:
```bash
agent-browser open https://docs.example.com
agent-browser snapshot -i -s "article" --json
agent-browser get text @e1 --json  # Title
agent-browser get html @e2 --json  # Content
```

### Visual Regression
Compare screenshots across deployments:
```bash
agent-browser open https://myapp.com/dashboard
agent-browser screenshot baseline.png --full
# Deploy changes
agent-browser reload
agent-browser screenshot current.png --full
# AI compares baseline vs current
```

## Documentation

- **[bundle.md](bundle.md)** - Bundle overview and architecture
- **[context/browser-automation-guide.md](context/browser-automation-guide.md)** - Complete command reference
- **[context/patterns.md](context/patterns.md)** - Common workflow examples
- **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)** - Solutions to common issues

## Architecture

```
┌─────────────────────────────────────────┐
│  Amplifier Agent (bash tool)            │
│                                          │
│  agent-browser CLI                       │
│         ↓                                │
├─────────────────────────────────────────┤
│  Rust CLI (fast parsing)                │
│         ↓                                │
│  Node.js Daemon (persistent)            │
│         ↓                                │
│  Playwright → Chromium                   │
└─────────────────────────────────────────┘
```

The daemon persists between commands for instant subsequent operations.

## Advanced Features

### Session Management

Run multiple isolated browser instances:
```bash
agent-browser --session user1 open site-a.com
agent-browser --session user2 open site-b.com
```

### Persistent Profiles

Maintain authentication across sessions:
```bash
agent-browser --profile ~/.myapp-profile open myapp.com
# Login once, reuse auth in future sessions
```

### Cloud Providers

For serverless/CI environments:
```bash
export BROWSERBASE_API_KEY="key"
export BROWSERBASE_PROJECT_ID="id"
agent-browser -p browserbase open https://example.com
```

Supported providers:
- **Browserbase** - Remote browser infrastructure
- **Browser Use** - Cloud browser for AI agents
- **Kernel** - Cloud browser with stealth mode

### CDP Mode

Connect to existing browsers:
```bash
# Chrome with remote debugging: chrome --remote-debugging-port=9222
agent-browser connect 9222
agent-browser snapshot -i --json
```

## Best Practices

### 1. Token Efficiency
- Always use `-i` flag: `snapshot -i --json`
- Combine filters: `snapshot -i -c -d 5`
- Scope selectors: `snapshot -i -s "#main"`
- Re-snapshot only after page changes

### 2. Reliability
- Use refs over CSS selectors (deterministic)
- Wait appropriately: `wait --text` or `wait --url`
- Check `success` field in JSON responses
- Handle errors gracefully (re-snapshot on failure)

### 3. Performance
- Reuse sessions (daemon persists)
- Use profiles (avoid repeated logins)
- Batch operations in scripts
- Minimize headed mode (debugging only)

## Token Usage Comparison

| Task | Playwright MCP | agent-browser | Savings |
|------|----------------|---------------|---------|
| Single page load | 10,000 tokens | 700 tokens | 93% |
| Form fill (3 fields) | 30,000 tokens | 2,100 tokens | 93% |
| 5-step navigation | 50,000 tokens | 3,500 tokens | 93% |
| 10-step automation | 100,000 tokens | 7,000 tokens | 93% |

## Example: Complete Login Flow

```bash
# 1. Navigate
agent-browser open https://app.example.com/login

# 2. Get form elements
agent-browser snapshot -i --json
# AI parses JSON, identifies: email (@e1), password (@e2), submit (@e3)

# 3. Fill credentials
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"

# 4. Submit
agent-browser click @e3

# 5. Wait for dashboard
agent-browser wait --url "**/dashboard"

# 6. Verify success
agent-browser get title --json
# {"success": true, "data": {"title": "Dashboard"}}

# 7. Close
agent-browser close
```

**Total tokens**: ~2,800 (vs ~40,000 with Playwright MCP)

## Debugging

### Show Browser Window
```bash
agent-browser --headed open https://example.com
```

### Check Console Errors
```bash
agent-browser console --json
agent-browser errors --json
```

### Trace Recording
```bash
agent-browser trace start trace.zip
# ... actions ...
agent-browser trace stop
npx playwright show-trace trace.zip
```

## Troubleshooting

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for solutions to common issues:
- Installation problems
- Browser launch errors
- Element not found
- Timeout issues
- Session management

## Contributing

This bundle wraps the upstream agent-browser project. 

**For agent-browser issues/features**:
- Repository: https://github.com/vercel-labs/agent-browser
- Issues: https://github.com/vercel-labs/agent-browser/issues

**For Amplifier-specific patterns**:
- Contribute workflow examples to `context/patterns.md`
- Add troubleshooting tips to `docs/TROUBLESHOOTING.md`
- Improve documentation clarity

## License

Apache-2.0 (matching upstream agent-browser)

## Links

- [agent-browser GitHub](https://github.com/vercel-labs/agent-browser)
- [Vercel Labs](https://vercel.com/labs)
- [Amplifier](https://github.com/microsoft/amplifier)

## Credits

agent-browser is developed by [Vercel Labs](https://vercel.com/labs).

This Amplifier bundle packages agent-browser capabilities with AI-friendly context and patterns.
