# Dirt Girls Field Journal
## App Design Specification — for building with Claude Code

*This document is a blueprint. Open Claude Code, paste the relevant sections as your prompt, and let it build. You can build it all at once or piece by piece — up to you.*

---

## What This App Is

A Progressive Web App (PWA) for Android (Pixel 7 Pro) that serves as a quick-capture field journal for a backyard ecology project. It needs to feel fast, simple, and thumb-friendly. The user is usually standing in a garden, often holding a child's hand or crouching next to a plant. Every extra tap is a reason to not use it.

The app is called **Dirt Girls**.

There are two observers: **Courtney** (adult naturalist, UHD student) and **Toni** (junior naturalist, the Third Sister). Both should be able to log observations in their own voice.

---

## Core Workflow — Photo First, Annotate Later

This is the most important design decision. The camera is always the first action.

### The 30-Second Path
1. User taps app icon on home screen
2. App opens to a **big camera button** (center of screen, impossible to miss)
3. User takes a photo (uses phone camera via the HTML file input with capture attribute)
4. Photo is captured with automatic timestamp
5. A quick annotation card appears:
   - **Which plant?** (tap to select from a scrollable list)
   - **What is it?** (category: plant health / insect / bird / wildlife / soil / bloom / weather / general)
   - **Quick note** (free text field — works with Android voice-to-text keyboard)
   - **Who's observing?** (toggle: Courtney / Toni)
   - **Connection** (optional: "I saw [this] on/near [this plant]" — for tracking ecological relationships)
6. User taps **Save**
7. Observation is stored locally in the app

### The "I Already Took a Photo" Path
- A secondary button: **"Add from gallery"** — lets user select a photo already in their camera roll
- Same annotation flow follows
- This is for when they snapped a photo during the walkabout using the regular camera and want to annotate it later

### The "Unsorted Queue"
- Photos captured but not yet annotated sit in an **Unsorted** queue
- User can come back anytime (evening, weekend) and annotate them
- The queue shows photo thumbnails with timestamps
- Tapping a thumbnail opens the annotation card

---

## Plant Database (Starts Small, Grows)

The plant list should be editable and expandable. It starts with these entries:

### Trees
- Mother Tree (Eastern Redbud — Cercis canadensis)
- Sister 1 (Eastern Redbud — volunteer)
- Sister 2 (Eastern Redbud — volunteer)

### Current Companions (add these as starting entries)
- Turk's Cap
- Sea Oats (Inland Sea Oats)
- Pentas
- Easter Lily
- Spider Lily

### Adding New Plants
- At the bottom of the plant selection list: **"+ Add New Plant"**
- Tapping opens a simple form: common name, optional scientific name, location in yard (front/back/side)
- New plants appear in the list permanently going forward
- This is how the app grows from a Redbud tracker into a whole-yard compendium

---

## Observation Data Model

Each observation should be stored as a structured object containing:

```json
{
  "id": "unique-id",
  "date": "2026-03-17",
  "time": "13:42",
  "observer": "Courtney",
  "plant": "Mother Tree",
  "category": "insect",
  "note": "Carpenter bee with black dot on back working the lower branches",
  "connection": {
    "subject": "Carpenter bee",
    "relationship": "visiting",
    "target": "Mother Tree"
  },
  "photo": "base64-or-blob-reference",
  "photoTimestamp": "2026-03-17T13:42:15"
}
```

---

## Export / Save to OneDrive

The user has OneDrive on their phone with a Garden folder that is manually synced (NOT auto-sync). The app needs to make it easy to get observations OUT of the app and INTO that folder.

### Export Options
1. **Share button** — Uses the Android Web Share API to share a generated file. The user can then select "Save to OneDrive" from the share sheet and choose the folder manually. This is the simplest approach.

2. **Export session** — A button that bundles all new observations since last export into a set of markdown files (one per observation) that can be shared/saved as a batch.

### Export File Format
Each observation exports as a markdown file:

```markdown
# Field Observation — March 17, 2026 1:42 PM
**Observer:** Courtney
**Plant:** Mother Tree (Cercis canadensis)
**Category:** Insect visitor

Carpenter bee with black dot on back working the lower branches.

**Connection:** Carpenter bee → visiting → Mother Tree

**Photo:** [filename reference]
```

File naming: `YYYY-MM-DD_HHMMSS_PlantName.md`

---

## Visual Design

### Principles
- **Thumb-friendly**: All primary actions reachable with one thumb
- **High contrast**: Usable in direct Houston sunlight
- **Earthy palette**: Greens, warm browns, cream backgrounds — it should feel like a field notebook, not a tech app
- **Minimal text on screen**: Icons and taps over reading and typing
- **Big touch targets**: Nothing smaller than 44px

### Color Palette Suggestion
- Background: warm cream (#FDF6EC) or soft sage
- Primary action (camera button): earthy terracotta or deep green
- Text: dark brown (#3E2723)
- Accents: muted gold, leaf green

### Home Screen
- App name "Dirt Girls" at top (small, not dominant)
- Large camera button (center, primary action)
- "Add from gallery" button (smaller, below camera)
- "Unsorted (3)" badge if there are un-annotated photos
- Recent observations list (scrollable, showing last 5-10 with thumbnails)

### Annotation Card
- Plant selector (horizontal scrollable chips or a dropdown)
- Category selector (icon grid: leaf, bug, bird, paw, dirt, flower, cloud, star)
- Note field (single line expanding to multiline, voice-to-text ready)
- Observer toggle (two buttons: Courtney | Toni)
- Connection field (collapsible, optional — don't clutter the default view)
- Save button (large, bottom of screen)

---

## Technical Implementation

### Architecture
- **Single HTML file** with embedded CSS and JavaScript
- This is a PWA — it needs a `manifest.json` for home screen installation and a service worker for offline capability
- So the actual deliverable is 3 files: `index.html`, `manifest.json`, `sw.js`

### Key Web APIs
- **Camera**: `<input type="file" accept="image/*" capture="environment">` for taking photos
- **File input without capture**: for selecting from gallery
- **IndexedDB**: for local storage of observations, plant database, and photo blobs
- **Web Share API**: `navigator.share()` for exporting to OneDrive via share sheet
- **Service Worker**: for offline capability (the garden doesn't have WiFi)

### PWA Installation
The app should be installable to the Android home screen. This requires:
- `manifest.json` with app name, icons, theme color, display: standalone
- Service worker registered
- Served over HTTPS (GitHub Pages provides this for free)

### Hosting
- **GitHub Pages** (free, HTTPS, simple deployment)
- The user pushes the files to a GitHub repository, enables Pages, and the app is live at a URL they can bookmark and install

---

## Growth Path (Future Features — Don't Build Yet)

These are ideas for later iterations. Document them but don't implement in v1:

1. **Yard map**: Visual map of the yard with plant locations you can tap to see observation history
2. **Bug & bird ID integration**: Photo recognition suggestions (could use an API later)
3. **Seasonal timeline view**: Scroll through observations for one plant across months/years
4. **Food web visualization**: Map the connections data into a visual network showing which bugs visit which plants and what eats what
5. **Weather auto-fill**: Pull weather data from an API based on date/location
6. **Data dashboard**: Charts showing observation frequency, species counts, bloom timing
7. **Toni's sketchbook**: A drawing mode where Toni can sketch what she sees instead of typing

---

## How to Build This with Claude Code

### First Time Setup
1. Open your terminal (on your computer, not your phone)
2. Create a project folder: `mkdir dirt-girls && cd dirt-girls`
3. Type `claude` to start Claude Code
4. Paste the relevant section of this spec and say "Build this"

### Suggested Build Order
Start small. Don't try to build everything at once.

**Session 1: The basics**
"Build me a single-page PWA called Dirt Girls. It should have a camera button that takes a photo, a simple annotation form with plant name, category, a note field, and an observer toggle between Courtney and Toni. Store observations in IndexedDB. Use an earthy color palette — warm cream background, terracotta accents, dark brown text. Make it thumb-friendly for a phone screen."

**Session 2: The plant database**
"Add an editable plant database. Start with these plants: [list]. Let me add new plants. Show the plant list as selectable chips in the annotation form."

**Session 3: Export**
"Add a share/export button that generates a markdown file from the observation and uses the Web Share API so I can save it to OneDrive."

**Session 4: The unsorted queue**
"Add an unsorted photo queue. Photos that are captured but not annotated should appear in a queue I can come back to later."

**Session 5: Connections tracking**
"Add an optional 'connection' field to observations. Something like 'I saw [subject] on/near [plant]'. Store these relationships so I can eventually visualize a food web."

**Session 6: PWA installation**
"Add a manifest.json and service worker so this app can be installed to an Android home screen and works offline."

### Testing
- After each build session, open the HTML file in your phone's Chrome browser
- Test taking photos, filling forms, saving
- If something doesn't work, tell Claude Code what went wrong — it'll fix it

### Deploying to Your Phone
1. Create a free GitHub account (if you don't have one)
2. Create a new repository called `dirt-girls`
3. Push your files to it
4. Go to repository Settings → Pages → Deploy from main branch
5. Your app will be live at `https://yourusername.github.io/dirt-girls/`
6. Open that URL on your Pixel, tap the browser menu, tap "Install app" or "Add to home screen"
7. Done — Dirt Girls lives on your phone

---

## The Philosophy

This app exists because a mother and daughter walk through their garden every day and notice things. The technology should be invisible. The observations are what matter. If the app ever feels like work, something is wrong — simplify it.

The data you collect is real ecological science. Over years, the patterns in these observations will answer questions about your yard that no textbook could. You're building a living dataset of a recovering urban ecosystem, one 30-second entry at a time.

Welcome to field biology, Dirt Girls.
