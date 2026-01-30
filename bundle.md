---
bundle:
  name: browser
  version: 0.1.0
  description: Token-efficient web automation and UX testing using agent-browser
  author: Amplifier Community
  license: Apache-2.0

includes:
  - bundle: foundation

context:
  browser-automation-guide:
    path: context/browser-automation-guide.md
  patterns:
    path: context/patterns.md
---

# Browser Automation Bundle

Token-efficient web automation for AI agents using [agent-browser](https://github.com/vercel-labs/agent-browser).

## What This Provides

This bundle enables AI agents to:
- Automate browser interactions with **93% less token usage** than Playwright MCP
- Test UX flows and visual components
- Scrape web content efficiently
- Fill forms and navigate sites
- Take screenshots and generate PDFs

## Key Innovation: Snapshot + Refs

Instead of sending thousands of DOM nodes, agent-browser provides a compact element list:

```
@e1: button "Sign In"
@e2: input[type=email] "Email"
@e3: input[type=password] "Password"
```

Agents interact using these deterministic refs:

```bash
agent-browser click @e1
agent-browser fill @e2 "user@example.com"
```

**Token savings**: A 5-step automation uses ~3.5k tokens instead of ~50k tokens (14x more efficient).

## Installation

agent-browser must be installed globally:

```bash
# Install CLI
npm install -g agent-browser

# Download Chromium browser
agent-browser install

# Linux only: Install system dependencies
agent-browser install --with-deps
```

## Quick Start

```bash
# Open a page
agent-browser open https://example.com

# Get interactive elements with refs
agent-browser snapshot -i --json

# Interact using refs
agent-browser click @e2
agent-browser fill @e3 "text"

# Take screenshot
agent-browser screenshot result.png

# Close browser
agent-browser close
```

## Context Files

- **context/browser-automation-guide.md** - Complete usage instructions
- **context/patterns.md** - Common workflows and best practices

## Use Cases

- **UX Testing**: Validate UI components and flows
- **Form Automation**: Fill and submit forms efficiently
- **Web Research**: Navigate and extract content
- **Visual Regression**: Screenshot comparison
- **Integration Testing**: End-to-end test automation

## Why Not Playwright MCP?

| Aspect | Playwright MCP | agent-browser |
|--------|---------------|---------------|
| Token per page | ~10,000 | ~700 |
| Setup complexity | JSON config required | Zero config |
| Element selection | CSS/XPath (fragile) | Deterministic refs |
| AI-friendly | No | Yes (designed for LLMs) |

## Architecture

```
Rust CLI (command parsing)
    ↓
Node.js Daemon (browser management)
    ↓
Chromium/Playwright (automation)
```

The daemon persists between commands for instant subsequent operations.

## Session Isolation

Multiple browser instances with isolated state:

```bash
# Different sessions
agent-browser --session test1 open site-a.com
agent-browser --session test2 open site-b.com

# Or via environment variable
AGENT_BROWSER_SESSION=test1 agent-browser click @e2
```

## Persistent Profiles

Maintain auth across sessions:

```bash
# Login once
agent-browser --profile ~/.myapp-profile open myapp.com

# Reuse authenticated session
agent-browser --profile ~/.myapp-profile open myapp.com/dashboard
```

## Cloud Deployment

For serverless environments, use cloud browser providers:

```bash
# Browserbase
export BROWSERBASE_API_KEY="key"
export BROWSERBASE_PROJECT_ID="id"
agent-browser -p browserbase open https://example.com

# Browser Use
export BROWSER_USE_API_KEY="key"
agent-browser -p browseruse open https://example.com

# Kernel
export KERNEL_API_KEY="key"
agent-browser -p kernel open https://example.com
```

## Documentation

- [Context Guide](context/browser-automation-guide.md) - Complete instructions
- [Common Patterns](context/patterns.md) - Workflow examples
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues
- [Upstream Docs](https://github.com/vercel-labs/agent-browser) - Full API reference

## Contributing

This bundle wraps the upstream agent-browser project. For:
- **Feature requests**: Open issue at [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser)
- **Amplifier patterns**: Contribute to this bundle's context files
- **Bug reports**: Check if it's agent-browser or Amplifier-specific

## License

Apache-2.0 (matching upstream agent-browser)
