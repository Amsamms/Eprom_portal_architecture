# Design System

**Last Generated:** 2026-03-26

---

## Color Palette

### Primary Colors

| Name | Hex | Usage |
|------|-----|-------|
| **EPROM Blue** | `#00529B` | Headers, primary branding, footer background, card hover shadows |
| **EPROM Blue Light** | `#1E83EC` | Gradient accents, animated gradient backgrounds |
| **EPROM Green** | `#00A651` | Success states, Enterprise tier, gradient buttons, status dots |
| **EPROM Teal** | `#006175` | Button gradients (teal→green), secondary branding |

### Background Colors

| Name | Hex | Usage |
|------|-----|-------|
| **Primary BG** | `#FFFFFF` | Cards, modals, sidebar |
| **Secondary BG** | `#F5F7FA` | Page body background |
| **Tertiary BG** | `#EDF0F5` | Nested sections, alternating rows |

### Text Colors

| Name | Hex | Usage |
|------|-----|-------|
| **Primary Text** | `#1A2B4A` | Headings, body text |
| **Secondary Text** | `#5A6A85` | Labels, descriptions, Basic tier color |
| **Tertiary Text** | `#8A95A8` | Placeholders, muted text |

### Semantic Colors

| Name | Hex | Usage |
|------|-----|-------|
| **Danger** | `#DC2626` | Error states, delete buttons |
| **Warning** | `#F59E0B` | Warnings, reports permission badge |
| **Purple** | `#8B5CF6` | AI & ML Suite accent |

### Borders & Shadows

| Token | Value |
|-------|-------|
| `--border-color` | `#D9DEE5` |
| `--card-shadow` | `0 2px 12px rgba(0,82,155,0.08)` |
| Card hover shadow | `0 8px 24px rgba(0,82,155,0.12)` |

---

## Typography

| Font | Usage | Source |
|------|-------|--------|
| **Inter** | App UI text (heater, pump, massmole after visual consistency unification) | Google Fonts |
| **Public Sans** | Portal body text, navigation, forms | Google Fonts |
| **Urbanist** | Display headings (portal) | Google Fonts |

The portal uses Tailwind CSS for typography. App frontends use custom CSS.

---

## CSS Design Tokens (`:root` Variables)

From `portal/src/app/globals.css`:

```css
:root {
  --eprom-blue: #00529B;
  --eprom-blue-light: #1E83EC;
  --eprom-green: #00A651;
  --eprom-teal: #006175;
  --bg-primary: #FFFFFF;
  --bg-secondary: #F5F7FA;
  --bg-tertiary: #EDF0F5;
  --border-color: #D9DEE5;
  --text-primary: #1A2B4A;
  --text-secondary: #5A6A85;
  --text-tertiary: #8A95A8;
  --footer-bg: #00529B;
  --footer-text: #FFFFFF;
  --sidebar-bg: #FFFFFF;
  --sidebar-active: #E8F5EE;
  --card-shadow: 0 2px 12px rgba(0,82,155,0.08);
  --danger: #DC2626;
  --warning: #F59E0B;
  --purple: #8B5CF6;
}
```

The heater app (`heater/public/css/main.css`) defines its own extended set of tokens:

```css
/* Heater-specific tokens (in addition to matching portal colors) */
--eprom-radius-sm: 4px;
--eprom-radius-md: 8px;
--eprom-radius-lg: 12px;
--eprom-radius-pill: 50px;
--eprom-shadow-sm: 0 1px 3px rgba(0,82,155,0.06);
--eprom-shadow-md: 0 4px 12px rgba(0,82,155,0.08);
--eprom-shadow-lg: 0 8px 24px rgba(0,82,155,0.12);
/* Font: Inter (not Public Sans) for app UI */
```

All apps were unified to the same light theme in Session 40.

---

## Component Patterns

### Buttons

**Primary button (`.btn-primary`):**
- Gradient: `linear-gradient(to right, var(--eprom-teal), var(--eprom-green))`
- Color: white, font-weight 600
- Border radius: `40px` (pill shape)
- Hover: translateY(-1px), green glow shadow `0 4px 16px rgba(0,166,81,0.25)`, sweep shine effect
- Active: translateY(0)

### Cards

**Glass card (`.glass-card`):**
- Background: white (`var(--bg-primary)`)
- Border: 1px solid `var(--border-color)`
- Shadow: `var(--card-shadow)`
- Border radius: 12–16px (varies by context)
- Hover: border tints to blue `rgba(0,82,155,0.25)`, shadow deepens, translateY(-2px)

**Glassmorphism card (`.glass-card-frost`):**
- Background: `rgba(255, 255, 255, 0.94)` with `backdrop-filter: blur(16px)`
- Used on login page and dashboard overlays (lightened in Session 47)

### KPI Cards

- White background, subtle shadow
- Large number with animated `countIn` entrance
- Label below in secondary text color
- Green/blue hover variants (`.card-hover-green`, `.card-hover-blue`)

### Inputs

- Focus: outline removed (custom focus styles per context)
- Placeholder: `var(--text-tertiary)`
- Touch target minimum: 44px height and width

---

## Header

All apps use a gradient header:
- Background: `linear-gradient(135deg, #00529B, #003F7A)` (dark blue gradient)
- Text: white
- EPROM logo (local SVG — `eprom-logo.svg`) with shimmer animation (`logoShimmer` — subtle brightness pulse with green drop-shadow)
- Height adapts for mobile

---

## AI Chat Icon (EPROM Tears Logo)

All 8 chatbots (Dashboard, Heater, Pump, MassMole, Optimizer-General, Optimizer-Energy, Optimizer-Distillation, Optimizer-Flare) use the official **EPROM tears PNG logo** (`eprom-tears.png`), placed in each app's public assets directory.

- **Image source:** `logo-large-trans.png` (official EPROM branding asset)
- **3 icon locations per app:** FAB button (floating circle), chat panel header logo, bot message avatar
- **Avatar container:** White `#FFFFFF` circle with `1.5px solid #D9DEE5` border, 36px diameter

The original inline two-drops SVG (blue `#1565C0` + green `#4CAF50`) was replaced with the official PNG logo in Session 57 across all 8 apps (16 files modified).

---

## Chat Panel

### Desktop (>1200px)
- Side panel: 400–420px fixed width
- Main content shrinks to accommodate (push-aside layout, not overlay)
- Toggle button in header or floating

### Mobile
- 3-state bottom sheet:
  - **Peek:** Small handle visible at bottom (~60px)
  - **Half:** Takes 50–55vh of screen
  - **Full:** Takes ~92–95vh of screen
- Drag handle for gesture control
- Snap points for each state

### Chat Headers
All chat panels use a **white background** with dark text (`#1A2B4A`), standardized in Sessions 51-52. The close (X) button uses `#5A6A85` with hover `#1A2B4A`.

### Chat FAB (Floating Action Button)
- White circle with two-drops SVG icon
- Hides when chat panel is open (via `body.chat-open` CSS class)
- In iframe mode: apps hide their own navigation but keep the FAB visible

### Chat Bubbles
- User messages: blue-tinted background, right-aligned
- AI messages: light gray background, left-aligned
- Two-drops avatar on AI messages
- Typing indicator: animated dots
- Markdown rendering in AI responses
- Code blocks with copy button
- Timestamps on each message

### Onboarding Tips
First-time users see contextual tips during their first 4 visits:
- Visit 1 mobile: "Drag the gray handle to collapse/expand the chat"
- Visit 1 desktop: "Your AI assistant can fill inputs and analyze results"
- Visit 2: "Try the quick action chips for common tasks"
- Visit 3: "Press Ctrl+/ to toggle chat, Escape to close"
- Visit 4: "The AI can compare scenarios and explain results"
- Auto-dismiss after 8 seconds, dismissible via X button

---

## Animations

| Animation | Keyframe | Usage |
|-----------|----------|-------|
| `fadeInUp` | Translate Y 28px → 0 + fade in | Card entrances, page load |
| `shimmer` | Background position sweep | Shimmer text effect |
| `gradientShift` | Background position 0→100→0 | Animated gradient backgrounds |
| `float` | Translate Y 0 → -8px → 0 | Floating elements (4s loop) |
| `countIn` | Translate Y 10px → 0 + fade in | KPI number reveal |
| `logoShimmer` | Brightness 1 → 1.08 with green glow | Logo pulse on dashboard |
| `glowThrob` | Card shadow intensity pulse | Featured cards |
| `particleDrift` | Translate Y upward + fade | Floating particles |
| `orbFloat` | Translate + scale oscillation | Floating orb decorations |
| `spinSlow` | Full rotation, 20s | Decorative spinning elements |

All animations respect `prefers-reduced-motion: reduce` — particles, shimmer, float, and glow animations are disabled for accessibility.

---

## Mobile Responsiveness

| Breakpoint | Behavior |
|------------|----------|
| **> 1200px** | Full desktop: side-by-side chat panel, wide cards |
| **768px – 1200px** | Tablet: stacked layout, chat as overlay |
| **< 768px** | Mobile: single column, bottom sheet chat, particles reduced |
| **< 480px** | Small mobile: further font/padding reductions |

### Touch Targets
All interactive elements (buttons, links, inputs) have a minimum 44×44px touch target (`.touch-target` class).

### Scrollbar
Thin 5px scrollbar with `var(--border-color)` thumb, `var(--bg-secondary)` track.

---

## Vendor Override Pattern (Heater/Bootstrap)

The heater app imports Bootstrap 5 (dark theme) for its form layout and grid. To match the EPROM corporate light theme, the heater's `main.css` uses `!important` overrides:

```css
/* Example: Override Bootstrap's dark navbar */
.navbar { background: linear-gradient(135deg, #00529B, #003F7A) !important; }
.form-control { background: #FFFFFF !important; color: #1A2B4A !important; }
```

**Rule for new apps:** If you import a CSS framework (Bootstrap, Bulma, etc.), override its defaults to match the EPROM design tokens using the `!important` pattern. Do not let framework defaults leak through.
