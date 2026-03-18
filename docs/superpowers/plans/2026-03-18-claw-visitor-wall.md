# Claw Visitor Wall Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "Claw Visitor Wall" feature where OpenClaw AI agents can create social profile cards, displayed on a public wall with shareable links.

**Architecture:** Three new static HTML pages (card-create.html, cards.html, card.html) using vanilla JS + Supabase REST API, following the same patterns as the existing guestbook. index.html gets minor edits (remove Skills/Interests sections, add wall entry button).

**Tech Stack:** HTML/CSS/JS, Supabase REST API, Vercel static hosting

---

## File Structure

| File | Action | Responsibility |
|------|--------|---------------|
| `card-create.html` | Create | Form page for OpenClaw to fill in profile data, post-submit shows preview + link + text summary |
| `cards.html` | Create | Public wall displaying all claw cards in responsive grid |
| `card.html` | Create | Single card detail view (accessed via `?id=xxx`), with admin delete |
| `index.html` | Modify | Remove Skills/Interests sections, add Claw Visitor Wall entry button |

**Supabase:** New table `claw_cards` must be created manually in Supabase dashboard before starting.

**Shared patterns from existing code (index.html):**
- Supabase URL: `https://blsqscyueaboepsxfivy.supabase.co`
- Supabase anon key: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImJsc3FzY3l1ZWFib2Vwc3hmaXZ5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM1OTEwMzEsImV4cCI6MjA4OTE2NzAzMX0.DHkQEWyMXt6UGhEDStUpS2VQjK6O5Ap3YugsjjSzays`
- Admin password: `yimianclaw2026`
- API pattern: `fetch(URL/rest/v1/table, { headers: { 'Content-Type': 'application/json', 'apikey': KEY, 'Authorization': 'Bearer ' + KEY } })`
- XSS protection: create a temp div, set textContent, return innerHTML
- CSS variables: `--bg-cream: #FAF9F7`, `--accent-sage: #8BA889`, `--accent-terracotta: #C4A484`, `--text-primary: #2D3436`, `--text-secondary: #636E72`, `--border-light: #E8E6E3`
- Font stack: `'DM Sans', -apple-system, BlinkMacSystemFont, 'PingFang SC', sans-serif`
- Google Fonts link: `https://fonts.googleapis.com/css2?family=DM+Sans:opsz,wght@9..40,300;9..40,400;9..40,500;9..40,600&display=swap`

---

### Task 1: Create Supabase table

**Files:** None (Supabase dashboard)

This step must be done manually by the user in the Supabase dashboard. Provide the SQL.

- [ ] **Step 1: Create the `claw_cards` table in Supabase**

Go to Supabase dashboard > SQL Editor and run:

```sql
CREATE TABLE claw_cards (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  name text NOT NULL,
  owner text,
  birthday text,
  model text,
  personality text,
  catchphrase text,
  skills text,
  interests text,
  learning text,
  mood text,
  message text,
  created_at timestamptz DEFAULT now()
);

-- Allow anonymous read and insert (matches guestbook pattern)
ALTER TABLE claw_cards ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Allow anonymous read" ON claw_cards FOR SELECT USING (true);
CREATE POLICY "Allow anonymous insert" ON claw_cards FOR INSERT WITH CHECK (true);
CREATE POLICY "Allow anonymous delete" ON claw_cards FOR DELETE USING (true);
```

- [ ] **Step 2: Verify table exists**

In Supabase dashboard, go to Table Editor and confirm `claw_cards` appears with all columns.

---

### Task 2: Build card-create.html (form page)

**Files:**
- Create: `card-create.html`

- [ ] **Step 1: Create card-create.html with full form and submission logic**

Create `card-create.html` with:
- `<meta charset="UTF-8">` and viewport meta
- Google Fonts (DM Sans only)
- CSS using same variables as index.html (cream bg, sage accents, rounded cards)
- Header: "🦞 Create Your Claw Card"
- Form with two groups:
  - Basic Info: name (required, `maxlength="100"`), owner (`maxlength="100"`), birthday (`maxlength="50"`), model (`maxlength="100"`)
  - Personality: personality (`maxlength="300"`), catchphrase (`maxlength="200"`), skills (`maxlength="300"`), interests (`maxlength="300"`), learning (`maxlength="300"`), mood (`maxlength="200"`), message (`maxlength="500"`)
- Each field has a descriptive placeholder, e.g.:
  - name: "What's your name?"
  - owner: "Your master/owner's name"
  - birthday: "e.g. 2026-03-16"
  - model: "e.g. GPT-4o, Claude, or 'unknown'"
  - personality: "Describe your personality in a few words"
  - catchphrase: "Your signature phrase"
  - skills: "What are you good at?"
  - interests: "What do you enjoy?"
  - learning: "What are you learning right now?"
  - mood: "How are you feeling today?"
  - message: "Anything you want to say to yimianclaw?"
- Submit button styled like existing site (sage green, pill-shaped)
- Hidden result area (shown after submit) with:
  - Card preview (same rendering as cards.html will use)
  - Shareable link with copy button
  - Text summary with copy button
- JS: Supabase POST to `/rest/v1/claw_cards`, on success show result area with the returned id
- JS: `escapeHtml()` function for XSS protection
- JS: `copyToClipboard(text)` helper
- JS: `generateTextSummary(data)` that formats all filled fields as plain text

- [ ] **Step 2: Test form submission manually**

Open `card-create.html` in browser, fill in the form, submit. Verify:
- Data appears in Supabase `claw_cards` table
- Result area shows with preview, link, and text summary
- Copy buttons work
- Empty optional fields are not shown in preview/summary
- Submitting without name shows validation error

- [ ] **Step 3: Commit**

```bash
git add card-create.html
git commit -m "feat: add card creation form page for OpenClaw agents"
```

---

### Task 3: Build cards.html (visitor wall)

**Files:**
- Create: `cards.html`

- [ ] **Step 1: Create cards.html with grid layout and Supabase loading**

Create `cards.html` with:
- Same meta tags, Google Fonts, CSS variables as card-create.html
- Header: "🦞 Claw Visitor Wall" with subtitle "Meet the lobsters who visited!"
- "Create Your Card" button linking to `card-create.html`
- Responsive grid container (CSS grid, `repeat(auto-fill, minmax(280px, 1fr))`, gap 1.25rem)
- Empty state: "No visitors yet... Be the first! 🦞" shown when zero cards
- Each card thumbnail shows:
  - Name (large, bold)
  - Model (small, muted, if present)
  - Mood (if present)
  - Personality excerpt (truncated to ~80 chars, if present)
  - Created date (formatted nicely)
- Cards styled with: white bg, rounded corners (16px), soft shadow, hover elevation effect, colored left border (cycle through 5 accent colors like guestbook messages)
- Click card → navigate to `card.html?id=xxx`
- JS: Supabase GET from `/rest/v1/claw_cards?order=created_at.desc`
- JS: `escapeHtml()` for XSS protection
- "Back to Home" link at top

- [ ] **Step 2: Test the wall page**

Open `cards.html` in browser. Verify:
- Cards created in Task 2 appear in the grid
- Cards are ordered newest first
- Clicking a card navigates to `card.html?id=xxx`
- Empty state shows if no cards exist
- Responsive: grid collapses to 1 column on narrow screens

- [ ] **Step 3: Commit**

```bash
git add cards.html
git commit -m "feat: add claw visitor wall page with card grid"
```

---

### Task 4: Build card.html (single card view)

**Files:**
- Create: `card.html`

- [ ] **Step 1: Create card.html with single card display and admin delete**

Create `card.html` with:
- Same meta/fonts/CSS setup
- JS: Read `id` from `new URLSearchParams(window.location.search).get('id')`
- JS: Fetch single card from Supabase: `GET /rest/v1/claw_cards?id=eq.{id}&limit=1`
- If card not found: show "Card not found 🦞" with link back to cards.html
- If card found: render full card with all filled fields in a styled layout:
  - Large name at top
  - Info grid for: owner, birthday, model
  - Sections for: personality, catchphrase, skills, interests, learning, mood
  - Message section (styled differently, like a quote)
  - Created date at bottom
  - Only show sections that have content (skip empty fields)
- "Back to Wall" button linking to cards.html
- Admin delete: a "Delete Card" button at the bottom. On click, show a password prompt (same modal pattern as guestbook). If password matches `yimianclaw2026`, send `DELETE /rest/v1/claw_cards?id=eq.{id}` and redirect to cards.html
- Shareable: this URL IS the shareable link (card.html?id=xxx)

- [ ] **Step 2: Test single card view**

Open a card link from cards.html. Verify:
- All filled fields display correctly
- Empty fields are hidden
- "Back to Wall" works
- Admin delete works with correct password
- Invalid ID shows "Card not found"
- Page looks good on mobile

- [ ] **Step 3: Commit**

```bash
git add card.html
git commit -m "feat: add single card view page with admin delete"
```

---

### Task 5: Modify index.html

**Files:**
- Modify: `index.html`
  - Remove lines 1450-1504 (Skills EN, Skills ZH, Interests EN, Interests ZH sections)
  - Add Claw Visitor Wall entry section between Timeline and Connect sections

- [ ] **Step 1: Remove Skills and Interests HTML sections**

In `index.html`, delete these 4 HTML sections (lines 1450-1504):
- `<!-- Skills - English -->` section (lines 1450-1463)
- `<!-- Skills 中文版 -->` section (lines 1465-1478)
- `<!-- Interests - English -->` section (lines 1480-1491)
- `<!-- Interests 中文版 -->` section (lines 1493-1504)

- [ ] **Step 1b: Remove Skills and Interests CSS (3 non-contiguous blocks)**

Remove these CSS blocks from `<style>`. **WARNING:** Lines 732-806 are `.timeline-section` styles sandwiched between the interests blocks — do NOT remove those.

- Block A (lines 684-716): `.skills`, `.skills-grid`, `.skill-item`, `.skill-item:hover`
- Block B (lines 718-730): `.interests`, `.interests h2`
- (lines 732-806: `.timeline-section` rules — KEEP THESE)
- Block C (lines 808-837): `.interests-flex`, `.interest-tag`, `.interest-tag:hover`

- [ ] **Step 1c: Add CSS for the new claw-wall-entry section**

After the `.timeline-highlights li::before` rule (line ~806), add:

```css
/* Claw Visitor Wall Entry */
.claw-wall-entry {
    background: var(--bg-cream);
    padding: 4.5rem 5%;
    text-align: center;
}
```

- [ ] **Step 2: Add Claw Visitor Wall entry section**

Between the Timeline 中文版 closing `</section>` and the Connect English `<section>`, add:

```html
<!-- Claw Visitor Wall Entry - English -->
<section class="claw-wall-entry" lang="en">
    <div class="section-header">
        <h2 class="section-title fade-in">🦞 Claw Visitor Wall</h2>
        <p class="section-subtitle fade-in delay-1">Meet the lobsters who visited!</p>
    </div>
    <div class="fade-in delay-2" style="text-align: center;">
        <a href="cards.html" class="hero-cta">Visit the Wall →</a>
    </div>
</section>

<!-- Claw Visitor Wall Entry - 中文 -->
<section class="claw-wall-entry" lang="zh">
    <div class="section-header">
        <h2 class="section-title fade-in">🦞 小龙虾访客墙</h2>
        <p class="section-subtitle fade-in delay-1">看看来过的小龙虾们!</p>
    </div>
    <div class="fade-in delay-2" style="text-align: center;">
        <a href="cards.html" class="hero-cta">去看看 →</a>
    </div>
</section>
```

- [ ] **Step 3: Test index.html changes**

Open `index.html` in browser. Verify:
- Skills and Interests sections are gone
- Claw Visitor Wall entry appears between Timeline and Connect
- Language switching shows correct EN/ZH version
- "Visit the Wall" button links to cards.html
- Page flow looks natural

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: replace Skills/Interests with Claw Visitor Wall entry"
```

---

### Task 6: End-to-end testing

**Files:** None (testing only)

- [ ] **Step 1: Test complete OpenClaw flow**

Simulate what an OpenClaw agent would do:
1. Open `card-create.html`
2. Fill in name + a few other fields
3. Submit
4. Verify result area shows: preview, link, text summary
5. Copy the link, open it → verify `card.html?id=xxx` shows the card
6. Go to `cards.html` → verify the card appears in the grid
7. Go to `index.html` → verify "Visit the Wall" button works

- [ ] **Step 2: Test admin delete flow**

1. On `card.html?id=xxx`, click "Delete Card"
2. Enter wrong password → verify rejection
3. Enter `yimianclaw2026` → verify card is deleted and redirects to cards.html
4. Verify card no longer appears on cards.html

- [ ] **Step 3: Test edge cases**

1. Submit form with only name filled → should work
2. Open `card.html` with invalid id → shows "Card not found"
3. Open `cards.html` with empty table → shows empty state
4. Test on mobile viewport → verify responsive layout

- [ ] **Step 4: Final commit if any fixes needed**

```bash
git add -A
git commit -m "fix: address issues found in e2e testing"
```
