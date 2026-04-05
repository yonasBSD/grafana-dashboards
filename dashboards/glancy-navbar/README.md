# Glancy Navigation Bar Panel

<div align="center">

**A team-aware navigation bar for Grafana that auto-discovers dashboards via tags and organizes them into categorized dropdowns.**

</div>

### Screenshot
![Glancy-Navbar](/screenshots/glancy-navbar.png)
### Demo
![Glancy-Navbar](/screenshots/glancy-navbar-demo.gif)

## ✨ Features

- **Team-Based Navigation** — Detects the logged-in user's Grafana team via `/api/user/teams` and renders a navbar tailored to that team. Users without a team get the `_default` config.
- **Tag-Driven Dashboard Discovery** — Dashboards are discovered automatically by scanning tags with the `nav:` prefix. No hardcoded links needed.
- **Categorized Dropdowns** — Dashboards and external links are grouped into hover-activated dropdown menus (e.g. Monitoring, Tools, Docs).
- **Simple Links** — Dashboards tagged with just `nav:Default` appear as standalone top-level links.
- **External Links** — Add links to external tools (CI, CRM, status pages, etc.) directly in the config — they appear alongside dashboard links in the appropriate dropdown.
- **SVG Category Icons** — Each dropdown category can have an inline SVG icon.
- **Font Awesome Dashboard Icons** — Individual dashboard entries can be mapped to FA icons by title.
- **Animated Hover Indicator** — A sliding underline indicator tracks the hovered nav item using `getBoundingClientRect()` positioning.
- **Frosted Glass Theme** — Dark blueish semi-transparent background with `backdrop-filter: blur(12px)`.
- **Logout Button** — Icon-only by default, expands with text on hover. Turns red on hover.
- **Responsive** — Adjusts padding/font at `≤768px`.
- **Library Panel Support** — Best used as a Library Panel for consistent navigation across dashboards.
- **Session Caching** — Team lookup is cached in `sessionStorage` to avoid repeated API calls.
- **Duplicate Prevention** — Skips re-rendering if the navbar was already built for the current dashboard.

<br>

## 🏷️ Tag System

Dashboard discovery is driven entirely by Grafana tags. Add tags to your dashboards to control where they appear in the navbar.

### Tag Format

```
nav:{Team}:{Category}:{Order}
```

| Tag | Behavior |
|---|---|
| `nav:Default` | Simple top-level link visible to **all** users |
| `nav:Default:Monitoring` | Appears in the "Monitoring" dropdown for **all** users |
| `nav:Default:Monitoring:1` | Same, but sorted at position 1 within the dropdown |
| `nav:Engineering` | Simple link visible only to the **Engineering** team |
| `nav:Engineering:Tools` | Dropdown item under "Tools" for Engineering only |
| `nav:Engineering:Tools:3` | Same, sorted at position 3 |

- Tags are **case-insensitive** — `nav:default:monitoring` works the same as `nav:Default:Monitoring`.
- A dashboard can have **multiple** `nav:` tags to appear in several teams' navbars.
- The `Default` team is special: its items are visible to **everyone**, regardless of team.

<br>

## 🏢 Team Configuration

The `TEAM_CONFIGS` object in `afterrender.js` controls per-team behavior. Each team entry can define:

| Property | Description |
|---|---|
| `dropdownOrder` | Array of category names in display order (e.g. `['Monitoring', 'Tools', 'Docs']`) |
| `categoryLinks` | Clicking a dropdown header navigates to this URL (e.g. `{ 'Docs': '/d/docs/docs' }`) |
| `categoryIcons` | Inline SVG icons shown next to category names in the navbar |
| `dashboardIcons` | Font Awesome class mapped by dashboard title (e.g. `{ 'Home': 'fa-house' }`) |
| `externalLinks` | Array of external links injected into dropdowns (see below) |

### Team Inheritance

Team configs **inherit** from `_default`. If a user is on the "Engineering" team:
1. The `_default` config loads first (categories, icons, external links)
2. The `Engineering` config merges on top (overriding/extending categories and links)
3. `dropdownOrder` is merged: default order first, then team-specific categories appended

### Pre-Configured Teams

The panel ships with example configs for: `Engineering`, `Operations`, `Product`, `Sales`, `Finance`, `HR`, `Leadership`. All use `example.com` placeholder URLs — customize them for your environment.

### Team Priority

When a user belongs to multiple Grafana teams, the first match in this priority order wins:

```
Leadership → Operations → Engineering → Product → Sales → Finance → HR
```

### Adding External Links

External links appear in dropdowns alongside dashboard links:

```js
externalLinks: [
  {
    category: 'Monitoring',    // Which dropdown to appear in
    order: 1,                  // Sort position within the dropdown
    title: 'Status Page',      // Display text
    url: 'https://status.example.com',
    faIcon: 'fa-signal'        // Font Awesome icon class
  }
]
```

<br>

## 📋 Prerequisites

- Grafana v12.0.0 or higher
- Plugins:
  - [Dynamic Text Panel](https://grafana.com/grafana/plugins/marcusolsson-dynamictext-panel/) (marcusolsson-dynamictext-panel v5.7.0+)

<br>

## ⚙️ Installation

### 🔽 Import via Grafana UI

1. Clone this repository:

```bash
git clone https://github.com/IT-BAER/grafana.git
```

2. In Grafana, navigate to **Dashboards** → **+ Import**
3. Click on **Upload JSON file** and select the `panel.json` file from this directory
4. Before finalizing the import, ensure the panel is using the correct data source

### 📚 Using as a Library Panel (Recommended)

For consistent navigation across all your dashboards:

1. After importing, convert the panel to a Library Panel:
   - Click the panel title → **More** (⋯) → **Make library panel**
   - Name it (e.g. "Navbar") and save

2. Add the Library Panel to any dashboard:
   - **Add** → **Visualization** → search for your library panel name

3. Tag your dashboards with `nav:` tags (see [Tag System](#%EF%B8%8F-tag-system) above)

🎉 **You're all Set Up!**

<br>

## 🔧 Customization

### Logo

Replace the image path in the HTML content section. The default points to `public/custom/img/logos/logo.png` — place your logo there, or use any URL:

```html
<img src="public/custom/img/logos/logo.png" height="24px">
```

### Adding a New Team

1. Create a Grafana team (e.g. "DevOps") in **Administration** → **Teams**
2. Add a config entry in `TEAM_CONFIGS` in the afterRender JS:
   ```js
   'DevOps': {
     dropdownOrder: ['Monitoring', 'Tools', 'Cloud'],
     categoryLinks: { 'Monitoring': '/d/devops-overview/overview' },
     externalLinks: [
       { category: 'Tools', order: 1, title: 'ArgoCD', url: 'https://argo.example.com', faIcon: 'fa-arrows-spin' }
     ]
   }
   ```
3. Tag dashboards with `nav:DevOps:Category:Order`

### Team Name Aliases

Short tag names can be mapped to full Grafana team names via `TEAM_NAME_MAP`:

```js
const TEAM_NAME_MAP = { 'Ops': 'Operations', 'Eng': 'Engineering' };
```

This allows using `nav:Ops:Monitoring` in tags instead of `nav:Operations:Monitoring`.

<br>

## 📁 Source Files

The panel JSON contains everything needed, but the source code is also available as separate files for easier editing:

| File | Description |
|---|---|
| `panel-content/navbar/content.html` | HTML structure (nav container, logo, logout button) |
| `panel-content/navbar/styles.css` | CSS (frosted glass theme, dropdowns, hover effects, responsive) |
| `panel-content/navbar/afterrender.js` | JavaScript (team detection, tag parsing, dropdown rendering) |

After editing source files, copy their contents back into the corresponding `options.content`, `options.styles`, and `options.afterRender` fields in `panel.json`.

<br>

## 🚨 Troubleshooting

**Navigation links not appearing:**
1. Verify dashboards have `nav:` prefixed tags (e.g. `nav:Default`)
2. Check browser console for `Navbar:` log messages — the script logs its progress
3. Ensure the Grafana API is accessible from the panel (`/api/search` and `/api/user/teams`)

**Wrong team's navbar showing:**
1. Clear `sessionStorage` (`itb_user_team` key) — team lookup is cached per session
2. Verify team membership in **Administration** → **Teams**
3. Check `TEAM_CONFIGS` has an entry matching the exact Grafana team name

**Styling issues:**
1. The panel must be set to **Transparent** mode
2. Check that no other CSS conflicts with `.custom-nav` or `.itb-dropdown-menu` selectors
3. Dropdown menus render in a `position:fixed` container appended to `document.body` — z-index issues may require adjusting `z-index:99999`

**Dropdowns not closing:**
1. The panel registers a global click handler — ensure no other script is calling `stopPropagation()` on click events

<br>

## 💜 Support Development

If you find any of these Dashboards useful, consider supporting this and future creations, which heavily relies on Coffee:

<div align="center">
<a href="https://www.buymeacoffee.com/itbaer" target="_blank"><img src="https://github.com/user-attachments/assets/64107f03-ba5b-473e-b8ad-f3696fe06002" alt="Buy Me A Coffee" style="height: 60px !important;max-width: 217px !important;" ></a>
</div>
