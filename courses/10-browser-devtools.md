# Course 10: Browser DevTools for QA

## 📖 What You Will Learn

- How to open and navigate Browser DevTools
- Using the Network tab to inspect API calls
- Using the Console to find JavaScript errors
- Using the Elements tab to inspect HTML/CSS
- Using the Application tab to inspect storage (cookies, localStorage)
- Practical QA debugging workflows

---

## 1. Opening DevTools

| Method | Shortcut |
|--------|----------|
| **Keyboard (Mac)** | `Cmd + Option + I` |
| **Keyboard (Windows/Linux)** | `F12` or `Ctrl + Shift + I` |
| **Right-click** | Right-click any element → "Inspect" |
| **Menu** | Chrome → More Tools → Developer Tools |

### DevTools Tabs Overview:

| Tab | What it does | QA use |
|-----|-------------|--------|
| **Elements** | Inspect HTML/CSS | Check UI structure, styles |
| **Console** | JavaScript logs & errors | Find JS errors |
| **Network** | All HTTP requests/responses | Inspect API calls ⭐ |
| **Application** | Storage, cookies, cache | Check auth tokens, data |
| **Performance** | Page load & runtime performance | Find slow pages |
| **Sources** | JavaScript source code | Debug scripts |

---

## 2. Network Tab (Most Important for QA)

The Network tab shows **every HTTP request** the page makes.

### How to use it:

1. Open DevTools → **Network** tab
2. **Reload the page** (requests only appear after the tab is open)
3. Interact with the application — every API call appears in the list

### Reading a Network Request:

```
┌─────────────────────────────────────────────────────────────┐
│ Network                                    🔴 Recording     │
├──────┬──────────┬────────┬──────┬──────────┬───────────────┤
│Status│ Name     │ Method │ Type │ Size     │ Time          │
├──────┼──────────┼────────┼──────┼──────────┼───────────────┤
│ 200  │ /api/user│ GET    │ json │ 1.2 KB   │ 45 ms         │
│ 201  │ /api/task│ POST   │ json │ 0.5 KB   │ 120 ms        │
│ 404  │ /api/old │ GET    │ json │ 0.1 KB   │ 23 ms    ❌   │
│ 500  │ /api/save│ POST   │ json │ 0.2 KB   │ 5000 ms  ❌   │
└──────┴──────────┴────────┴──────┴──────────┴───────────────┘
```

### Inspecting a Single Request:

Click on any request to see details:

**Headers tab:**
```
Request URL: https://api.example.com/api/users/42
Request Method: GET
Status Code: 200 OK

Request Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
  Content-Type: application/json
  Accept: application/json

Response Headers:
  Content-Type: application/json
  X-Request-Id: abc-123-def
```

**Response tab (Preview):**
```json
{
  "id": 42,
  "name": "Nastya",
  "email": "nastya@example.com",
  "role": "qa"
}
```

**Payload tab (for POST/PUT requests):**
```json
{
  "name": "Nastya",
  "email": "nastya@example.com"
}
```

### Network Tab Filters:

| Filter | Shows |
|--------|-------|
| **All** | Everything |
| **Fetch/XHR** | API calls only (most useful for QA ⭐) |
| **JS** | JavaScript files |
| **CSS** | Stylesheets |
| **Img** | Images |
| **Doc** | HTML documents |

### QA Tips for Network Tab:

- 🔴 **Red requests** = errors (4xx, 5xx status codes)
- 🕐 **Slow requests** = performance issues (check Time column)
- 🔍 **Filter by "Fetch/XHR"** to see only API calls
- 📋 **Right-click → Copy as cURL** to reproduce a request in terminal
- 🚫 **Check for failed requests** that the UI might silently ignore

---

## 3. Console Tab

Shows **JavaScript logs, warnings, and errors**.

### Common messages:

```
// Error (red) — something is broken
❌ Uncaught TypeError: Cannot read property 'name' of undefined
   at UserProfile.js:42

// Warning (yellow) — potential issue
⚠️ React Warning: Each child in a list should have a unique "key" prop

// Info (blue) — informational
ℹ️ API response received in 234ms

// Log (white) — debug output
📝 User logged in: nastya@example.com
```

### QA Tips for Console:

- **Red errors** = JavaScript bugs — report them!
- **Note the file and line number** (e.g., `UserProfile.js:42`)
- **Check console after reproducing a bug** — errors often appear here
- **CORS errors** appear here when API calls are blocked:
  ```
  ❌ Access to fetch at 'https://api.example.com' from origin
     'https://app.example.com' has been blocked by CORS policy
  ```

---

## 4. Elements Tab

Inspect and modify **HTML structure and CSS styles** live.

### How to use:

1. **Right-click any element** → "Inspect"
2. The Elements tab highlights the HTML element
3. Right panel shows CSS styles applied to it

### What QA can do:

| Action | How | Why |
|--------|-----|-----|
| **Check element text** | Click element in HTML tree | Verify displayed text |
| **Check CSS** | Look at Styles panel | Verify colors, fonts, spacing |
| **Edit text live** | Double-click text in HTML | Test long text, edge cases |
| **Hide elements** | Press `H` on selected element | Test layout without element |
| **Check responsiveness** | Click 📱 device toolbar | Test mobile layouts |

### Device Toolbar (Responsive Testing):

1. Click the **device icon** 📱 (or `Cmd+Shift+M` / `Ctrl+Shift+M`)
2. Select a device preset (iPhone, iPad, etc.) or enter custom dimensions
3. Test how the page looks on different screen sizes

---

## 5. Application Tab

Inspect **cookies, localStorage, sessionStorage, and cache**.

### Cookies:

```
┌──────────────────────────────────────────────────────┐
│ Cookies > https://example.com                         │
├──────────────┬──────────────────┬─────────┬──────────┤
│ Name         │ Value            │ Expires │ HttpOnly │
├──────────────┼──────────────────┼─────────┼──────────┤
│ session_id   │ abc123def456     │ Session │ ✅       │
│ auth_token   │ eyJhbGciOi...    │ 1 hour  │ ✅       │
│ user_pref    │ dark_mode=true   │ 1 year  │ ❌       │
│ _ga          │ GA1.2.123456     │ 2 years │ ❌       │
└──────────────┴──────────────────┴─────────┴──────────┘
```

### localStorage:

```
┌──────────────────────────────────────────┐
│ Local Storage > https://example.com       │
├──────────────────┬───────────────────────┤
│ Key              │ Value                 │
├──────────────────┼───────────────────────┤
│ user_settings    │ {"theme":"dark"}      │
│ recent_searches  │ ["api","test","bug"]  │
│ cart_items       │ [{"id":1,"qty":2}]    │
└──────────────────┴───────────────────────┘
```

### QA Tips for Application Tab:

- 🍪 **Delete cookies** to test logout / session expiry behavior
- 🗑️ **Clear localStorage** to test fresh user experience
- 🔑 **Check auth tokens** — are they present? Are they expired?
- 📦 **Check cached data** — is stale data causing issues?

---

## 6. QA Debugging Workflows

### Workflow 1: "The button doesn't work"

```
1. Open DevTools → Console tab
2. Click the broken button
3. Check for JavaScript errors (red messages)
4. Switch to Network tab → click button again
5. Check if an API request was sent
6. If yes: check the response status and body
7. If no: it's a frontend JavaScript issue
```

### Workflow 2: "The page shows wrong data"

```
1. Open DevTools → Network tab → filter "Fetch/XHR"
2. Reload the page
3. Find the API call that loads the data
4. Check the response body — is the data correct from the API?
5. If API data is correct → frontend display bug
6. If API data is wrong → backend bug
```

### Workflow 3: "The page is slow"

```
1. Open DevTools → Network tab
2. Reload the page
3. Look at the bottom bar: "X requests, X MB transferred, Finish: X s"
4. Sort by Time column — find the slowest requests
5. Check if any request takes > 2 seconds
6. Check for large responses (Size column)
7. Check for too many requests (request count)
```

### Workflow 4: "I can't log in"

```
1. Open DevTools → Network tab
2. Try to log in
3. Find the login API request (POST /api/auth/login)
4. Check request payload — are credentials sent correctly?
5. Check response — what error does the API return?
6. Check Application tab → Cookies — is a session cookie set?
7. Check Console — any CORS or security errors?
```

---

## 📝 Exercises

### Exercise 1: Explore Network Tab

On any website:
1. Open DevTools → Network tab → filter "Fetch/XHR"
2. Navigate around the site
3. Find at least 3 API calls and for each note:
   - URL, Method, Status Code
   - Response body content

### Exercise 2: Find Information

Using DevTools, find:
1. All cookies set by the website
2. Any JavaScript errors in the Console
3. The CSS color of the main heading
4. Whether the site is responsive (test with device toolbar)

### Exercise 3: Debug a Scenario

Describe the DevTools steps you would take to debug:
1. A form submission that shows "Error" with no details
2. An image that doesn't load
3. A page that takes 10 seconds to load
4. A user who gets logged out unexpectedly

---

## 📚 Additional Resources

- [Chrome DevTools Docs](https://developer.chrome.com/docs/devtools/) — Official documentation
- [Firefox DevTools](https://firefox-source-docs.mozilla.org/devtools-user/) — Firefox alternative
- [Network Analysis Reference](https://developer.chrome.com/docs/devtools/network/reference/) — Detailed network tab guide

---

## ✅ Self-Check

- [ ] Open DevTools and navigate between tabs
- [ ] Use the Network tab to inspect API requests and responses
- [ ] Filter network requests to show only API calls (Fetch/XHR)
- [ ] Find JavaScript errors in the Console
- [ ] Inspect an HTML element and view its CSS styles
- [ ] Use the device toolbar to test responsive layouts
- [ ] Check cookies and localStorage in the Application tab
- [ ] Copy an API request as cURL from the Network tab
- [ ] Follow a debugging workflow to identify frontend vs backend bugs