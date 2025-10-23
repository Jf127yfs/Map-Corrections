# Quick Start Guide

## What Changed

✅ **Multi-Page Routing**: Access map via `?page=MD`
✅ **Auto-Start Routing**: Line tracing begins automatically
✅ **Updated Data Source**: Now pulls from sheet "FRC", Column F
✅ **Configurable**: Easy-to-modify CONFIG objects

---

## Immediate Actions

### 1. Replace Files in Google Apps Script

**Replace Code.gs:**
```
Use: Code.gs.updated
Key Changes:
- Multi-page router in doGet()
- CONFIG object with SHEET_NAME='FRC', ZIP_COLUMN=6
- Better error handling
```

**Replace MapDisplay HTML:**
```
Use: MapDisplay.html.updated
Key Changes:
- AUTO_START_ROUTING: true
- AUTO_START_DELAY_MS: 1500
- Fixed XSS vulnerability with DOM methods
```

### 2. Test the Integration

1. Deploy as Web App
2. Access base URL → goes to home/index
3. Access `?page=MD` → goes to map
4. Verify map loads data from "FRC" sheet
5. Confirm routing starts automatically after 1.5 seconds

### 3. Customize (Optional)

**Change Target Location (Code.gs):**
```javascript
const CONFIG = {
  TARGET_ZIP: '64110',  // Change this
  TARGET_ADDRESS: '5317 Charlotte St, Kansas City, MO 64110',  // Change this
  TARGET_DISPLAY_NAME: '5317 Charlotte'  // Change this
};
```

**Disable Auto-Start (MapDisplay.html):**
```javascript
const APP_CONFIG = {
  AUTO_START_ROUTING: false,  // Set to false
};
```

---

## File Summary

| File | Purpose | Status |
|------|---------|--------|
| Code.gs.updated | Backend with routing | ✅ Ready |
| MapDisplay.html.updated | Map page with auto-start | ✅ Ready |
| Index.html.example | Home page template | 📋 Template |
| INTEGRATION_GUIDE.md | Full documentation | 📖 Reference |
| QUICK_START.md | This file | 🚀 Start here |

---

## URLs After Deployment

```
Base URL:          https://script.google.com/.../exec
Home Page:         https://script.google.com/.../exec?page=home
Map Display:       https://script.google.com/.../exec?page=MD
```

---

## Configuration Quick Reference

### Backend (Code.gs)
```javascript
SHEET_NAME: 'FRC'              // Your data sheet
ZIP_COLUMN: 6                  // Column F (1-indexed)
TARGET_ZIP: '64110'            // Highlight this zip
GEOCODE_DELAY_MS: 100          // API rate limit delay
```

### Frontend (MapDisplay.html)
```javascript
AUTO_START_ROUTING: true       // Auto-trace routes
AUTO_START_DELAY_MS: 1500      // Wait 1.5s before start
```

---

## Verification Checklist

- [ ] Map loads at `?page=MD`
- [ ] Data pulled from "FRC" sheet
- [ ] Zip codes from Column F displayed
- [ ] Target marker shows "5317 Charlotte"
- [ ] Routing starts automatically
- [ ] Routes follow roads (not straight lines)
- [ ] Statistics panel shows correct counts
- [ ] No console errors

---

## Next Steps

1. ✅ Review this Quick Start
2. 📖 Read INTEGRATION_GUIDE.md for details
3. 🔧 Update files in Apps Script
4. 🚀 Deploy and test
5. 🎨 Customize as needed

---

## Support

For detailed integration instructions, troubleshooting, and advanced features, see **INTEGRATION_GUIDE.md**.
