# SPQ Prototype — Claude Code Project Brief

A prototype for **Suggested Prompted Questions (SPQs)** inside a floating Shopping Assistant chat widget, on a realistic recreation of **Hotel Lobby Candle** (https://hotellobbycandle.com/), with a Gorgias admin settings surface (built with Axiom) that controls the experience.

---

## 🚀 How to use this brief in Claude Code

You (the Claude Code agent) are building a real local project, not a single-file artifact. Scaffold it, run it, and iterate.

**Files in this directory:**

- `spq-prototype-brief.md` — this file. The full product brief.
- `gorgias-settings-scaffold.html` — a static HTML/CSS scaffold of the Gorgias admin Shopping Assistant page. Use it as a structural reference for sidebar layout, spacing, Axiom token usage, and toggle / card styling. **Don't copy it blindly** — the scaffold is missing some elements (notification badge, Beta/New tags, "Ask Gaia" pill, gradient AI orb, gear icons, per-card descriptions, "Set Up" button on the Embedded card, "Track Performance" links) and has some out-of-date copy. The canonical source for content is this brief plus the reference screenshot the user attached.
- The user may also attach a screenshot of the Gorgias Shopping Assistant page. Treat it as the visual ground truth.

**Recommended tech stack:**

- Vite + React + TypeScript
- Tailwind CSS
- framer-motion (for the chat panel and drawer animations)
- lucide-react (icons)
- Zustand (state — chat, cart, SPQ context, settings)
- react-router-dom (for routing between `/`, `/collections/:slug`, `/admin/shopping-assistant`)
- react-markdown (assistant message rendering)

**Project setup (do this first):**

```bash
npm create vite@latest . -- --template react-ts
npm install
npm install tailwindcss postcss autoprefixer
npx tailwindcss init -p
npm install framer-motion lucide-react zustand react-router-dom react-markdown
```

Configure Tailwind for the project (`content` paths, custom theme tokens for the pastel gradient palette, Axiom purple `#7e55f6`, neutrals `#1e242e` / `#ebecef` / `#fafafa`).

**Suggested directory layout:**

```
src/
  routes/
    Home.tsx
    Collection.tsx           // dynamic — Candles / Diffusers / Handwash
    AdminShoppingAssistant.tsx
  components/
    storefront/
      TopNav.tsx
      ShopMegaMenu.tsx
      ProductCard.tsx
      FloatingAdminButton.tsx  // top-left "⚙ Gorgias settings"
    chat/
      ChatWidget.tsx           // launcher + panel shell + thread + composer
      SuggestedPromptChips.tsx // SPQ chip row, reads page context
      ChatProductCard.tsx
      ExpandableAddTile.tsx
      CompactCartList.tsx
      SubscriptionCard.tsx
      ChatToast.tsx
    admin/
      Sidebar.tsx
      FeatureCard.tsx
      SpqDrawer.tsx            // right-side drawer for "AI FAQs: Floating above chat"
      AskGaiaPill.tsx
      AiOrb.tsx
  store/
    cartStore.ts
    chatStore.ts
    spqStore.ts                // chip sets per page context + settings (Home/Collection/Product toggles, overrides)
  styles/
    index.css                  // Tailwind + global tokens
```

**Build order (do it iteratively, validate as you go):**

1. Project scaffold + Tailwind + routing + global tokens
2. Storefront base (Home + three collection pages with 6 products each)
3. Top-left "⚙ Gorgias settings" button + corresponding "← View store" button in admin
4. Chat widget shell — launcher, open panel, gradient hero, message thread, composer
5. SPQ chip row inside the chat widget, with page-context-aware content
6. Chip tap → first user message + assistant response flow
7. Gorgias admin Shopping Assistant page (sidebar + tabs + four feature cards), Axiom-styled
8. SPQ configuration drawer (slides in when "AI FAQs: Floating above chat" is clicked)
9. Wire the drawer's toggles to actually hide/show the chip row on the storefront
10. Mobile responsive pass — verify storefront and admin both work down to ~375px

**Run it:**

```bash
npm run dev
```

After each meaningful step, take a screenshot or describe the state and confirm with the user before moving on.

---

## 🧩 What this prototype should demonstrate

Think of this less like a feature and more like a behavior shift:

**Core story**
"The Shopping Assistant meets shoppers with the right questions, in the right context, before they have to ask — and merchants control what those questions are from inside Gorgias."

Mental model:

- The chat widget is **one persistent surface**, floating bottom-right on every page of the storefront
- **SPQs are the suggested prompt chips inside the widget** — they appear in the open panel before the shopper sends their first message
- The chips **adapt to the page the widget is opened from** — exploration questions on Home, selection questions on Collection pages
- A **Gorgias admin settings page** lets the merchant toggle, preview, and customize the SPQ experience
- The storefront and the admin are **linked by a top-left "⚙ Gorgias settings" button on the brand site** and a "← View store" button in the admin, so it's easy to round-trip and demonstrate that merchant changes affect the storefront live
- 📱 Everything works on mobile and desktop — mobile is the primary surface for the storefront; the admin is desktop-first but should still be responsive

---

## 🏗️ Section-by-section spec

### 1. Storefront base (Hotel Lobby Candle recreation)

Recreate the storefront realistically so the prototype feels like a real candle brand, not a wireframe.

**Visual reference:** https://hotellobbycandle.com/ — calm, editorial, lots of whitespace, warm neutral palette, serif headlines, large product photography.

**Navigation (match the live site):**

- Top nav with the same items as hotellobbycandle.com
- When the user clicks **Shop**, the dropdown reveals three options:
  - **Candles** → `/collections/candles` (styled after https://hotellobbycandle.com/collections/candles)
  - **Diffusers** → `/collections/reed-diffusers` (https://hotellobbycandle.com/collections/reed-diffusers)
  - **Handwash** → `/collections/hand-wash` (https://hotellobbycandle.com/collections/hand-wash)

**Collection pages:**

- Each collection page shows **6 products** in a clean grid
- Preserve the distinct, photographic look of the candles — varied vessel shapes, colors, labels
- Product cards: large image, product name, scent name, price
- You can use placeholder imagery or pull representative images from the live site (note in a comment that they're for prototype purposes only)

**Mobile:** collapse top nav to a hamburger, stack the product grid (2 columns or 1 with prominent imagery), preserve the editorial feel.

### 2. Cross-surface navigation (the bridge between storefront and admin)

- In the **upper-left corner** of every page on the brand site, add a small floating utility button labeled "⚙ Gorgias settings" (or a Gorgias logo icon with a tooltip). Visually distinct from the brand's nav — small, unobtrusive, obviously a prototype-only affordance.
- Clicking it routes to `/admin/shopping-assistant`.
- In the Gorgias admin, add a "← View store" button (top-left or header area) that returns to `/`.

### 3. Chat widget (lives on every page of the storefront)

Build a floating AI shopping chat widget. Match the structure, IA, and visual style closely.

**Launcher (closed state)**

- Fixed circular button at `bottom-6 right-6`, `z-50`, `h-14 w-14`, `rounded-full`
- Background: a soft animated conic/linear gradient of warm pastels (peach → blush → lavender → mint), e.g. `background: conic-gradient(from 180deg at 50% 50%, #F8D7C2, #F5C6D6, #D6C9F0, #C9E4D6, #F8D7C2)`. Subtle 8s rotate animation via framer-motion or CSS
- Soft outer shadow, small white sparkle/chat icon centered
- Tiny cart badge (number) overlaid top-right when items > 0
- Hover: gentle scale 1.05

**Open panel container**

- Slides up from bottom-right into a fixed panel: width ~`420px` desktop, full-width on mobile, height ~`min(720px, 90vh)`
- `rounded-3xl`, white background, large soft shadow, `overflow-hidden`
- framer-motion entry: `initial={{ opacity: 0, y: 20, scale: 0.96 }} animate={{ opacity: 1, y: 0, scale: 1 }}`, spring transition

**Information architecture (top → bottom)**

1. **Header bar (~56px)** — left: back arrow (only when not on home), brand name "Hotel Lobby Candle" in a refined serif. Right: cart icon with live badge, close X.
2. **Context breadcrumb (only on collection pages)** — thin muted line under the header: `💬 From {Collection name} collection`. Absent on Home.
3. **Gradient hero strip (only on initial/empty state)** — ~140px tall, same pastel gradient as launcher, soft horizontal/radial wash. Centered greeting in serif: "How can I help you today?" Collapses once the conversation starts.
4. **Message thread (scrollable, flex-1)** — assistant messages plain text on white, user messages right-aligned pill bubbles, typing indicator dots, render markdown via react-markdown.
5. **Rich inline message types** — 2-col product grid cards, option pills, expandable add-to-cart tile, compact cart list, subscription card, styled action chips.
6. **SPQ chip row (only before first user message, hidden once typing or in an active flow)** — horizontally scrollable, pills `rounded-full border border-border px-3 py-1.5 text-xs`, hover `bg-secondary`. Content adapts based on page context (see Section 4).
7. **Composer (sticky bottom)** — rounded input row, image/attachment icon, textarea with placeholder "Ask me anything…", send button (circular, pastel gradient).

**Interactions** — Zustand state (non-persistent), cart badge live, simple keyword intent detection, smart attribute resolution, add-to-cart confirmation toast, PDP modal on product card click, checkout sub-view.

**Visual system** — Inter for body, refined serif (Cormorant or Instrument Serif) for brand mark + hero greeting. White surfaces, pastel gradient only on launcher/hero/send button. `rounded-2xl` cards, `rounded-3xl` panel, `rounded-full` pills. Soft shadows. AnimatePresence transitions. Sentence case buttons.

### 4. SPQ chip content per page context

**Home page → exploration intents (no declared category):**

- "Help me find the right candle for my space"
- "Which scent feels most like a hotel lobby?"
- "I'm new to luxury candles — where do I start?"
- "What's the difference between candles and diffusers?"
- "What's your bestselling scent right now?"
- 🛟 Pin a safe boilerplate last: "I'm looking for a specific product"

**Candles collection page → selection intents (category intent declared):**

- "What's the difference between your top three scents?"
- "Which candle is best for a bedroom?"
- "Which scent works for a dinner party?"
- "What's your longest-burning candle?"
- "Which candles come in larger sizes?"

**Diffusers and Handwash collection pages** — equivalent selection-style chips tailored to those categories.

👉 Key signal: if a collection-page chip could be lifted onto the Home page unchanged, it's too generic. Make the contrast visible.

### 5. Chip → first message → assistant response

Tapping a chip:

- Inserts the chip's text as the shopper's first user message
- Hides the chip row
- Collapses the gradient hero
- Triggers a warm, editorial, brand-appropriate assistant response (canned per chip for the prototype is fine — no real model call needed)

### 6. Gorgias admin Shopping Assistant page (match the reference screenshot exactly)

Built with the **Axiom design system look** (Inter font, purple accent `#7e55f6`, rounded cards with subtle `#ebecef` borders, Axiom-style Toggle / Tag / Button / Tabs). The user has the `axiom-vibecoder` skill available — invoke it for guidance on Axiom components and tokens. Use `gorgias-settings-scaffold.html` as a structural reference for layout and spacing, but rebuild it as a clean React + Tailwind component tree, not a port of the static HTML.

**Layout — two regions:**

**Left sidebar (~280px, light `#FAFAFA`):**

- Product switcher "AI Agent ▾" at the top with logo + notification bell (badge "3") + search + collapse icon
- Store selector "ariagroult" with a green online dot
- Collapsible nav sections with chevrons and item icons:
  - **Analyze**: Overview, Intents, Opportunities (Beta tag)
  - **Train**: Skills (New tag), Knowledge, Tone of Voice, Support Actions, Products, **Shopping Assistant (active — purple highlight)**
  - **Test** (play icon)
  - **Deploy**: Chat (green status dot), Email / SMS / Socials (grey dots)
  - **Settings** (gear)
- Footer: user avatar "AP" + icons
- Floating "Ask Gaia ⌘+G" pill bottom-left
- Gradient AI orb bottom-right corner

**Main area:**

- Header with title "Shopping Assistant" and top-right "Save Changes" + "Test" buttons
- "← View store" button (prototype-only) returns to the brand site
- Tab row: **Strategy / Customer Engagement (active, underlined) / Product Recommendations**
- Vertical list of feature activation cards

**Feature card pattern** (left preview thumbnail with gradient bg and mini UI mockup · center title + grey description + optional "Track Performance" link · right-side control varies per card):

- **AI FAQs: Embedded in page** → "Set Up" button (no toggle)
  - "Show up to 3 dynamic, AI-generated questions embedded directly in product pages to resolve pre-sales questions and drive conversion. On pages where this is installed, the Floating AI FAQs are automatically hidden to avoid duplicates."
- **AI FAQs: Floating above chat** → gear icon + purple toggle (on) + "Track Performance" link
  - "Show up to 3 AI-generated questions above chat to answer common shopper questions and start conversations. Automatically hidden on product pages where Embedded AI FAQs are installed."
- **Search assist** → toggle (on) + "Track Performance" link (no gear)
  - "Send a personalized message right after a shopper searches to guide them to the right product and drive more conversions."
- **Ask anything input** → gear icon + toggle (on)
  - "Drive more sales by adding an always-on input field that encourages shoppers to start a conversation."

Toggles, tabs, and gear icons should all be interactive. Status dots, tags, notification badge, gradient preview thumbnails, and the bottom-right AI orb all matter — easy to skip, sells the realism.

### 7. SPQ configuration drawer (slides in from the right)

Clicking the **"AI FAQs: Floating above chat"** card (anywhere on the card, or specifically on the gear icon — both should work) slides a drawer in from the right side of the screen, overlaying the main content with a soft scrim behind it.

- ~480px wide on desktop, full-width on mobile
- Axiom drawer pattern: rounded inner corners, header bar, close X top-right, scrollable body, sticky footer
- framer-motion: slide from right, ~200ms ease-out

**Drawer contents (top to bottom):**

- **Header**: "AI FAQs: Floating above chat" + close (X) button
- **Description**: one-liner reminding what this controls
- **Per-surface toggles**:
  - "Show on Home Page" — default ON
  - "Show on Collection Pages" — default OFF (staged rollout)
  - "Show on Product Pages" — default ON
- **Preview pane**: realistic mock of the chat widget open state with the SPQ chip row visible
  - Tabs above the preview: **Home / Collection / Product**
  - For the Collection tab: small dropdown picker (Candles / Diffusers / Handwash)
  - "Regenerate" button next to the preview
- **Per-collection overrides** table: rows for Candles / Diffusers / Handwash with columns Status (Auto / Custom / Disabled), Last generated, Edit action
- **Edit modal** (opens on Edit click): pin / hide / reorder / rewrite individual chips
- **Knowledge sources** (placeholder section)
- **Language** (read-only)
- **Sticky footer**: "Cancel" + "Save changes" buttons

**Critical wiring:** toggling a surface off in the drawer should hide the SPQ chip row in the widget on the corresponding storefront page when the merchant flips back to "View store." This is the demo's punchline — settings actually affect the storefront live.

---

## 🎯 What the finished prototype must communicate

1. The storefront feels like a real Hotel Lobby Candle site, including the Shop → Candles / Diffusers / Handwash navigation
2. The Shopping Assistant chat widget is a single persistent surface, floating bottom-right on every page
3. The SPQ chips inside the widget adapt their content based on page context — exploration on Home, selection on each collection page
4. A context breadcrumb in the widget shows which page the conversation started from (on collection pages only)
5. The Gorgias admin Shopping Assistant page matches the reference screenshot exactly, built with Axiom styling
6. Clicking "AI FAQs: Floating above chat" opens a right-side drawer where the merchant configures the SPQ experience
7. The "⚙ Gorgias settings" button (top-left of brand site) and "← View store" button (in admin) make round-tripping easy, and merchant changes in the drawer visibly affect the storefront

---

## 🧠 Smart details worth including

- **Pin a "safe" chip last on Home** ("I'm looking for a specific product") — anchors the experience in familiarity
- **Breadcrumb appears only on Collection pages** — its absence on Home is itself a signal that the conversation is open-ended
- **Default Collection SPQs to OFF** in the drawer — staged rollout pattern
- **The "⚙ Gorgias settings" button is a prototype-only affordance** — style it slightly out-of-band so demo viewers understand it's a navigation aid
- **The right-side controls on the feature cards are NOT all the same** — Set Up button vs. gear + toggle vs. toggle vs. link. Build each explicitly.
- **Status dots, tags, notification badge, AI orb, gradient thumbnails** — these are what make the admin page read as real Gorgias UI. Don't skip them.

---

## 🧪 Contrarian variant worth trying (optional, after the main flow works)

The current spec shows the chips only after the shopper opens the panel. Try a version where the launcher itself **previews one rotating chip** as a small tooltip-style label next to the orb (e.g. "Ask: Which candle is best for a bedroom?") on the Candles collection page. Pulls the SPQ value into the closed state. Good A/B candidate.
