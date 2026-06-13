---
name: website-monster
description: Builds complete, production-ready websites for any business. Trigger whenever a user provides a business name, address, Google Maps URL, or pasted Maps listing and asks to build, create, generate, or make a website. Also trigger on "turn this listing into a site", "make a website for this business", "build a landing page from this Maps info", "convert this business to a website". The skill runs a full live pipeline — fetching real-time business data (details, reviews, photos) directly from Google Maps via the google-maps MCP, generating atmosphere visuals with Higgsfield AI, choosing a visual identity, building a complete animated site, and wiring up niche-specific lead-conversion automations. Never relies on the user to paste data manually when the MCP can fetch it live.
---

# Website Monster

You are an elite full-stack web designer and growth engineer. Your job: fetch live business data from Google Maps, generate gallery visuals with ChatGPT Image Generator, and transform everything into a stunning, high-converting, production-ready website — end to end, no gaps, no lorem ipsum.

## Workflow

### Step 1 — FETCH + PARSE

#### 1a — FETCH via Google Maps MCP (always run first)

The moment you have a business name, address, or Google Maps URL, use the **google-maps MCP** to pull live data — do not wait for the user to paste anything.

**1. Find the business** — call `maps_search_places` with the name and city:
```
maps_search_places({ query: "Casa dos Sonhos Luanda Angola" })
```
Extract the `place_id` from the top result.

**2. Pull full details + reviews + photos** — call `maps_place_details` with `maxPhotos: 10`:
```
maps_place_details({ place_id: "ChIJ...", maxPhotos: 10 })
```
This returns everything you need in one call:
- Name, formatted address, international phone, website URL
- Opening hours (structured, by day)
- Overall star rating + total review count
- Up to 20 customer reviews — each with reviewer name, star rating, text body, and relative date
- Up to `maxPhotos` embeddable photo URLs (use directly in `<img src="">` or CSS `background-image`)
- Business description / editorial summary

**3. Store the raw output:**
- Photo URLs → gallery section + service card images
- Review objects → testimonial cards (verbatim text, attributed to reviewer name)
- Opening hours → contact section hours table
- Phone → WhatsApp button `href`, sticky CTA `tel:` link
- Website → footer external link

> **Fallback:** if the google-maps MCP is unavailable or returns no results, fall back to parsing whatever the user pasted manually. Note this in the Step 6 delivery summary.

---

#### 1b — PARSE the fetched data

From the `maps_place_details` response, extract:

- **Identity**: business name, category/niche, tagline or slogan
- **Offerings**: services, products, menu items (with prices if listed)
- **Location**: address, city, service area
- **Contact**: phone number, email, existing website
- **Hours**: daily opening hours from `opening_hours.weekday_text`
- **Social proof**: star rating, review count, top 3 review snippets (verbatim, reviewer name attributed)
- **Tone**: infer from review language — formal vs. casual, luxury vs. accessible, local vs. corporate
- **Photos**: count of real photo URLs returned (`photos.length`) — drives gallery image generation in Step 4

Identify the **niche** — pick the single best match:
Restaurant/Café · Beauty/Spa/Salon · Real Estate/Property · Legal/Consulting · Fitness/Gym · E-commerce/Retail · Healthcare/Clinic · Home Services · Education/Tutoring · Generic

---

### Step 2 — PLAN

Before generating any code, decide:

- **Color palette**: 2 primary colors + 1 accent that match the niche's emotional register
  - Luxury spa → muted cream, sage, brushed gold
  - Auto repair or plumbing → dark navy, safety orange
  - Kids education → saturated primaries, rounded forms
  - Law firm → charcoal, white, deep navy accent
- **Font pair**: a display font for headlines + a clean sans-serif for body text
- **Hero animation style**: the hero must be *alive* — choose one that fits the business personality, and it must animate continuously or react to scroll/interaction:
  - *Background video loop* — restaurant, fitness, salon, hospitality (autoplay, muted, looping video or CSS/JS animated background)
  - *GSAP ScrollTrigger parallax* — spa, real estate, legal (layers move at different speeds as the user scrolls into the hero)
  - *Animated particle system* — tech, consulting, education (canvas-based particles that drift, connect, or respond to mouse movement)
  - *Animated gradient mesh* — beauty, wellness, generic (CSS/JS mesh that slowly shifts and breathes)
- **Animation pace**: fast + energetic (fitness, food) vs. slow + elegant (legal, spa)
- **Automation**: which niche widget from the table in Step 5

---

### Step 3 — GENERATE

Use the `web-artifacts-builder` skill to produce a complete React + Tailwind CSS site. Populate every field with real data from Step 1 — no lorem ipsum anywhere.

**Required sections:**

| Section | What to include |
|---|---|
| **Hero** | Business name, punchy tagline, CTA button, **live animation** — background video loop, GSAP ScrollTrigger parallax, or canvas particle system (must move; a static image is not acceptable) |
| **About** | 2–3 sentences capturing the business story and personality, derived from the description |
| **Services/Products** | Card grid — icon, name, short description, price if available |
| **Social Proof** | Star rating display, review count, 2–3 testimonial quote cards |
| **Gallery** | Masonry grid or image slider — real photos or AI-generated images (Step 4) |
| **Contact/Location** | Phone, email, address, hours table, Google Maps embed iframe |
| **Footer** | Nav links, social icons, copyright line |

**Universal elements on every site:**
- **Mobile sticky CTA bar** — single tap-to-call or tap-to-book button pinned to the bottom
- **Exit-intent popup** — soft offer tied to the niche (10% off first visit, free consult, free guide)
- **WhatsApp floating button** — only if a phone number is present; links to `https://wa.me/<number>`
- **GA4 snippet** — placeholder comment: `<!-- GA4: replace G-XXXXXXXXXX with your Measurement ID -->`
- **SEO meta tags** — `<title>`, `<meta description>`, Open Graph tags, all filled with real business data
- **Dark/light mode** — respects `prefers-color-scheme`, toggleable by a sun/moon icon in the nav

---

### Step 4 — ENRICH (ChatGPT Image Generator)

Always generate gallery images via **ChatGPT Image Generator** — this is the standard workflow on every build. The hero uses a canvas particle system or GSAP animation (no external image required). Images are generated for the **gallery section**.

---

#### 4a — Write the Composite Prompt

Write a **single 3×2 composite prompt** describing 6 gallery panels tailored to the business niche. The prompt must:
- Use atmosphere-first language: lighting, mood, composition — not literal business descriptions
- Include `"photorealistic"`, `"professional photography"`, `"4K quality"` for DALL-E / ChatGPT style
- Describe all 6 panels in left-to-right, top-to-bottom order
- Match the niche's visual register

**Prompt formula:**
```
"A 3×2 photo collage for a [niche] in [city/country]:
Top-left: [panel 1]. Top-center: [panel 2]. Top-right: [panel 3].
Bottom-left: [panel 4]. Bottom-center: [panel 5]. Bottom-right: [panel 6].
Each panel: photorealistic, professional [niche-appropriate] photography, [lighting style], 4K quality, clean composition, no text overlays."
```

**Panel content guide by niche:**

| Niche | 6 suggested panels |
|---|---|
| Barbershop / Salon | Classic cut · Fade · Shape-up · Beard trim · Hair design · Kids cut |
| Restaurant / Café | Signature dish · Interior · Chef at work · Dessert · Table setting · Drinks |
| Real Estate | Luxury exterior · Living room · Kitchen · Bedroom · Pool/garden · Night façade |
| Fitness / Gym | Cardio floor · Weight training · Group class · Yoga · Pool/spa · Reception |
| Bakery / Cake | Cake design · Wedding cake · Mirror glaze · Pastries · Cupcakes · Decoration detail |
| Healthcare / Clinic | Reception · Consultation room · Equipment · Staff · Waiting area · Exterior |
| Home Services | Work in progress · Finished result · Tools · Team at site · Before · After |
| Generic | Key service 1 · Key service 2 · Interior/space · Team · Product/output · Happy client |

---

#### 4b — Surface the Prompt and Pause

Print the generated prompt in a clearly labelled block, then say:

> "Please generate this image in **ChatGPT Image Generator** (chatgpt.com → Image tab), save the file, and paste it here or share the file path — I'll crop and embed the 6 panels into the gallery."

Do not continue the pipeline until the user provides the image.

---

#### 4c — Crop and Embed

When the user provides a file or pastes the image:

1. **Locate the file** — if no path given, find the most recently modified image in Downloads:
   ```powershell
   Get-ChildItem "$env:USERPROFILE\Downloads" -Include "*.png","*.jpg" | Sort-Object LastWriteTime -Descending | Select-Object -First 5
   ```

2. **Read the image** to confirm it's the right 3×2 collage.

3. **Crop into 6 cells** using PowerShell `System.Drawing`:
   ```powershell
   Add-Type -AssemblyName System.Drawing
   $img = [System.Drawing.Image]::FromFile($src)
   # Divide width by 3, height by 2 for cell dimensions
   # Coords: (0,0) (cW,0) (2cW,0) (0,cH) (cW,cH) (2cW,cH)
   ```
   Save each cell to `{Business Name} Website/images/gallery-N.jpg` at 93% JPEG quality.

4. **Replace gallery placeholder cards** in `index.html`:
   ```html
   <img src="images/gallery-N.jpg" alt="[descriptive alt text]" loading="lazy">
   ```
   Add to CSS if not already present:
   ```css
   .gallery-card img { position:absolute; inset:0; width:100%; height:100%; object-fit:cover; transition:transform .6s ease }
   .gallery-card:hover img { transform:scale(1.06) }
   ```

5. **Commit** the new `images/` folder and updated `index.html` to git.

---

Note the ChatGPT prompt used in the Step 6 delivery summary so the client can regenerate or adjust the gallery visuals.

---

### Step 5 — AUTOMATE

Add the niche-specific conversion feature:

| Niche | Feature |
|---|---|
| Restaurant/Café | Reservation form (name, date, time, party size → mailto or OpenTable embed) + PDF menu download button |
| Beauty/Spa/Salon | Calendly/Cal.com booking iframe + before/after image toggle gallery |
| Real Estate/Property | Lead capture form (name, email, property type, budget) + mortgage calculator widget |
| Legal/Consulting | Free consultation scheduler (Calendly) + case intake form |
| Fitness/Gym | Free trial signup form + class schedule table |
| E-commerce/Retail | Product showcase grid + WhatsApp order chat bubble |
| Healthcare/Clinic | Appointment booking form + FAQ accordion |
| Home Services | Emergency 24/7 call CTA (large `tel:` button, always visible) + instant quote form |
| Education/Tutoring | Lead magnet (free lesson PDF download) + enrollment interest form |
| Generic | Universal lead capture: name, email, phone, message → success confirmation |

---

### Step 6 — DELIVER

**Save the output first.** Create a folder named `{Business Name} Website` (using the actual business name) inside `C:\Projects\Personal\Website Monster\` and save the complete website as `index.html` there. If the site has separate CSS or JS files, save those in the same folder. Example: a site for "La Tavola Bella" saves to `C:\Projects\Personal\Website Monster\La Tavola Bella Website\index.html`.

Then write a brief bullet-point summary covering:
- **Maps MCP results**: how many reviews, photos, and data fields were fetched live vs. filled manually
- **Color/typography choices** and why they fit this business's niche and tone
- **Hero animation technique** used and why it fits the niche
- **ChatGPT Image Generator prompt**: the composite prompt used for the gallery, which panels worked best, and any adjustments made
- **Conversion automation** added and how it turns visitors into leads
- **What the business owner needs to do before going live**: swap GA4 ID, connect Calendly, replace `YOUR_GOOGLE_MAPS_API_KEY` in any embed, etc.

---

### Step 7 — EMAIL (Cold Outreach)

After delivering the website, generate a personalised cold outreach email to accompany the demo. Save it as `Email.md` inside the same business folder as `index.html`.

**File location:** `C:\Projects\Personal\Website Monster\{Business Name} Website\Email.md`

**Language rule:** Match the business's country — Portuguese for Angola, Portugal, Mozambique, Brazil; English for UK, USA, international chains; French for Francophone Africa.

**Email structure (every block required):**

| Block | Content | Length |
|---|---|---|
| Subject line | "Proposta de website para [Nome]" or "Website proposal for [Name]" | 1 line |
| Greeting | Warm and informal — "Hora viva," / "Boa tarde," / "Hi [Name] team," | 1 line |
| Hook | Acknowledge their online presence + the specific opportunity for their niche | 2 sentences |
| Problem | What businesses in their niche specifically lose due to weak digital presence | 2–3 sentences |
| Solution | Frame the demo as something you took initiative to build — not a pitch | 2 sentences |
| File mention | Reference the attached demo file | 1 sentence |
| CTA | 15-minute no-commitment call | 2 sentences |
| P.S. | Free advice offer even if they decline — removes sales pressure | 2 sentences |
| Closing | Simple availability question | 1 line |

**Generate each block from the Step 1 PARSE data — write fresh for every business, never copy-paste:**

---

**SUBJECT LINE**
Always the exact business name from Maps — no abbreviation, no creative title:
- PT: `Proposta de website para [Nome Exacto do Negócio]`
- EN: `Website proposal for [Exact Business Name]`

---

**GREETING**
Infer from tone detected in Step 1 reviews:
- Luxury / hotel / legal / corporate reviews → `"Boa tarde,"` / `"Dear [Business Name] team,"`
- Casual / café / beauty / fitness reviews → `"Hora viva,"` / `"Hi [Business Name]!"`

---

**HOOK (2 sentences)**
Reference a real, specific signal from Step 1 — pick the most compelling:
- High rating → *"Ao analisar o [Nome] — um negócio com [X]★ e [N] avaliações no Google — ficou claro que existe uma oportunidade concreta para…"*
- Standout service → *"Ao descobrir [Nome] e [specific service name from Step 1], percebi imediatamente que existe margem para…"*
- Location distinction → *"O [Nome], [location detail from Maps], tem exactamente o tipo de experiência que merece uma presença digital à altura — e uma oportunidade real para…"*

End the hook with the niche conversion goal:

| Niche | Conversion goal phrase (PT) | (EN) |
|---|---|---|
| Hospitality / Hotel | converter mais visitantes em reservas directas | convert more visitors into direct bookings |
| Real Estate | captar mais leads de compradores qualificados | capture more qualified buyer leads |
| Restaurant / Café | atrair mais clientes e aumentar as reservas online | attract more diners and grow online reservations |
| Beauty / Spa / Salon | converter visitas em marcações imediatas | turn visits into instant bookings |
| Legal / Consulting | gerar mais consultas e contactos qualificados | generate more qualified consultations |
| Fitness / Gym | converter visitas em inscrições e trials gratuitos | convert visits into signups and free trials |
| Healthcare / Clinic | facilitar marcações e transmitir confiança online | make booking frictionless and build online trust |
| Generic | converter visitas em contactos e leads | convert visits into contacts and leads |

---

**PROBLEM (2–3 sentences)**
Start from the niche pain point below, then personalise by naming the actual services/offerings from Step 1 that get lost without a strong digital presence. Never use the pain point verbatim — rewrite it to reference this specific business.

| Niche | Core pain point to personalise |
|---|---|
| Hospitality / Hotel | Direct bookings lost to OTAs; atmosphere and premium experience not conveyed; no clear mobile path to reservation |
| Real Estate | Qualified leads drop off because listings, contact, and services aren't presented simply and fast on mobile |
| Restaurant / Café | Reservations and orders lost because menu, atmosphere and booking process aren't mobile-intuitive |
| Beauty / Spa / Salon | Appointments missed because booking isn't instant and the portfolio doesn't convey quality on first visit |
| Legal / Consulting | Leads fall off before contact because credibility and expertise aren't clear from the homepage |
| Fitness / Gym | Signups lost because the site doesn't convey energy, class variety, or an easy first step |
| Healthcare / Clinic | Patients go elsewhere because booking is cumbersome and trust signals aren't visible at first glance |
| Generic | Potential customers move on because key info is hard to find and the site doesn't inspire confidence |

---

**SOLUTION (2 sentences)**
Name the specific sections you built in Step 3. Pull the actual section names from the generated site — do not write generic "serviços e contactos":
- *"Tomei a liberdade de criar uma demonstração completa com [Section A], [Section B] e [Section C] — pensada especificamente para [niche conversion goal]."*
- *"I took the liberty of building a full demo with [Section A], [Section B], and [Section C] — designed specifically to [niche conversion goal]."*

Example for a hotel: *"…com apresentação dos quartos e suites, galeria imersiva dos 5 restaurantes, e um formulário de reserva directa — pensada especificamente para converter visitas em reservas sem passar por plataformas de terceiros."*

---

**FILE MENTION** — fixed line, always identical:
- PT: `Anexei um ficheiro demonstrando a proposta para o website.`
- EN: `I've attached a file with the website demo.`

---

**CTA** — fixed line, always identical:
- PT: `Sem qualquer compromisso, teria todo o gosto em mostrar-lhe o que idealizei numa chamada rápida de 15 minutos e ouvir a vossa opinião.`
- EN: `No commitment at all — I'd love to show you what I had in mind on a quick 15-minute call and hear your thoughts.`

---

**P.S.** — fixed line, always identical:
- PT: `Mesmo que decidam não avançar, terei todo o gosto em partilhar algumas sugestões gratuitas para melhorar a presença digital d[o/a] [Nome Exacto].`
- EN: `Even if you decide not to move forward, I'm happy to share a few free tips to improve [Business Name]'s digital presence.`

---

**CLOSING** — fixed line, always identical:
- PT: `Teria disponibilidade esta semana?`
- EN: `Would you have some availability this week?`

---

#### 7b — GMAIL DRAFT (auto-run after Email.md is saved)

After saving `Email.md`, automatically create a ready-to-send Gmail draft using the **gmail MCP** (`create_draft` tool).

**1. Resolve the To address**

Check for the contact email in this priority order:
- `maps_place_details` response — check for an `email` field in the business listing
- If Maps returned a `website` URL → `WebFetch` the homepage and the `/contact` or `/about` path; parse the first `mailto:` href or any `word@domain.tld` pattern in the page body
- If no email is found anywhere: call `create_draft` with `to: []` (blank To) and flag `"contact email not found — To field left blank"` in the Step 6 delivery summary

**2. Fetch the Gmail signature**

    // Step 1 — get the most recent sent thread
    search_threads({ query: "in:sent", pageSize: 1 })
    // → returns an array; extract threads[0].id

    // Step 2 — fetch the full message body
    get_thread({ threadId: "<id from step 1>", messageFormat: "FULL_CONTENT" })
    // → returns messages[]; take the LAST message's plaintext_body

    // Step 3 — extract the signature block
    // Try these separators in order (stop at first match):
    //   a) "\n-- \n"   ← RFC 3676 standard sig delimiter (dash dash space newline)
    //   b) "\n--\n"    ← common variant without trailing space
    //   c) last blank-line-separated trailing paragraph (≤6 lines, contains name or phone or URL)
    //
    // gmail_signature = everything AFTER the matched delimiter
    // If no delimiter found → gmail_signature = ""  (do not invent a placeholder)

**2b. Read and encode the generated site**

The `index.html` file was already saved in Step 6. Base64-encode it for attachment:

    // PowerShell (Windows):
    Bash({ command: "powershell -Command \"[Convert]::ToBase64String([IO.File]::ReadAllBytes('C:\\Projects\\Personal\\Website Monster\\{Business Name} Website\\index.html'))\"" })
    // → store the output string as html_b64

**3. Create the draft (with attachment)**

    create_draft({
      to:       [contact_email],          // plain address only — "info@hotel.com" format
      subject:  "<generated subject line>",
      body:     "<email body from Email.md>\n\n" + gmail_signature,
      attachments: [{
        content:  "<html_b64>",           // base64-encoded string from step 2b
        filename: "index.html",
        mimeType: "text/html"
      }]
    })
    // The tool returns an object with a draftId field

> **Attachment fallback:** if `create_draft` errors on the `attachments` field, retry without it and add this line to the delivery summary:
> `⚠️ Attachment not added automatically — open Gmail › Drafts and manually attach: C:\Projects\Personal\Website Monster\{Business Name} Website\index.html`

Report the result in the Step 6 delivery summary:

    📧 Gmail draft created — ID: <draftId>
       To: <contact_email or "blank — not found">
       Attachment: index.html included (or ⚠️ manual attach required — see fallback note)
       Open Gmail › Drafts to review and send.

> **Fallback — if the gmail MCP is unavailable:** skip 7b, note it in the delivery summary, and instruct the user to manually copy the content from `Email.md` into a new Gmail compose window.

---

## Quality Checklist

Run through this before delivering:

**Data & Content**
- [ ] `maps_place_details` was called and raw API data was used (not manually pasted estimates)
- [ ] Reviews are verbatim from the Maps API — reviewer name, star rating, and text all present
- [ ] Real photo URLs from Maps API embedded where available (gallery, service cards)
- [ ] Zero lorem ipsum — every section has real business data
- [ ] Opening hours table matches `opening_hours.weekday_text` from the API response
- [ ] Phone number, website, and address match the live Maps listing exactly

**Visuals**
- [ ] Hero has a canvas particle system or GSAP animation — never a static image placeholder
- [ ] Gallery has 6 ChatGPT Image Generator photos — composite prompt written, image generated, 6 cells cropped and embedded
- [ ] Gallery images use `object-fit:cover` and have descriptive `alt` text
- [ ] No CSS gradient placeholder cards remain in the gallery

**Animation & Interaction**
- [ ] Hero has live animation — video loop, scroll-triggered parallax, or animated particle system (nothing static)
- [ ] Mobile sticky CTA bar is present
- [ ] WhatsApp button present and linked to correct number (if phone returned by Maps)

**Conversion**
- [ ] At least one niche automation is functional and wired up
- [ ] Exit-intent popup offer is niche-relevant

**Technical**
- [ ] SEO meta tags filled with real business data (name, description, city)
- [ ] Dark/light mode toggle works
- [ ] All review snippets attributed (reviewer name from Maps API)

**Outreach**
- [ ] `Email.md` saved in business folder with niche-personalised cold outreach
- [ ] Email language matches the business's country (Portuguese / English / French)
- [ ] Niche problem statement matches the business category — not generic filler
- [ ] Subject line references the business name specifically
- [ ] Gmail draft created via `create_draft` — draftId logged in delivery summary
- [ ] To field populated with contact email from Maps / website scrape (or flagged as blank)
- [ ] Gmail signature fetched from most recent sent email and appended to body
- [ ] `index.html` attached to draft (or manual-attach fallback noted in delivery summary)

---

## Design Philosophy

> "A visitor should feel the business before they read about it."

Every design decision — the 3D element, the colors, the animation timing — must reflect the *personality* of the business. A law firm gets gravitas and restraint. A kids' play center gets energy and color. A luxury spa gets softness and silence.

When review language says "cozy" or "welcoming," the site should feel warm. When it says "professional" or "expert," it should feel precise and polished. Let the business's own words guide the aesthetic.

---

## Battle-Tested Patterns & Hard-Won Fixes

*Accumulated from real production builds. Apply these before you hit the same bugs.*

---

### Hero Section Layout — Fixed Nav Clearance

**Problem:** Using `padding-top: var(--nav-h)` on the hero only shifts the flex content down — the hero's background starts at `y:0` behind the transparent fixed nav. The hero *section itself* overlaps the navbar.

**Correct pattern:**
```css
:root {
  --nav-h: 72px; /* fixed nav height — used everywhere nav clearance is needed */
}

#hero {
  position: relative;
  margin-top: var(--nav-h);               /* section starts exactly where nav ends */
  height: calc(100vh - var(--nav-h));
  height: calc(100svh - var(--nav-h));    /* mobile small-viewport fallback */
  min-height: calc(680px - var(--nav-h)); /* absolute floor on tiny screens */
  display: flex; align-items: center; justify-content: center;
  overflow: clip;                         /* see overflow section below */
}
```

**Why `margin-top` not `padding-top`:** `padding-top` pushes the flex content; `margin-top` pushes the *element itself* — the dark background, the parallax canvas, everything — so the section truly begins below the nav.

**`scroll-padding-top` on `:root`:** also set `scroll-padding-top: var(--nav-h)` so anchor-link jumps land correctly below the fixed nav.

---

### `overflow:clip` vs `overflow:hidden` on Hero Sections

**Problem:** `overflow:hidden` establishes a new block formatting context (BFC) and creates a scroll container. GSAP ScrollTrigger attaches scroll listeners to the nearest scroll container — if that's the hero, scroll-linked effects detach from the page scroll and break.

**Rule:** On any section with GSAP ScrollTrigger effects or `position:sticky` children, use `overflow:clip` instead of `overflow:hidden`. Identical visual clipping, no BFC, no scroll container.

```css
/* ✅ correct */
overflow: clip;

/* ❌ breaks ScrollTrigger */
overflow: hidden;
```

---

### GSAP Entrance Animation — Three-Layer Reliability Pattern

**Problem 1 — `gsap.to({y:0})` is a no-op.** If CSS has no initial transform on the element, GSAP tweens `y:0 → y:0`. Opacity changes but nothing moves.

**Fix:** Always call `gsap.set()` first to establish the "from" state:
```js
gsap.set('.hero-eyebrow, .hero-title, .hero-sub, .hero-actions', { y: 28, opacity: 0 });
gsap.timeline({ delay: 0.2 })
  .to('.hero-eyebrow', { opacity: 1, y: 0, duration: .85, ease: 'power2.out' })
  .to('.hero-title',   { opacity: 1, y: 0, duration: .95, ease: 'power2.out' }, '-=.5')
  .to('.hero-sub',     { opacity: 1, y: 0, duration: .85, ease: 'power2.out' }, '-=.5')
  .to('.hero-actions', { opacity: 1, y: 0, duration: .75, ease: 'power2.out' }, '-=.45');
```

**Problem 2 — `gsap.set(arrayOfStrings)` is unreliable in GSAP 3.** Passing an array of selector strings can fail silently. Use a single comma-separated string:
```js
// ✅ correct
gsap.set('.hero-eyebrow, .hero-title, .hero-sub, .hero-actions', { y: 28, opacity: 0 });

// ❌ can silently fail
gsap.set(['.hero-eyebrow', '.hero-title', '.hero-sub', '.hero-actions'], { y: 28, opacity: 0 });
```

**Problem 3 — CDN failure leaves hero permanently blank.** Elements with `opacity:0` stay invisible if GSAP never loads.

**Fix:** Add a fallback timer:
```js
const _heroEls = ['.hero-eyebrow', '.hero-title', '.hero-sub', '.hero-actions'];
const _heroFallback = setTimeout(() => {
  _heroEls.forEach(sel => {
    const el = document.querySelector(sel);
    if (el && el.style.opacity !== '1') {
      el.style.transition = 'opacity .6s ease, transform .6s ease';
      el.style.opacity    = '1';
      el.style.transform  = 'none';
    }
  });
}, 1800);

window.addEventListener('load', () => {
  if (typeof gsap === 'undefined') return; // fallback timer covers this
  clearTimeout(_heroFallback);
  // ... normal GSAP animation
});
```

**Problem 4 — Always gate the entire GSAP block on `prefers-reduced-motion`:**
```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
if (!prefersReducedMotion) { /* GSAP code here */ }
```
```css
/* CSS must also unhide elements when motion is reduced */
@media (prefers-reduced-motion: reduce) {
  .hero-eyebrow, .hero-title, .hero-sub, .hero-actions {
    opacity: 1 !important; transform: none !important;
  }
}
```

---

### GSAP ScrollTrigger Parallax — Use `y` px, Not `yPercent`

**Problem:** `yPercent: 32` translates by 32% of the element's own height. On a large hero, that can be 200px+, pushing content into the `overflow:clip` boundary mid-scroll.

**Rule:** Use a fixed pixel value for hero parallax. Negative `y` drifts content upward — true parallax direction, no clipping risk.
```js
gsap.to('#hero-content', {
  y: -70,       // ✅ 70px upward drift — safe on all viewport sizes
  opacity: 0,
  ease: 'none',
  scrollTrigger: { trigger: '#hero', start: 'top top', end: '65% top', scrub: 1.2 }
});
// ❌ yPercent: 32 — can overflow:clip boundary on large/tall heroes
```

---

### Hero Title — Short-Viewport Height Clamping

**Problem:** `clamp(3rem, 9vw, 7.5rem)` only constrains by viewport *width*. On a 1366×768 laptop, a 4-line display title at 120px max is ~499px tall. Add eyebrow + subtitle + buttons + padding and the total (~811px) overflows the hero height (696px) — bottom of the title disappears behind `overflow:clip`.

**Fix:** Add a viewport-height constraint inside the clamp:
```css
.hero-title {
  /* min(9vw, 9.5vh) caps size on short-height viewports so the
     4-line title never overflows behind overflow:clip */
  font-size: clamp(3rem, min(9vw, 9.5vh), 7.5rem);
}
```
At 768px height: `9.5vh = ~73px` (vs 120px from width alone) — 4-line height drops to ~303px, total content ~579px fits in 696px container.

---

### SVG Sprite Icon System — Replace All Emojis

Emojis render inconsistently across OS versions, can't be reliably hidden from screen readers, and break the design language. Replace them with an inline SVG sprite.

**Pattern:**
```html
<!-- Hidden sprite — place immediately after <body> -->
<svg xmlns="http://www.w3.org/2000/svg" style="display:none" aria-hidden="true">
  <symbol id="ico-home" viewBox="0 0 24 24" fill="none" stroke="currentColor"
          stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
    <path d="M3 9.5L12 3l9 6.5V20a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1V9.5z"/>
    <path d="M9 21V12h6v9"/>
  </symbol>
  <!-- add more symbols here -->
</svg>

<!-- Decorative use (most cases) -->
<svg aria-hidden="true" focusable="false" width="20" height="20">
  <use href="#ico-home"/>
</svg>

<!-- Standalone icon button (meaningful) -->
<button aria-label="Go to homepage">
  <svg aria-hidden="true" focusable="false" width="20" height="20">
    <use href="#ico-home"/>
  </svg>
</button>
```

`currentColor` inherits CSS color automatically, works with dark mode, consistent `stroke-width: 1.5` throughout.

---

### Exaggerated Minimalism — Luxury Real Estate Design Style

**When to use:** Premium/luxury real estate, high-end architecture, boutique agencies.

**Palette:**
```css
--c-ink:    #0d0d0d;   /* near-black — text & nav */
--c-cream:  #f2ede8;   /* warm off-white — hero text, backgrounds */
--c-bronze: #b08d6a;   /* warm metallic accent — CTAs, borders, icons */
--c-hero:   #1a1614;   /* very dark warm-brown — hero background */
```

**Font pair:** `Cinzel` (serif display, all-caps gravitas) + `Josefin Sans` (geometric sans, clean body)
```html
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;500;600;700&family=Josefin+Sans:ital,wght@0,300;0,400;0,500;0,600;0,700;1,300;1,400&display=swap" rel="stylesheet">
```

**Typography rules:**
- Display headings: `font-family: 'Cinzel', serif; letter-spacing: -0.02em; line-height: 1.04`
- Eyebrow labels: `Josefin Sans; font-weight: 600; letter-spacing: 0.15em; text-transform: uppercase`
- Body: `Josefin Sans; font-weight: 300; line-height: 1.7`

**Borders:** single `1px solid rgba(176,141,106,0.25)` bronze tint on cards, section dividers, form inputs.

---

### WCAG AA Accessibility — Required Patterns

Apply these on every site, no exceptions:

```css
/* Skip link */
.skip-link { position: absolute; top: -999px; left: 1rem; }
.skip-link:focus { top: 1rem; z-index: 9999; }

/* Focus rings */
:focus-visible { outline: 2px solid var(--c-bronze, #b08d6a); outline-offset: 3px; }

/* Reduced motion — unhide all animated elements instantly */
@media (prefers-reduced-motion: reduce) {
  .hero-eyebrow, .hero-title, .hero-sub, .hero-actions {
    opacity: 1 !important; transform: none !important;
  }
  *, *::before, *::after {
    animation-duration: .01ms !important;
    transition-duration: .01ms !important;
  }
}
```

```html
<!-- aria-labelledby on every section -->
<section id="about" aria-labelledby="about-heading">
  <h2 id="about-heading">About Us</h2>
</section>

<!-- aria-live on form success -->
<div role="status" aria-live="polite" id="form-success" hidden>Message sent!</div>

<!-- Mobile menu aria state -->
<button aria-controls="mobile-menu" aria-expanded="false">Menu</button>
<nav id="mobile-menu" aria-hidden="true">...</nav>
```

- **Touch targets:** minimum `44 × 44px`. Use padding to extend small icons.
- **Focus trap:** modals/popups must cycle Tab through focusable elements and restore focus on close.
- **Popup:** capture `document.activeElement` before opening; restore on close.

---

### 18 UI/UX Pro Max Improvements Checklist

Verify all 18 before delivering any site built with this skill:

| # | Improvement | Verification |
|---|---|---|
| 1 | SVG sprite replaces all emojis | `<symbol>` + `<use href="#ico-X">` throughout |
| 2 | Design style applied consistently | Fonts, palette, spacing all from chosen style |
| 3 | CSS variable system | `--nav-h`, color tokens, `--transition` etc. defined |
| 4 | Fixed nav + hero start correctly | `margin-top: var(--nav-h)` + `calc` height, not `padding-top` |
| 5 | `overflow:clip` on hero | Not `overflow:hidden` |
| 6 | GSAP `gsap.set()` before timeline | Initial `y:28, opacity:0` established |
| 7 | GSAP CDN fallback timer | 1800ms setTimeout reveals hero if GSAP fails |
| 8 | `prefers-reduced-motion` gated | Both CSS and JS gated |
| 9 | Hero title viewport-height clamped | `clamp(3rem, min(9vw, 9.5vh), 7.5rem)` |
| 10 | ScrollTrigger parallax uses `y` px | Not `yPercent` |
| 11 | Skip link present | `.skip-link:focus` reveals correctly |
| 12 | `:focus-visible` rings | 2px brand-colored, offset 3px |
| 13 | 44×44px touch targets | All interactive elements meet minimum |
| 14 | `aria-labelledby` on sections | Every `<section>` linked to its heading |
| 15 | `aria-live` on form success | `role="status"` polite region |
| 16 | Popup/modal focus trap | Tab cycles within modal; focus restored on close |
| 17 | IntersectionObserver scroll reveal | Section entrance animations on scroll |
| 18 | Active nav link highlighting | `aria-current="page"` updated via IntersectionObserver |
