# Lessons Learned Dashboard — Self-Hosting Setup

Your dashboard is now a standalone HTML file with live Smartsheet integration. Here's how to deploy it.

## Quick Start (Local Testing)

1. **Open the file** in any web browser:
   - `pic-dashboard.html`

2. **On first load**, the dashboard will prompt you to enter your Smartsheet API token
   - The token is stored in your browser's local storage (never sent to external servers except Smartsheet)
   - To reset it, open browser DevTools → Application → Local Storage → clear `smartsheet_token`

3. **Your token** comes from:
   - Smartsheet → Account Settings → API Access → Generate Token
   - Copy the token (it will only show once)

## Deploy to Production

### Option A: Simple Hosting (Netlify, Vercel, GitHub Pages)

1. **Download** `pic-dashboard.html`
2. **Upload** to your hosting service:
   - **Netlify**: Drag & drop the file or push to GitHub
   - **Vercel**: Import from GitHub repo with this file
   - **GitHub Pages**: Commit to `docs/` folder and enable Pages in repo settings
3. **Access** your dashboard at your hosted URL
4. **Enter API token** when prompted (first use only)

### Option B: Self-Hosted (Your Own Server)

1. **Place** `pic-dashboard.html` in your web root:
   ```bash
   /var/www/html/dashboard/pic-dashboard.html
   # or wherever your web server serves files
   ```

2. **Access** at: `https://yourdomain.com/dashboard/pic-dashboard.html`

### Option C: Internal PIC Server

Deploy to your internal PIC infrastructure:
- Copy to any internal web server (Apache, Nginx, IIS)
- Access via `https://pic-internal.local/dashboard`
- No external dependencies (all assets are self-contained)

## Security Notes

- **API Token**: Stored locally in your browser. Only transmitted to Smartsheet.com
- **CORS**: If you get a CORS error, your hosting needs to allow requests to `api.smartsheet.com`. This is handled by Smartsheet's CORS policy (should work from any origin).
- **Token Exposure**: Don't share the dashboard URL if you're concerned about token visibility. Better practice: use a backend proxy (see below).

## Advanced: Backend Proxy (More Secure)

For production, use a backend server to hide your API token:

```javascript
// Node.js example (Express)
app.get('/api/sheet-data', async (req, res) => {
  const response = await fetch('https://api.smartsheet.com/2.0/sheets/6140950832172932', {
    headers: {
      'Authorization': `Bearer ${process.env.SMARTSHEET_TOKEN}`
    }
  });
  res.json(await response.json());
});
```

Then update the dashboard to call `fetch('/api/sheet-data')` instead of the Smartsheet API directly.

## Refresh Schedule

The dashboard refreshes hourly automatically. To change:

In `pic-dashboard.html`, find:
```javascript
setInterval(loadDashboard, 3600000); // 3600000 ms = 1 hour
```

Change `3600000` to:
- `1800000` for 30 minutes
- `600000` for 10 minutes
- `300000` for 5 minutes

## Troubleshooting

**"Invalid API token" error**
- Generate a new token from Smartsheet Account Settings
- Clear browser cache and local storage
- Re-enter token when prompted

**"CORS error" or "blocked by CORS policy"**
- This shouldn't happen with Smartsheet's API, but if it does:
- Use a backend proxy (see Advanced section above)
- Check that your browser is accessing over HTTPS (not always required, but recommended)

**Dashboard shows demo data instead of real data**
- Check browser console (F12 → Console) for errors
- Verify API token is correct
- Confirm Smartsheet sheet ID matches: `6140950832172932`

**Charts not loading**
- Chart.js requires JavaScript to be enabled
- Check browser security settings aren't blocking CDN resources

## Customization

Edit the file directly to customize:

- **Colors**: Look for `--stellar: #3C6283` in the `<style>` section and update PIC brand colors
- **Refresh interval**: Change `setInterval(loadDashboard, 3600000)`
- **Demo data**: Modify the `loadDemoData()` function
- **Sheet ID**: Update `SHEET_ID: '6140950832172932'` if you move to a different sheet

## File Structure

```
pic-dashboard.html
├── HTML structure
├── PIC brand CSS (embedded)
├── Chart.js library (from CDN)
├── Google Fonts (Inter, JetBrains Mono)
└── JavaScript with Smartsheet API integration
```

All dependencies are external CDNs (CDNjs for Chart.js, Google Fonts) — no build process needed.

## Support

The dashboard pulls live data from your Smartsheet Request Tracker sheet:
- https://app.smartsheet.com/sheets/gvJcwJ6vWcRxvwW7mmh5vQmmHfpgvvJ7hXrMcjj1

If metrics are wrong, check that your Smartsheet sheet has data in these columns:
- ID (for Request ID)
- Change Needed (Title) (for item titles)
- Phase
- Priority
- Status
- Building
- Root Cause Category
- Is Open, Is Stale, Is Past Due (checkbox formulas)
- Age (Days) (for cycle time)
- Month Submitted, Month Closed (for trend data)

---

Ready to deploy? Your standalone dashboard is production-ready. Pick your hosting option above and you're live.
