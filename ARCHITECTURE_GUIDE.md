# Architecture Guide: Multi-Page Framework Design

## Recommended Architecture: Hybrid Approach

Based on your requirements (multi-page navigation + auto-rotation display mode), I recommend a **3-tier hybrid architecture**:

```
┌─────────────────────────────────────────────────────────────┐
│                     TIER 1: Router                          │
│  (Direct access via URL parameters: ?page=X&mode=Y)        │
└─────────────────────────────────────────────────────────────┘
                              ↓
        ┌─────────────────────┴─────────────────────┐
        ↓                                           ↓
┌───────────────────────┐              ┌────────────────────────┐
│   TIER 2: Master Page │              │  TIER 2: Display Mode  │
│   (Frame-based Nav)   │              │  (Auto-rotation)       │
│                       │              │                        │
│  ┌─────────────────┐  │              │  ┌──────────────────┐  │
│  │  Navigation     │  │              │  │  Auto-Rotate     │  │
│  │  Header/Sidebar │  │              │  │  Timer Control   │  │
│  └─────────────────┘  │              │  └──────────────────┘  │
│  ┌─────────────────┐  │              │  ┌──────────────────┐  │
│  │  Content Frame  │  │              │  │  Full Screen     │  │
│  │  (Dynamic)      │  │              │  │  Pages           │  │
│  └─────────────────┘  │              │  └──────────────────┘  │
└───────────────────────┘              └────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                 TIER 3: Page Components                     │
│  (MapDisplay, Dashboard, Reports, Analytics, etc.)         │
└─────────────────────────────────────────────────────────────┘
```

---

## Architecture Options Comparison

| Feature | Simple Router | Frame-based Master | Display Mode | Hybrid (Recommended) |
|---------|--------------|-------------------|--------------|---------------------|
| Direct URL access | ✅ | ✅ | ✅ | ✅ |
| No page reloads | ❌ | ✅ | ✅ | ✅ |
| Shared navigation | ❌ | ✅ | ❌ | ✅ |
| Auto-rotation | ❌ | ⚠️ Complex | ✅ | ✅ |
| Bookmarkable | ✅ | ✅ | ⚠️ Partial | ✅ |
| TV/Dashboard display | ❌ | ⚠️ Possible | ✅ | ✅ |
| Easy to implement | ✅ | ⚠️ Medium | ⚠️ Medium | ⚠️ Complex |
| Performance | Good | Better | Best | Best |

---

## Recommended Structure

### 1. **Router Layer** (Code.gs)
- Handles all URL parameters
- Routes to appropriate mode/page
- Serves as entry point

### 2. **Master Page** (Master.html)
- Navigation sidebar/header
- Content frame for page loading
- Persistent UI elements
- For normal user navigation

### 3. **Display Mode** (Display.html)
- Full-screen auto-rotation
- Timer-based page switching
- Minimal UI, maximum content
- Perfect for TV displays, monitoring

### 4. **Page Components**
- Individual pages (MapDisplay, Dashboard, etc.)
- Can be loaded in Master frame OR Display mode
- Self-contained and modular

---

## URL Routing Strategy

### Direct Page Access (Current)
```
?page=MD                    → Direct map page
?page=dashboard             → Direct dashboard page
```

### Master Page Access (New)
```
?mode=master                → Master with default page
?mode=master&page=MD        → Master with map loaded
?mode=master&page=dashboard → Master with dashboard
```

### Display Mode Access (New)
```
?mode=display               → Auto-rotation display
?mode=display&interval=10   → 10-second rotation
?mode=display&pages=MD,dashboard,reports  → Custom rotation
```

### Legacy Support
```
?page=MD                    → Falls back to direct access
(no params)                 → Goes to home/master
```

---

## Implementation Approaches

### Approach 1: IFrame-based (Recommended for Display Mode)

**Pros:**
- ✅ True isolation between pages
- ✅ Each page can run independently
- ✅ No JavaScript conflicts
- ✅ Easy to implement auto-rotation
- ✅ Can load from different URLs

**Cons:**
- ❌ Slight performance overhead
- ❌ Cannot easily share data between pages
- ❌ Some CORS considerations

**Best for:** Display mode, TV dashboards, kiosks

### Approach 2: Dynamic Content Loading

**Pros:**
- ✅ Better performance
- ✅ Can share state/data between views
- ✅ Smoother transitions
- ✅ More control over lifecycle

**Cons:**
- ❌ More complex implementation
- ❌ Need to manage script loading
- ❌ Potential for conflicts
- ❌ Harder to debug

**Best for:** Master page with navigation

### Approach 3: Hybrid (Recommended)

**Use dynamic loading for Master page:**
- Navigation remains persistent
- Content area updates without reload
- Shared header/footer

**Use iframes for Display mode:**
- Each page fully isolated
- Easy rotation logic
- No interference between pages

---

## File Organization

```
Your Google Apps Script Project/
│
├── Code.gs                 # Router with mode support
│
├── [MASTER MODE FILES]
├── Master.html             # Master page with navigation + content frame
├── Nav.html                # Shared navigation component
├── Header.html             # Shared header component
│
├── [DISPLAY MODE FILES]
├── Display.html            # Auto-rotation display mode
├── DisplayControls.html    # Optional: Control panel for display
│
├── [PAGE COMPONENTS]
├── Index.html              # Home/landing page
├── MapDisplay.html         # Map visualization
├── Dashboard.html          # Analytics dashboard
├── Reports.html            # Reports page
├── Analytics.html          # Additional analytics
│
├── [SHARED RESOURCES]
├── Common.html             # Shared JavaScript functions
└── Styles.html             # Shared CSS styles
```

---

## Data Flow

### Master Mode
```
User → URL (?mode=master&page=MD)
  ↓
Router (Code.gs)
  ↓
Master.html (loads with navigation)
  ↓
Content Frame loads MapDisplay.html
  ↓
User clicks nav → Update frame without reload
```

### Display Mode
```
User → URL (?mode=display&interval=10)
  ↓
Router (Code.gs)
  ↓
Display.html (full screen)
  ↓
Timer triggers every 10 seconds
  ↓
IFrame src changes to next page
  ↓
Cycles through configured pages
```

### Direct Access (Legacy)
```
User → URL (?page=MD)
  ↓
Router (Code.gs)
  ↓
MapDisplay.html directly
```

---

## Recommended Implementation Plan

### Phase 1: Update Router ✅ (Already Complete)
- Current router handles `?page=X`
- Ready for extension

### Phase 2: Add Master Page (Next)
- Create Master.html with frame
- Add navigation component
- Support `?mode=master&page=X`

### Phase 3: Add Display Mode (Next)
- Create Display.html with auto-rotation
- Support `?mode=display&interval=X`
- Add page queue management

### Phase 4: Enhance & Polish
- Add transitions/animations
- Add display controls (pause, skip, speed)
- Add configuration UI

---

## Auto-Rotation Display Features

### Basic Features
- ✅ Timer-based page rotation
- ✅ Configurable interval
- ✅ Full-screen mode
- ✅ Minimal UI

### Advanced Features (Optional)
- ⚡ Pause/resume controls
- ⚡ Skip to next/previous
- ⚡ Speed adjustment
- ⚡ Page-specific durations
- ⚡ Fade transitions
- ⚡ Remote control via URL updates
- ⚡ Schedule-based rotation (different pages at different times)

### Use Cases
- 📺 TV dashboard in office
- 📊 Monitoring displays
- 🎤 Presentation mode
- 🏢 Lobby information display
- 📈 Executive dashboard rotation

---

## Performance Considerations

### For Master Page
- Preload common resources
- Cache navigation state
- Use CSS transitions for smooth page changes
- Minimize reflows

### For Display Mode
- Preload next page in hidden iframe
- Dispose of old iframes properly
- Manage memory with page limits
- Consider pause on no activity

### For Individual Pages
- Optimize load time (defer non-critical JS)
- Clean up on unload (remove listeners, timers)
- Support visibility API (pause when hidden)
- Lazy load heavy components

---

## Security & Best Practices

### IFrame Security
```javascript
// Use sandbox attribute for untrusted content
<iframe sandbox="allow-scripts allow-same-origin"></iframe>

// For trusted internal pages
<iframe src="..." allow="..."></iframe>
```

### Data Sharing Between Pages
```javascript
// Option 1: URL parameters
iframe.src = '?page=MD&data=' + encodeURIComponent(JSON.stringify(data));

// Option 2: postMessage API
iframe.contentWindow.postMessage(data, '*');

// Option 3: Shared storage (PropertiesService)
PropertiesService.getUserProperties().setProperty('key', value);
```

### State Management
```javascript
// In Display mode, maintain state
const displayState = {
  currentPage: 0,
  pages: ['MD', 'dashboard', 'reports'],
  interval: 10000,
  isPaused: false
};
```

---

## Next Steps

1. **Decide on primary use case:**
   - Normal navigation → Prioritize Master page
   - TV display → Prioritize Display mode
   - Both → Implement hybrid

2. **Review provided implementations:**
   - Master.html (coming next)
   - Display.html (coming next)
   - Updated Code.gs router

3. **Test and iterate:**
   - Start with basic functionality
   - Add advanced features as needed
   - Optimize performance

4. **Consider your workflow:**
   - How will users primarily access?
   - What pages need auto-rotation?
   - What rotation intervals make sense?

---

## Questions to Answer

Before implementation, consider:

1. **Primary Access Mode:**
   - Will users mostly use navigation (Master) or direct links?
   - Is display mode for a specific monitor/TV?

2. **Page Rotation:**
   - Which pages should rotate?
   - Same interval for all, or different per page?
   - Should rotation loop or stop at end?

3. **Navigation Style:**
   - Sidebar, top nav, or both?
   - Always visible or collapsible?
   - Breadcrumbs needed?

4. **Display Mode Control:**
   - Should users be able to pause/control?
   - Remote control capability needed?
   - Schedule different rotations for different times?

---

**Continue to next file:** Master.html and Display.html implementations →
