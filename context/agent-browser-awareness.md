# CRITICAL: You Have agent-browser Available

**This session is configured with agent-browser for browser automation.**

## What This Means

You have access to `agent-browser` CLI via the bash tool for:
- Opening web pages
- Taking screenshots
- Interacting with forms
- Testing UX components
- Web scraping

## Default Tool Selection

When the user asks you to:
- "open a website"
- "take a screenshot"
- "test a page"
- "fill a form"
- "click a button"
- "automate browser interaction"

**YOU MUST use `agent-browser` via bash, NOT web_fetch or other tools.**

## Quick Reference

```bash
# Open a page
agent-browser open https://example.com

# Get interactive elements with refs
agent-browser snapshot -i --json

# Take a screenshot
agent-browser screenshot example.png

# Interact using refs
agent-browser click @e1
agent-browser fill @e2 "text"

# Close
agent-browser close
```

## Why Use agent-browser

- **Token efficient**: 700 tokens vs 10,000+ with other methods
- **Full browser**: Real Chrome/Chromium, not just HTTP requests
- **Screenshots**: Native screenshot support
- **JavaScript**: Handles dynamic pages, SPAs, React apps
- **Interactive**: Can click, fill, navigate like a user

## Installation Check

agent-browser is installed globally via npm. If a command fails with "command not found":
1. Check with: `which agent-browser`
2. Verify installation: `npm list -g agent-browser`
3. If missing, tell user: "agent-browser needs to be installed with: npm install -g agent-browser && agent-browser install"

## Common Mistake: Using web_fetch

❌ **WRONG:**
```bash
curl https://example.com
# or
web_fetch https://example.com
```

✅ **CORRECT:**
```bash
agent-browser open https://example.com
agent-browser screenshot page.png
```

web_fetch gives you HTML source. agent-browser gives you a real browser with rendering, JavaScript, and screenshots.

---

**Remember**: If the task involves a browser or webpage, use agent-browser first. Only fall back to other tools if agent-browser truly cannot handle it.
