# Troubleshooting Guide

Common issues and solutions when using agent-browser with Amplifier.

## Installation Issues

### "npm: command not found"

**Problem**: Node.js/npm not installed.

**Solution**:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nodejs npm

# macOS
brew install node

# Or use nvm for version management
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
```

### "agent-browser: command not found" after installation

**Problem**: Global npm bin directory not in PATH.

**Solution**:
```bash
# Find npm global bin path
npm config get prefix

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$PATH:$(npm config get prefix)/bin"

# Reload shell
source ~/.bashrc
```

### "Permission denied" during npm install

**Problem**: Insufficient permissions for global install.

**Solution**:
```bash
# Option 1: Use npx (no global install needed)
npx agent-browser install
npx agent-browser open https://example.com

# Option 2: Fix npm permissions (recommended)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH
npm install -g agent-browser

# Option 3: Use sudo (not recommended)
sudo npm install -g agent-browser
```

---

## Browser Launch Issues

### "Executable doesn't exist at chromium_headless_shell"

**Problem**: Chromium not downloaded or version mismatch.

**Solution**:
```bash
# Download Chromium
agent-browser install

# If still failing, install matching Playwright browsers
npx playwright@1.57.0 install chromium chromium-headless-shell

# Ensure version consistency if using npx
npx agent-browser@0.4.4 install
npx agent-browser@0.4.4 open https://example.com
```

### "Browser closed unexpectedly"

**Problem**: Missing system dependencies (Linux).

**Solution**:
```bash
# Install system dependencies
agent-browser install --with-deps

# Or manually (Ubuntu/Debian)
sudo apt-get install -y \
  libnss3 libnspr4 libatk1.0-0 libatk-bridge2.0-0 \
  libcups2 libdrm2 libxkbcommon0 libxcomposite1 \
  libxdamage1 libxfixes3 libxrandr2 libgbm1 \
  libasound2 libpango-1.0-0 libcairo2
```

### "--executable-path ignored: daemon already running"

**Problem**: Browser daemon running with different executable.

**Solution**:
```bash
# Close existing daemon
agent-browser close

# Or kill all browser processes
pkill -f "agent-browser"

# Then specify executable path
agent-browser --executable-path "/path/to/chrome" open https://example.com
```

### "Timeout waiting for browser to launch"

**Problem**: Slow system or resource constraints.

**Solution**:
```bash
# Check system resources
free -h
df -h

# Close other applications
# Or use cloud provider instead
export BROWSERBASE_API_KEY="key"
export BROWSERBASE_PROJECT_ID="id"
agent-browser -p browserbase open https://example.com
```

---

## Runtime Errors

### "Element not found: @e5"

**Problem**: Element no longer exists or page changed.

**Solution**:
```bash
# Re-run snapshot to get fresh refs
agent-browser snapshot -i --json

# Verify element exists in new snapshot
# Use new ref for interaction
agent-browser click @e3  # Updated ref
```

**Prevention**: Always re-snapshot after page changes (clicks, navigation, AJAX).

### "Browser not running"

**Problem**: No active browser session.

**Solution**:
```bash
# Check if browser is running
agent-browser get url --json

# If not running, open a page first
agent-browser open https://example.com

# Then proceed with commands
agent-browser snapshot -i --json
```

### "Timeout waiting for element"

**Problem**: Element taking longer to appear than default timeout.

**Solution**:
```bash
# Increase timeout (default is 30 seconds)
agent-browser wait @e5 --timeout 60000

# Or wait for specific condition
agent-browser wait --text "Loading complete"
agent-browser wait --load networkidle

# Or use polling approach
agent-browser wait 2000
agent-browser is visible @e5 --json
# If false, wait more and retry
```

### "Navigation timeout"

**Problem**: Page taking too long to load.

**Solution**:
```bash
# Open a fast page first
agent-browser open https://example.com

# Then navigate via eval (no timeout)
agent-browser eval "window.location.href='https://slow-site.com'"

# Wait for load
agent-browser wait --load domcontentloaded
agent-browser wait 5000
```

### "Failed to parse JSON output"

**Problem**: Command output is not valid JSON or includes extra text.

**Solution**:
```bash
# Ensure --json flag is used
agent-browser snapshot -i --json

# Check for errors in stderr
agent-browser get text @e1 --json 2>&1

# If output includes non-JSON text, parse carefully
# Extract only the JSON portion
```

---

## Session Management Issues

### "Session not found: user1"

**Problem**: Session doesn't exist or was closed.

**Solution**:
```bash
# List active sessions
agent-browser session list

# Create session explicitly
agent-browser --session user1 open https://example.com

# Or check environment variable
echo $AGENT_BROWSER_SESSION
```

### "Multiple browsers running with same session"

**Problem**: Daemon confusion with session isolation.

**Solution**:
```bash
# Close all sessions
agent-browser close

# Use explicit session names
agent-browser --session session1 open site-a.com
agent-browser --session session2 open site-b.com

# Always specify session for commands
agent-browser --session session1 click @e2
```

### "Profile directory locked"

**Problem**: Another process using the same profile.

**Solution**:
```bash
# Close browsers using that profile
agent-browser --profile ~/.myapp-profile close

# Or use different profile path
agent-browser --profile ~/.myapp-profile-temp open https://example.com

# Or clear lock file (dangerous - ensure no other process)
rm -f ~/.myapp-profile/.lock
```

---

## Network Issues

### "net::ERR_CONNECTION_REFUSED"

**Problem**: Server not reachable or localhost issue.

**Solution**:
```bash
# Verify URL is correct
agent-browser open https://example.com

# For localhost, use 127.0.0.1
agent-browser open http://127.0.0.1:3000

# Check if site is up
curl -I https://example.com
```

### "SSL certificate error"

**Problem**: Self-signed certificate or HTTPS issue.

**Solution**:
```bash
# Ignore HTTPS errors (use with caution)
agent-browser --ignore-https-errors open https://self-signed.example.com
```

### "Request blocked by CORS"

**Problem**: Cross-origin request blocked.

**Solution**:
```bash
# Use proxy or set headers
agent-browser --headers '{"Origin": "https://allowed-origin.com"}' open https://api.example.com

# Or use CDP to bypass CORS
agent-browser eval "fetch('https://api.example.com/data').then(r => r.text())"
```

---

## Performance Issues

### "Commands are slow"

**Problem**: Daemon not running or overhead from snapshots.

**Solution**:
```bash
# Ensure daemon is running (first command starts it)
agent-browser open https://example.com
# Subsequent commands should be instant

# Use compact snapshots
agent-browser snapshot -i -c --json

# Scope snapshots to specific area
agent-browser snapshot -i -s "#main-content" --json

# Avoid full page snapshots
# BAD: agent-browser snapshot --json (10k+ tokens)
# GOOD: agent-browser snapshot -i --json (700 tokens)
```

### "High memory usage"

**Problem**: Too many browser sessions or large pages.

**Solution**:
```bash
# Close unused sessions
agent-browser --session old-session close

# Set viewport to smaller size
agent-browser set viewport 1024 768

# Use headless mode (not --headed)
agent-browser open https://example.com

# Monitor resources
top -p $(pgrep -f agent-browser)
```

---

## Cloud Provider Issues

### Browserbase: "Invalid API key"

**Problem**: API key not set or incorrect.

**Solution**:
```bash
# Verify environment variable
echo $BROWSERBASE_API_KEY

# Set API key
export BROWSERBASE_API_KEY="bb_live_..."
export BROWSERBASE_PROJECT_ID="proj_..."

# Test connection
agent-browser -p browserbase open https://example.com --json
```

### Browser Use: "Session creation failed"

**Problem**: API issue or quota exceeded.

**Solution**:
```bash
# Check API key
echo $BROWSER_USE_API_KEY

# Verify quota/balance
# Visit: https://cloud.browser-use.com/settings

# Retry with explicit provider
export AGENT_BROWSER_PROVIDER=browseruse
agent-browser open https://example.com
```

### Kernel: "Profile not found"

**Problem**: Profile name doesn't exist or creation failed.

**Solution**:
```bash
# Profile is created automatically on first use
export KERNEL_PROFILE_NAME="my-profile"
agent-browser -p kernel open https://example.com

# Verify profile created
# Check Kernel dashboard: https://dashboard.onkernel.com

# Use different profile name if needed
export KERNEL_PROFILE_NAME="test-profile-2"
```

---

## Platform-Specific Issues

### Linux: "libgobject-2.0.so.0: cannot open shared object file"

**Problem**: Missing system libraries.

**Solution**:
```bash
# Install dependencies
agent-browser install --with-deps

# Or manually
sudo apt-get install -y libgobject-2.0-0 libglib2.0-0
```

### macOS: "App is damaged and can't be opened"

**Problem**: Chromium quarantine on macOS.

**Solution**:
```bash
# Remove quarantine attribute
xattr -cr ~/.cache/ms-playwright/chromium-*/

# Or download again
agent-browser install
```

### Windows/WSL: "ECONNREFUSED connecting to localhost"

**Problem**: WSL networking issue.

**Solution**:
```bash
# Use explicit 127.0.0.1 instead of localhost
agent-browser open http://127.0.0.1:3000

# Or use Windows host IP
agent-browser open http://$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):3000
```

---

## Debugging Techniques

### Enable Debug Logging

```bash
# Set debug flag
agent-browser --debug open https://example.com

# Or use environment variable
export AGENT_BROWSER_DEBUG=1
agent-browser open https://example.com
```

### View Console Messages

```bash
# Open page and check console
agent-browser open https://example.com
agent-browser console --json

# Filter by type
agent-browser console --json | jq '.data[] | select(.type=="error")'
```

### View Page Errors

```bash
# Check for JavaScript errors
agent-browser errors --json

# Clear errors
agent-browser errors --clear
```

### Take Screenshots for Visual Debugging

```bash
# Screenshot at each step
agent-browser open https://example.com
agent-browser screenshot step1.png

agent-browser click @e2
agent-browser screenshot step2.png

agent-browser fill @e3 "data"
agent-browser screenshot step3.png
```

### Record Trace

```bash
# Start trace
agent-browser trace start trace.zip

# Perform actions
agent-browser open https://example.com
agent-browser click @e2
agent-browser fill @e3 "data"

# Stop trace
agent-browser trace stop

# View in Playwright Trace Viewer
npx playwright show-trace trace.zip
```

### Use Headed Mode

```bash
# See browser window
agent-browser --headed open https://example.com

# Keep window open after commands
agent-browser --headed open https://example.com
agent-browser --headed click @e2
# Window stays open - manually verify state
```

---

## Getting Help

### Check Version

```bash
agent-browser --version
npm list -g agent-browser
```

### Review Documentation

- [GitHub Repository](https://github.com/vercel-labs/agent-browser)
- [Issues](https://github.com/vercel-labs/agent-browser/issues)
- [Browser Automation Guide](../context/browser-automation-guide.md)
- [Common Patterns](../context/patterns.md)

### Report Issues

When reporting issues, include:

1. **Version information**:
   ```bash
   agent-browser --version
   node --version
   npm --version
   ```

2. **Platform**:
   ```bash
   uname -a
   ```

3. **Command that failed**:
   ```bash
   agent-browser --debug <command> --json
   ```

4. **Error output**:
   - Full error message
   - Stack trace if available
   - Console output

5. **Minimal reproduction**:
   - Smallest command sequence that reproduces the issue

### Community Resources

- [GitHub Discussions](https://github.com/vercel-labs/agent-browser/discussions)
- [Vercel Discord](https://vercel.com/discord)
- [Amplifier Community](https://github.com/microsoft/amplifier/discussions)

---

## Common Anti-Patterns

### ❌ Not re-snapshotting after page changes

```bash
agent-browser click @e2
# Page changed, but refs still reference old elements
agent-browser click @e5  # FAILS: Element not found
```

**✅ Correct approach**:
```bash
agent-browser click @e2
agent-browser wait 2000
agent-browser snapshot -i --json  # Get fresh refs
agent-browser click @e5  # Works with new ref
```

### ❌ Using arbitrary timeouts

```bash
agent-browser click @e2
agent-browser wait 5000  # Hope it's done?
agent-browser snapshot -i --json
```

**✅ Correct approach**:
```bash
agent-browser click @e2
agent-browser wait --text "Success"  # Wait for specific condition
agent-browser snapshot -i --json
```

### ❌ Full snapshots in production

```bash
agent-browser snapshot --json  # 10,000+ tokens
```

**✅ Correct approach**:
```bash
agent-browser snapshot -i -c --json  # 500 tokens
```

### ❌ Hard-coded CSS selectors

```bash
agent-browser click "#app > div.container > main > form > button.submit-btn-primary"
```

**✅ Correct approach**:
```bash
agent-browser snapshot -i --json
# Parse to find submit button ref
agent-browser click @e5
```

---

## Quick Reference: Common Fixes

| Problem | Quick Fix |
|---------|-----------|
| "command not found" | `npm install -g agent-browser` |
| "Executable doesn't exist" | `agent-browser install` |
| "Element not found" | Re-run `snapshot -i --json` |
| "Browser not running" | Run `open` command first |
| "Timeout" | Use `wait --text` or increase timeout |
| Slow commands | Ensure daemon running, use `-i -c` |
| High tokens | Always use `-i` flag for snapshots |
| Multiple sessions | Use explicit `--session` names |
| SSL errors | Use `--ignore-https-errors` |

---

If your issue isn't covered here, check the [GitHub issues](https://github.com/vercel-labs/agent-browser/issues) or open a new issue with details.
