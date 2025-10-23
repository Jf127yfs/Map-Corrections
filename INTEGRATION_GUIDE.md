# Integration Guide: Multi-Page Google Apps Script Framework

## Overview
This guide explains how to integrate the Map Visualization into a larger Google Apps Script web application with multiple pages using URL parameter routing.

---

## Key Changes Made

### 1. **Multi-Page Routing System**
- Modified `doGet()` to handle URL parameters (`?page=MD`)
- Added support for multiple pages in a single web app
- Centralized navigation through query parameters

### 2. **Data Source Configuration**
- Changed sheet reference from `"Form Responses (Clean)"` to `"FRC"`
- Data now pulled from Column F: "Current 5 Digit Zip Code"
- Configurable via `CONFIG` object in Code.gs

### 3. **Auto-Start Routing**
- Line tracing now starts automatically after data loads
- Configurable delay (default: 1.5 seconds)
- Can be disabled via `APP_CONFIG.AUTO_START_ROUTING = false`

---

## File Structure

```
Your Google Apps Script Project/
│
├── Code.gs                 # Main backend logic with router
├── Index.html              # Home/dashboard page
├── MapDisplay.html         # Map visualization page
├── Dashboard.html          # (Optional) Your dashboard page
├── Reports.html            # (Optional) Your reports page
└── Settings.html           # (Optional) Your settings page
```

---

## Step-by-Step Integration

### Step 1: Update Code.gs

Replace your current `Code.gs` with `Code.gs.updated`:

```javascript
// Key sections:
// 1. doGet() router function (lines 8-32)
// 2. CONFIG object (lines 43-51)
// 3. Updated getZipCodesFromSheet() (lines 58-89)
```

**Important Configuration Values:**
```javascript
const CONFIG = {
  SHEET_NAME: 'FRC',                    // Your sheet name
  ZIP_COLUMN: 6,                        // Column F (1-indexed)
  TARGET_ZIP: '64110',                  // Target zip code
  TARGET_ADDRESS: '5317 Charlotte St, Kansas City, MO 64110',
  TARGET_DISPLAY_NAME: '5317 Charlotte',
  GEOCODE_DELAY_MS: 100                 // Delay between API calls
};
```

### Step 2: Update MapDisplay.html

Replace your current HTML file with `MapDisplay.html.updated`:

**Key Feature - Auto-Start Configuration:**
```javascript
const APP_CONFIG = {
  AUTO_START_ROUTING: true,      // Auto-start routing
  AUTO_START_DELAY_MS: 1500      // Delay before starting
};
```

**To disable auto-start:**
```javascript
const APP_CONFIG = {
  AUTO_START_ROUTING: false,     // User must click button
  AUTO_START_DELAY_MS: 1500
};
```

### Step 3: Create Index.html (Optional)

Use `Index.html.example` as a template for your main navigation page:
- Provides navigation to all pages
- Shows system info and quick links
- Maintains consistent theme with Map Display

### Step 4: Update Existing Pages

If you have existing pages, update their navigation links:

```html
<!-- Navigation menu in your existing pages -->
<nav>
  <a href="?page=home">Home</a>
  <a href="?page=MD">Map Visualization</a>
  <a href="?page=dashboard">Dashboard</a>
  <a href="?page=reports">Reports</a>
</nav>
```

### Step 5: Deploy

1. In Google Apps Script Editor:
   - Click **Deploy** → **New Deployment**
   - Choose **Web app**
   - Set "Execute as" to **Me**
   - Set "Who has access" to your preference
   - Click **Deploy**

2. Copy the deployment URL
3. Access your pages:
   - Home: `https://script.google.com/macros/s/YOUR_ID/exec`
   - Map: `https://script.google.com/macros/s/YOUR_ID/exec?page=MD`

---

## URL Parameter Routing

### How It Works

The `doGet(e)` function receives URL parameters in the `e.parameter` object:

```javascript
function doGet(e) {
  const page = e.parameter.page || 'home';  // Default to 'home'

  switch(page.toLowerCase()) {
    case 'md':
      return HtmlService.createHtmlOutputFromFile('MapDisplay');
    case 'home':
      return HtmlService.createHtmlOutputFromFile('Index');
    // Add more cases...
  }
}
```

### Adding New Pages

1. Create new HTML file (e.g., `Dashboard.html`)
2. Add case to `doGet()` switch statement:

```javascript
case 'dashboard':
  return HtmlService.createHtmlOutputFromFile('Dashboard')
    .setTitle('Dashboard');
```

3. Add navigation link in your pages:
```html
<a href="?page=dashboard">Dashboard</a>
```

---

## Configuration Options

### Backend Configuration (Code.gs)

```javascript
const CONFIG = {
  SHEET_NAME: 'FRC',              // Sheet containing zip codes
  ZIP_COLUMN: 6,                  // Column number (1-indexed)
  TARGET_ZIP: '64110',            // Zip code to highlight
  TARGET_ADDRESS: '5317 Charlotte St, Kansas City, MO 64110',
  TARGET_DISPLAY_NAME: '5317 Charlotte',
  GEOCODE_DELAY_MS: 100           // API rate limiting delay
};
```

### Frontend Configuration (MapDisplay.html)

```javascript
const APP_CONFIG = {
  AUTO_START_ROUTING: true,       // Auto-start route tracing
  AUTO_START_DELAY_MS: 1500       // Delay before auto-start
};

const KC_BOUNDS = {
  minLat: 38.8,                   // Metro area boundaries
  maxLat: 39.4,
  minLng: -94.9,
  maxLng: -94.3
};
```

---

## Data Source Setup

### Sheet Structure (FRC Sheet)

Ensure your "FRC" sheet has:
- Column F: "Current 5 Digit Zip Code"
- Valid 5-digit zip codes (strings)
- Header row in Row 1
- Data starting in Row 2

Example:
```
| A      | B      | ... | F (Current 5 Digit Zip Code) |
|--------|--------|-----|------------------------------|
| Header | Header | ... | Current 5 Digit Zip Code     |
| Data   | Data   | ... | 64110                        |
| Data   | Data   | ... | 64111                        |
```

### Data Validation

The system automatically:
- Trims whitespace
- Validates 5-digit format
- Ignores empty/invalid entries
- Counts duplicate zip codes

---

## Navigation Between Pages

### From Map Display to Other Pages

Add navigation header to MapDisplay.html:

```html
<!-- Add after opening <body> tag -->
<div class="nav-bar">
  <a href="?page=home">← Back to Home</a>
  <a href="?page=dashboard">Dashboard</a>
  <a href="?page=reports">Reports</a>
</div>
```

### JavaScript Navigation

```javascript
// Navigate programmatically
function goToPage(pageName) {
  window.location.href = '?page=' + pageName;
}

// Use in onclick handlers
<button onclick="goToPage('MD')">View Map</button>
```

---

## Advanced Integration

### Passing Data Between Pages

Use URL parameters to pass data:

```javascript
// From one page to Map Display
window.location.href = '?page=MD&targetZip=64110&autoStart=true';

// In doGet(), access parameters
function doGet(e) {
  const page = e.parameter.page;
  const targetZip = e.parameter.targetZip;
  const autoStart = e.parameter.autoStart === 'true';

  // Pass to HTML template
  const template = HtmlService.createTemplateFromFile('MapDisplay');
  template.targetZip = targetZip;
  template.autoStart = autoStart;
  return template.evaluate();
}
```

### Shared Functions Across Pages

Create a `Common.html` file with shared JavaScript:

```html
<!-- Common.html -->
<script>
  function navigateTo(page) {
    window.location.href = '?page=' + page;
  }

  function getQueryParam(param) {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(param);
  }
</script>
```

Include in other pages:
```html
<?!= HtmlService.createHtmlOutputFromFile('Common').getContent(); ?>
```

### Caching Geocoded Results

Add caching to prevent repeated API calls:

```javascript
// In Code.gs
function getZipCodeCoordinates(zipCode) {
  const cache = CacheService.getScriptCache();
  const cached = cache.get('zip_' + zipCode);

  if (cached) {
    return JSON.parse(cached);
  }

  // ... existing geocoding code ...

  if (result) {
    cache.put('zip_' + zipCode, JSON.stringify(result), 21600); // 6 hours
  }

  return result;
}
```

---

## Troubleshooting

### Common Issues

**1. "Sheet not found" error**
- Verify `CONFIG.SHEET_NAME` matches your sheet name exactly
- Check for extra spaces in sheet name

**2. "No zip codes found" error**
- Verify data is in Column F
- Check that zip codes are in rows 2 and below
- Ensure zip codes are 5 digits

**3. Map doesn't auto-start routing**
- Check `APP_CONFIG.AUTO_START_ROUTING` is `true`
- Verify metro zip codes exist within KC_BOUNDS
- Check browser console for JavaScript errors

**4. Page routing not working**
- Verify you're using `?page=MD` (case insensitive)
- Check doGet() includes the page in switch statement
- Ensure HTML file name matches exactly

**5. Geocoding errors**
- Maps API quota exceeded: increase `GEOCODE_DELAY_MS`
- Invalid addresses: check formatting
- Too many zip codes: consider caching (see Advanced section)

---

## Testing Checklist

- [ ] Home page loads at base URL
- [ ] Map page loads at `?page=MD`
- [ ] Data pulls from "FRC" sheet, Column F
- [ ] Auto-routing starts after delay
- [ ] Navigation links work between pages
- [ ] Target location displays correctly
- [ ] Metro/non-metro zips categorized correctly
- [ ] Route network draws along roads
- [ ] Error messages display properly
- [ ] Log section shows activity

---

## Performance Optimization

### For Large Datasets

1. **Implement Geocoding Cache** (see Advanced section)
2. **Batch Geocoding**: Process in chunks
3. **Lazy Loading**: Load non-metro zips on demand
4. **Rate Limiting**: Increase delays if hitting API limits

### For Better User Experience

1. **Progressive Enhancement**: Show markers first, routes later
2. **Cancel Button**: Allow users to stop routing mid-process
3. **Progress Bar**: Visual feedback during data loading
4. **Responsive Design**: Optimize for mobile devices

---

## Next Steps

1. **Replace existing files** with `.updated` versions
2. **Update CONFIG values** for your project
3. **Test routing** with `?page=MD` parameter
4. **Customize styling** to match your project theme
5. **Add additional pages** to your application
6. **Implement caching** for production use
7. **Monitor API usage** to avoid quota issues

---

## Support Resources

- [Google Apps Script Documentation](https://developers.google.com/apps-script)
- [Leaflet.js Documentation](https://leafletjs.com/)
- [OSRM Routing API](http://project-osrm.org/)
- [Google Maps Geocoding API](https://developers.google.com/maps/documentation/geocoding)

---

## Contact

For issues specific to this integration, refer to your project documentation or contact your development team.
