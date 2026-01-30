# Common Browser Automation Patterns

This document provides proven workflows for common browser automation tasks using agent-browser.

## Pattern Index

- [UX Component Testing](#ux-component-testing)
- [Multi-Page Form Flows](#multi-page-form-flows)
- [Authentication & Session Management](#authentication--session-management)
- [Web Scraping & Content Extraction](#web-scraping--content-extraction)
- [Visual Regression Testing](#visual-regression-testing)
- [Dynamic Content Handling](#dynamic-content-handling)
- [File Upload & Download](#file-upload--download)
- [Multiple Account Management](#multiple-account-management)
- [Error Recovery](#error-recovery)
- [Performance Testing](#performance-testing)

---

## UX Component Testing

**Goal**: Validate UI components work correctly across interactions.

### Basic Component Test

```bash
# Open component page
agent-browser open https://myapp.com/components/modal

# Get initial snapshot
agent-browser snapshot -i --json
# Parse to find trigger button (@e1)

# Trigger modal
agent-browser click @e1

# Wait for modal to appear
agent-browser wait --text "Modal Title"

# Get new snapshot (modal elements)
agent-browser snapshot -i --json
# Parse to find close button (@e5), submit (@e6)

# Verify modal contents
agent-browser get text @e5 --json  # Check text

# Take screenshot for visual verification
agent-browser screenshot modal-open.png

# Close modal
agent-browser click @e5

# Verify modal closed
agent-browser wait 1000
agent-browser is visible @e5 --json
# Should return: {"success": true, "data": {"visible": false}}
```

### Responsive Component Testing

```bash
# Test desktop view
agent-browser set viewport 1920 1080
agent-browser open https://myapp.com/component
agent-browser screenshot desktop.png

# Test tablet view
agent-browser set viewport 768 1024
agent-browser reload
agent-browser screenshot tablet.png

# Test mobile view
agent-browser set viewport 375 667
agent-browser reload
agent-browser screenshot mobile.png

# Or use device emulation
agent-browser set device "iPhone 14"
agent-browser reload
agent-browser screenshot iphone14.png
```

### Interaction Testing

```bash
# Test hover states
agent-browser open https://myapp.com/buttons
agent-browser snapshot -i --json

agent-browser hover @e1
agent-browser screenshot button-hover.png

# Test focus states
agent-browser focus @e2
agent-browser screenshot button-focus.png

# Test click states
agent-browser click @e3
agent-browser screenshot button-active.png
```

---

## Multi-Page Form Flows

**Goal**: Complete forms that span multiple pages with validation.

### Wizard Flow with Validation

```bash
# Step 1: Personal Info
agent-browser open https://app.example.com/signup/step1
agent-browser snapshot -i --json

agent-browser fill @e1 "John Doe"
agent-browser fill @e2 "john@example.com"
agent-browser click @e3  # Next button

# Wait for validation or next page
agent-browser wait --url "**/step2"

# Step 2: Address Info
agent-browser snapshot -i --json

agent-browser fill @e1 "123 Main St"
agent-browser fill @e2 "New York"
agent-browser select @e3 "NY"
agent-browser fill @e4 "10001"
agent-browser click @e5  # Next button

agent-browser wait --url "**/step3"

# Step 3: Payment Info
agent-browser snapshot -i --json

agent-browser fill @e1 "4111111111111111"
agent-browser fill @e2 "12/25"
agent-browser fill @e3 "123"
agent-browser click @e4  # Submit button

# Wait for confirmation
agent-browser wait --text "Thank you"
agent-browser screenshot confirmation.png
```

### Handling Validation Errors

```bash
# Fill form with invalid data
agent-browser fill @e2 "invalid-email"
agent-browser click @e3  # Submit

# Wait for error message
agent-browser wait --text "Invalid email"

# Get error text
agent-browser snapshot -i --json
agent-browser get text @e4 --json
# Parse error message

# Correct the error
agent-browser fill @e2 "valid@example.com"
agent-browser click @e3

# Verify success
agent-browser wait --url "**/success"
```

---

## Authentication & Session Management

### Login with Persistent Profile

```bash
# Create persistent profile for authenticated sessions
agent-browser --profile ~/.myapp-profile open https://app.example.com/login

agent-browser snapshot -i --json
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3

# Wait for dashboard
agent-browser wait --url "**/dashboard"

# Close browser (profile persists)
agent-browser close

# Later: Reuse authenticated session
agent-browser --profile ~/.myapp-profile open https://app.example.com/dashboard
# Already logged in!
```

### Multi-Account Testing

```bash
# Session 1: Admin account
agent-browser --session admin --profile ~/.profiles/admin open app.com/login
agent-browser snapshot -i --json
agent-browser fill @e1 "admin@example.com"
agent-browser fill @e2 "admin-pass"
agent-browser click @e3

# Session 2: Regular user
agent-browser --session user --profile ~/.profiles/user open app.com/login
agent-browser snapshot -i --json
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "user-pass"
agent-browser click @e3

# Test admin functionality
agent-browser --session admin open app.com/admin
agent-browser snapshot -i --json
# Verify admin panel visible

# Test user restrictions
agent-browser --session user open app.com/admin
agent-browser wait --text "Access Denied"
# Verify user cannot access admin
```

### OAuth/SSO Flow

```bash
# Start OAuth flow
agent-browser open https://app.example.com/login

agent-browser snapshot -i --json
agent-browser click @e2  # "Sign in with Google" button

# Wait for OAuth provider
agent-browser wait --url "**accounts.google.com**"

# Fill OAuth credentials
agent-browser snapshot -i --json
agent-browser fill @e1 "user@gmail.com"
agent-browser click @e2  # Next

agent-browser wait 2000
agent-browser snapshot -i --json
agent-browser fill @e1 "password"
agent-browser click @e2  # Sign in

# Wait for redirect back
agent-browser wait --url "**app.example.com**"
agent-browser wait --url "**/dashboard"
```

---

## Web Scraping & Content Extraction

### Single Page Scraping

```bash
# Navigate to content
agent-browser open https://docs.example.com/api/reference

# Get structured snapshot
agent-browser snapshot -i -s "article" --json
# Parse to identify content elements

# Extract title
agent-browser get text @e1 --json

# Extract description
agent-browser get text @e2 --json

# Extract code examples
agent-browser get html @e3 --json

# Take screenshot for reference
agent-browser screenshot api-reference.png
```

### Multi-Page Scraping

```bash
# Start at index
agent-browser open https://docs.example.com/guides

agent-browser snapshot -i --json
# Parse to find all guide links (@e1, @e2, @e3...)

# Scrape first guide
agent-browser click @e1
agent-browser wait --load domcontentloaded

agent-browser snapshot -i -s "article" --json
agent-browser get text @e5 --json  # Title
agent-browser get html @e6 --json  # Content

# Go back and scrape next
agent-browser back
agent-browser wait 1000

agent-browser click @e2
# Repeat extraction...
```

### Table Data Extraction

```bash
agent-browser open https://app.example.com/reports/data

# Get table snapshot
agent-browser snapshot -i -s "table" --json

# Extract table headers
agent-browser get text @e1 --json  # Header 1
agent-browser get text @e2 --json  # Header 2

# Extract row data
agent-browser get text @e5 --json  # Row 1, Col 1
agent-browser get text @e6 --json  # Row 1, Col 2

# Or get entire table HTML and parse
agent-browser get html @e0 --json
```

---

## Visual Regression Testing

### Baseline Capture

```bash
# Capture baseline screenshots
agent-browser open https://myapp.com/home
agent-browser screenshot baselines/home.png --full

agent-browser open https://myapp.com/dashboard
agent-browser screenshot baselines/dashboard.png --full

agent-browser open https://myapp.com/settings
agent-browser screenshot baselines/settings.png --full
```

### Regression Testing

```bash
# After deployment, capture new screenshots
agent-browser open https://myapp.com/home
agent-browser screenshot current/home.png --full

agent-browser open https://myapp.com/dashboard
agent-browser screenshot current/dashboard.png --full

# AI agent can now compare:
# - baselines/home.png vs current/home.png
# - baselines/dashboard.png vs current/dashboard.png
# Report any visual differences
```

### Component-Specific Regression

```bash
# Test specific component states
agent-browser open https://myapp.com/components/button

# Normal state
agent-browser screenshot states/button-normal.png

# Hover state
agent-browser hover @e1
agent-browser screenshot states/button-hover.png

# Disabled state
agent-browser eval "document.querySelector('button').disabled = true"
agent-browser screenshot states/button-disabled.png

# Compare with baselines
```

---

## Dynamic Content Handling

### Waiting for Dynamic Content

```bash
# Open page with lazy-loaded content
agent-browser open https://app.example.com/dashboard

# Wait for specific element to appear
agent-browser wait --text "Welcome back"

# Or wait for element by selector
agent-browser wait ".dashboard-loaded"

# Or wait for network idle
agent-browser wait --load networkidle

# Then take snapshot
agent-browser snapshot -i --json
```

### Infinite Scroll

```bash
# Open page with infinite scroll
agent-browser open https://app.example.com/feed

# Initial snapshot
agent-browser snapshot -i --json
# Extract items...

# Scroll down to trigger load
agent-browser scroll down 1000
agent-browser wait 2000  # Wait for load

# Get new snapshot (includes new items)
agent-browser snapshot -i --json

# Continue scrolling
agent-browser scroll down 1000
agent-browser wait 2000
agent-browser snapshot -i --json
```

### AJAX Form Submission

```bash
# Fill form
agent-browser snapshot -i --json
agent-browser fill @e1 "data"
agent-browser click @e2  # Submit (AJAX)

# Don't wait for URL change (AJAX doesn't navigate)
# Instead wait for success indicator
agent-browser wait --text "Success"

# Or wait for element to appear
agent-browser wait ".success-message"

# Verify result
agent-browser snapshot -i --json
agent-browser get text @e5 --json
```

---

## File Upload & Download

### File Upload

```bash
agent-browser open https://app.example.com/upload

agent-browser snapshot -i --json

# Upload single file
agent-browser upload @e1 "/path/to/document.pdf"

# Upload multiple files
agent-browser upload @e1 "/path/to/file1.pdf,/path/to/file2.jpg"

# Submit upload form
agent-browser click @e2

# Wait for upload complete
agent-browser wait --text "Upload successful"
```

### File Download Verification

```bash
# Trigger download
agent-browser open https://app.example.com/reports
agent-browser snapshot -i --json
agent-browser click @e3  # "Download Report" button

# Wait for download to start
agent-browser wait 2000

# Note: Files download to system default directory
# Agent can verify file exists using bash tool
```

---

## Multiple Account Management

### Parallel Session Testing

```bash
# User 1: Create content
agent-browser --session user1 open app.com
agent-browser --session user1 snapshot -i --json
agent-browser --session user1 fill @e1 "New Post"
agent-browser --session user1 click @e2

# User 2: View content
agent-browser --session user2 open app.com
agent-browser --session user2 snapshot -i --json
agent-browser --session user2 wait --text "New Post"

# Verify User 2 sees User 1's content
agent-browser --session user2 get text @e5 --json
```

### Role-Based Access Testing

```bash
# Admin: Access all features
agent-browser --session admin --profile ~/.profiles/admin open app.com
agent-browser --session admin open app.com/admin
agent-browser --session admin snapshot -i --json
# Verify admin panel visible

# Editor: Limited access
agent-browser --session editor --profile ~/.profiles/editor open app.com
agent-browser --session editor open app.com/admin
agent-browser --session editor wait --text "Forbidden"

# Viewer: Read-only
agent-browser --session viewer --profile ~/.profiles/viewer open app.com
agent-browser --session viewer snapshot -i --json
# Verify no edit buttons present
```

---

## Error Recovery

### Retry on Element Not Found

```bash
# Attempt interaction
agent-browser click @e5 --json

# Check success field
# If {"success": false, "error": "Element not found"}:

# Re-snapshot to get fresh refs
agent-browser snapshot -i --json

# Retry with new ref
agent-browser click @e5 --json
```

### Handling Timeouts

```bash
# Attempt with timeout
agent-browser wait @e3 --timeout 5000 --json

# If timeout:
# {"success": false, "error": "Timeout waiting for element"}

# Try different strategy
agent-browser wait --text "Loading" --timeout 10000
agent-browser wait --load networkidle
agent-browser snapshot -i --json
```

### Handling Popup Dialogs

```bash
# Trigger action that shows confirm dialog
agent-browser click @e2

# Accept dialog
agent-browser dialog accept

# Or dismiss
agent-browser dialog dismiss

# Or provide input (for prompt dialogs)
agent-browser dialog accept "input text"
```

---

## Performance Testing

### Page Load Timing

```bash
# Navigate and measure
agent-browser open https://app.example.com

# Get performance metrics via eval
agent-browser eval "JSON.stringify(performance.timing)" --json

# Or specific metrics
agent-browser eval "performance.timing.loadEventEnd - performance.timing.navigationStart" --json
```

### Network Monitoring

```bash
# Track network requests
agent-browser open https://app.example.com
agent-browser network requests --json

# Filter specific requests
agent-browser network requests --filter "api" --json

# Check for slow requests, failed requests, etc.
```

### Resource Analysis

```bash
# Check page resources
agent-browser eval "document.images.length" --json
agent-browser eval "document.scripts.length" --json
agent-browser eval "document.styleSheets.length" --json

# Take full page screenshot to verify rendering
agent-browser screenshot page.png --full
```

---

## Best Practices Summary

### Token Efficiency

1. **Always use `-i` flag** for snapshots (interactive elements only)
2. **Combine filters**: `snapshot -i -c -d 5` for maximum reduction
3. **Scope selectors**: `snapshot -i -s "#main"` when possible
4. **Re-snapshot sparingly**: Only after page changes

### Reliability

1. **Use refs over CSS selectors**: Deterministic and AI-friendly
2. **Wait appropriately**: Use `wait --text` or `wait --url` instead of arbitrary timeouts
3. **Verify state**: Check `success` field in JSON responses
4. **Handle errors gracefully**: Re-snapshot and retry on element not found

### Performance

1. **Reuse sessions**: Daemon persists between commands
2. **Use profiles**: Avoid repeated logins
3. **Batch operations**: Script multiple commands together
4. **Minimize headed mode**: Only for debugging

### Debugging

1. **Start with --headed**: See what's happening visually
2. **Check console**: `agent-browser console --json` for JS errors
3. **Use screenshots**: Capture state at each step
4. **Enable trace**: `agent-browser trace start` for deep debugging

---

## Next Steps

- Review [Browser Automation Guide](browser-automation-guide.md) for command reference
- Check [Troubleshooting](../docs/TROUBLESHOOTING.md) for common issues
- Explore [agent-browser GitHub](https://github.com/vercel-labs/agent-browser) for updates
