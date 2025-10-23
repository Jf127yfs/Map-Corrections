# Implementation Guide: Multi-Mode Framework

## Overview

This framework provides **3 access modes** for your Google Apps Script web application:

1. **Master Mode** - Frame-based navigation with persistent sidebar
2. **Display Mode** - Auto-rotating full-screen display for TV/dashboards
3. **Direct Page Access** - Legacy direct links to specific pages

---

## Quick Start

### 1. File Setup in Google Apps Script

Add these files to your project:

| File | Purpose | Required |
|------|---------|----------|
| Code.gs.v2 | Router with mode support | ✅ Yes |
| Master.html | Frame-based navigation | ✅ Yes |
| Display.html | Auto-rotation display | ✅ Yes |
| MapDisplay.html.updated | Map page | ✅ Yes |
| Index.html.example | Home page | ⚠️ Recommended |

**File Naming in Apps Script:**
- Rename `Code.gs.v2` → `Code.gs` (replace existing)
- Rename `MapDisplay.html.updated` → `MapDisplay.html` (replace existing)
- Remove `.example` from Index.html (or create your own)

### 2. Deploy as Web App

1. In Apps Script Editor → **Deploy** → **New Deployment**
2. Type: **Web app**
3. Execute as: **Me**
4. Who has access: Choose your preference
5. Click **Deploy**
6. Copy the deployment URL

### 3. Access Your App

```
Master Mode (Default):
https://script.google.com/.../exec

Master with specific page:
https://script.google.com/.../exec?mode=master&page=MD

Display Mode:
https://script.google.com/.../exec?mode=display

Display with custom settings:
https://script.google.com/.../exec?mode=display&interval=15&pages=MD,dashboard

Direct Page Access (Legacy):
https://script.google.com/.../exec?page=MD
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         ROUTER                              │
│                        (Code.gs)                            │
│  Handles: ?mode=master | ?mode=display | ?page=X           │
└─────────────────────────────────────────────────────────────┘
                              │
                ┌─────────────┼─────────────┐
                │             │             │
                ▼             ▼             ▼
         ┌──────────┐  ┌──────────┐  ┌──────────┐
         │  MASTER  │  │ DISPLAY  │  │  DIRECT  │
         │   MODE   │  │   MODE   │  │   PAGE   │
         └──────────┘  └──────────┘  └──────────┘
                │             │             │
                │             │             │
         ┌──────────┐  ┌──────────┐  ┌──────────┐
         │ Sidebar  │  │  Auto-   │  │   Full   │
         │    +     │  │  Rotate  │  │   Page   │
         │  IFrame  │  │  IFrame  │  │  Direct  │
         └──────────┘  └──────────┘  └──────────┘
                │             │             │
                └─────────────┴─────────────┘
                              │
                    ┌─────────┴─────────┐
                    │  PAGE COMPONENTS  │
                    │ (MD, Dashboard,   │
                    │  Reports, etc.)   │
                    └───────────────────┘
```

---

## Mode 1: Master Mode (Navigation)

### Features
- ✅ Persistent navigation sidebar
- ✅ Header with controls
- ✅ Content loads in iframe (no page reloads)
- ✅ Breadcrumb navigation
- ✅ Browser back/forward support
- ✅ Mobile responsive

### Usage

**Access Master Mode:**
```
?mode=master                # Default page
?mode=master&page=MD        # Load map
?mode=master&page=dashboard # Load dashboard
```

**Customize Navigation:**

Edit `Master.html` navigation sections:

```html
<div class="nav-section">
  <div class="nav-title">▸ Your Section</div>
  <a class="nav-item" data-page="yourpage" onclick="loadPage('yourpage', 'Your Page')">
    <span class="nav-item-icon">📄</span>
    <span>Your Page</span>
  </a>
</div>
```

**Add New Page to Router:**

Edit `Code.gs`, add to `getPage()` function:

```javascript
case 'yourpage':
  return HtmlService.createHtmlOutputFromFile('YourPage')
    .setTitle('Your Page Title')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
```

### Controls

| Button | Action |
|--------|--------|
| ☰ MENU | Toggle sidebar (mobile) |
| 📺 DISPLAY | Open display mode |
| ↻ REFRESH | Reload current page |
| ↗ OPEN IN TAB | Open current page in new tab |

---

## Mode 2: Display Mode (Auto-Rotation)

### Features
- ✅ Full-screen auto-rotating pages
- ✅ Configurable interval (1-300 seconds)
- ✅ Custom page rotation list
- ✅ Play/pause controls
- ✅ Progress bar
- ✅ Keyboard shortcuts
- ✅ Settings panel
- ✅ Fade transitions
- ✅ Loop/no-loop modes

### Usage

**Basic Display Mode:**
```
?mode=display
# Uses defaults: 10s interval, pages: home,MD,dashboard,reports
```

**Custom Interval:**
```
?mode=display&interval=15
# 15-second intervals
```

**Custom Pages:**
```
?mode=display&pages=MD,dashboard,analytics
# Only rotate through these 3 pages
```

**Custom Everything:**
```
?mode=display&interval=20&pages=MD,dashboard&loop=true
# 20s intervals, 2 pages, loop continuously
```

### Controls

| Button | Action | Keyboard |
|--------|--------|----------|
| ◀ PREV | Previous page | ← or P |
| ⏸ PAUSE | Pause rotation | Space |
| NEXT ▶ | Next page | → or N |
| ⚙ SETTINGS | Open settings | S |
| ⛶ FULLSCREEN | Toggle fullscreen | F |
| ✕ EXIT | Exit to master | - |

### Settings Panel

Open with **⚙ SETTINGS** button or press **S**

- **Interval**: 1-300 seconds
- **Pages**: Comma-separated list (e.g., `MD,dashboard,reports`)
- **Loop**: Continuously loop or stop at end
- **Fade Transitions**: Enable/disable fade effect
- **Auto-Hide Controls**: Show controls only on hover

**Click APPLY SETTINGS to save changes**

### Use Cases

**Office TV Dashboard:**
```
?mode=display&interval=30&pages=dashboard,reports,MD
```

**Lobby Information Display:**
```
?mode=display&interval=15&pages=home,MD,analytics
```

**Executive Monitoring:**
```
?mode=display&interval=60&pages=dashboard,reports
```

**Presentation Mode:**
```
?mode=display&interval=120&loop=false&pages=intro,MD,results
```

---

## Mode 3: Direct Page Access (Legacy)

### Features
- ✅ Direct link to specific page
- ✅ Backward compatible
- ✅ Bookmarkable
- ✅ Full page (no frames)

### Usage

```
?page=MD          # Direct to map
?page=dashboard   # Direct to dashboard
?page=reports     # Direct to reports
```

**When to use:**
- Bookmarking specific pages
- Sharing direct links
- Embedding in other sites
- Legacy compatibility

---

## File Descriptions

### Code.gs.v2 (Router)

**Router Logic:**
```javascript
function doGet(e) {
  const params = e.parameter || {};
  const mode = params.mode || '';
  const page = params.page || '';

  if (mode === 'display') return Display.html;
  if (mode === 'master') return Master.html;
  if (page) return getPage(page);
  return Master.html; // Default
}
```

**Key Functions:**
- `doGet(e)` - Main router
- `getPage(pageName)` - Get specific page
- `getAllZipData()` - Fetch data for map
- `getZipCodesFromSheet()` - Read from FRC sheet
- `getConfig()` - Get configuration

### Master.html (Navigation Frame)

**Structure:**
```
┌─────────────────────────────────┐
│          HEADER                 │
│  Title | Menu | Display | Refresh │
├──────────┬──────────────────────┤
│ SIDEBAR  │   CONTENT FRAME      │
│          │                      │
│ Nav      │   <iframe>           │
│ Items    │   Loaded Page        │
│          │   </iframe>          │
│          │                      │
└──────────┴──────────────────────┘
```

**Key Features:**
- Sidebar with navigation items
- Content iframe for page loading
- No full page reloads
- URL history management
- Mobile responsive

### Display.html (Auto-Rotation)

**Structure:**
```
┌─────────────────────────────────┐
│                                 │
│      <iframe>                   │
│      Current Page               │
│      (Full Screen)              │
│      </iframe>                  │
│                                 │
├─────────────────────────────────┤
│  CONTROLS (Auto-hide)           │
│  Prev | Pause | Next | Settings │
└─────────────────────────────────┘
```

**Key Features:**
- Full-screen iframe
- Auto-rotation timer
- Progress bar
- Control panel (auto-hide)
- Settings panel (slide-in)
- Keyboard shortcuts
- Transition effects

---

## Configuration

### Backend Config (Code.gs)

```javascript
const CONFIG = {
  SHEET_NAME: 'FRC',              // Data sheet name
  ZIP_COLUMN: 6,                  // Column F
  TARGET_ZIP: '64110',
  TARGET_ADDRESS: '5317 Charlotte St, Kansas City, MO 64110',
  TARGET_DISPLAY_NAME: '5317 Charlotte',
  GEOCODE_DELAY_MS: 100
};
```

### Frontend Config (MapDisplay.html)

```javascript
const APP_CONFIG = {
  AUTO_START_ROUTING: true,       // Auto-start routing
  AUTO_START_DELAY_MS: 1500       // Delay before start
};
```

### Display Config (Display.html)

```javascript
const config = {
  pages: ['home', 'MD', 'dashboard', 'reports'],
  interval: 10000,  // 10 seconds
  loop: true,
  transition: true,
  autoHide: true
};
```

**These are defaults. Users can override via Settings Panel or URL parameters.**

---

## Adding New Pages

### Step 1: Create HTML File

Create `YourPage.html` in Apps Script:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Your Page</title>
  <style>
    /* Your styles */
  </style>
</head>
<body>
  <h1>Your Page Content</h1>
  <!-- Your content -->
  <script>
    // Your JavaScript
  </script>
</body>
</html>
```

### Step 2: Add to Router

Edit `Code.gs`, add to `getPage()`:

```javascript
case 'yourpage':
  return HtmlService.createHtmlOutputFromFile('YourPage')
    .setTitle('Your Page')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
```

### Step 3: Add to Master Navigation

Edit `Master.html`, add nav item:

```html
<a class="nav-item" data-page="yourpage" onclick="loadPage('yourpage', 'Your Page')">
  <span class="nav-item-icon">📄</span>
  <span>Your Page</span>
</a>
```

### Step 4: (Optional) Add to Display Rotation

Users can add via Settings Panel, or set default in `Display.html`:

```javascript
const config = {
  pages: ['home', 'MD', 'dashboard', 'yourpage'], // Add here
  // ...
};
```

---

## Customization

### Change Theme Colors

Edit CSS variables in any HTML file:

```css
:root {
  --bg: #000;        /* Background */
  --fg: #0f0;        /* Foreground (text/borders) */
  --accent: #ff0;    /* Accent color */
  --panel: #0a0a0a;  /* Panel background */
  --border: #0f0;    /* Border color */
}
```

### Change Default Landing

Edit `Code.gs`, change default in `doGet()`:

```javascript
// Current: Returns Master mode
return HtmlService.createHtmlOutputFromFile('Master');

// Change to: Return Index/Home page
return HtmlService.createHtmlOutputFromFile('Index');

// Or: Redirect to Display mode
return HtmlService.createHtmlOutputFromFile('Display');
```

### Change Display Defaults

Edit `Display.html`, modify config:

```javascript
const config = {
  pages: ['MD', 'dashboard'],  // Only these 2 pages
  interval: 30000,             // 30 seconds
  loop: false,                 // Stop after last page
  transition: false,           // No fade
  autoHide: false              // Always show controls
};
```

### Add Page-Specific Durations

Modify `Display.html` to support different durations per page:

```javascript
const config = {
  pages: [
    { name: 'MD', duration: 30000 },        // 30s for map
    { name: 'dashboard', duration: 15000 }, // 15s for dashboard
    { name: 'reports', duration: 20000 }    // 20s for reports
  ]
};
```

(Requires code modification in `scheduleNext()` function)

---

## Troubleshooting

### Issue: Iframe not loading

**Symptom:** Blank iframe, or "Refused to display" error

**Solution:**
Ensure `setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)` is set in router:

```javascript
.setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
```

### Issue: Master page navigation not updating

**Symptom:** Clicking nav items doesn't load new pages

**Solution:**
Check that page name in `data-page` attribute matches router case:

```html
<!-- Must match -->
<a data-page="MD" onclick="loadPage('MD', 'Map')">

<!-- Router -->
case 'md':  // Case insensitive, but must exist
```

### Issue: Display mode not rotating

**Symptom:** Stays on first page

**Solution:**
1. Check page names are valid
2. Open browser console for errors
3. Verify pages exist in router
4. Try clicking NEXT button manually

### Issue: Map not loading data

**Symptom:** "No zip codes found" error

**Solution:**
1. Verify sheet name is exactly "FRC"
2. Check Column F has 5-digit zip codes
3. Ensure data starts in Row 2 (Row 1 is header)
4. Check Apps Script logs for errors

### Issue: Performance/slow loading

**Solutions:**
1. **Add Geocoding Cache** (see below)
2. Reduce `GEOCODE_DELAY_MS` if not hitting limits
3. Preload pages in display mode
4. Optimize page load times

**Geocoding Cache Example:**

```javascript
function getZipCodeCoordinates(zipCode) {
  const cache = CacheService.getScriptCache();
  const cached = cache.get('zip_' + zipCode);

  if (cached) {
    return JSON.parse(cached);
  }

  // ... existing code ...

  if (result) {
    cache.put('zip_' + zipCode, JSON.stringify(result), 21600); // 6 hours
  }

  return result;
}
```

---

## Best Practices

### For Master Mode
- Keep navigation items under 10 per section
- Use clear, descriptive names
- Group related pages together
- Test on mobile devices

### For Display Mode
- 3-5 pages ideal for rotation
- 10-30 second intervals recommended
- Test on actual TV/display hardware
- Consider room viewing distance when designing
- Use high contrast for readability
- Minimize text, maximize visuals

### For All Modes
- Optimize page load times
- Test with slow connections
- Handle errors gracefully
- Provide user feedback (loading indicators)
- Support keyboard navigation
- Use semantic HTML
- Add ARIA labels for accessibility

---

## Next Steps

1. ✅ **Deploy** the application
2. ✅ **Test** all three modes
3. ✅ **Customize** navigation and pages
4. ✅ **Configure** display settings for your use case
5. ✅ **Optimize** with caching if needed
6. ✅ **Train** users on keyboard shortcuts
7. ✅ **Monitor** usage and performance

---

## Summary

You now have a flexible multi-mode framework:

- **Master Mode** for normal navigation
- **Display Mode** for auto-rotation dashboards
- **Direct Access** for legacy links

All modes share the same page components and data sources, giving you maximum flexibility with minimal duplication.

**Questions? Refer to ARCHITECTURE_GUIDE.md for detailed technical documentation.**
