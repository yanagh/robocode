---
name: robocode-carousel
description: >
  Generate Facebook carousel posts for Robocode — a children's robotics and
  programming school in Tbilisi, Georgia. Use this skill whenever the user asks
  to create a carousel, social media post, Facebook slides, or any visual
  content for Robocode. Also trigger when the user provides text and says
  "make a carousel", "create slides", "post for Facebook", or similar. Always
  apply Robocode brand guidelines automatically — never ask for brand details,
  they are embedded in this skill.
---

# Robocode Facebook Carousel Generator

This skill generates a single HTML file containing all carousel slides.
The user screenshots each slide manually.

---

## Brand Reference

**Colors**
- Primary Blue: `#1646FF`
- Green: `#33F981`
- Orange: `#FF7939`
- Yellow: `#E5FB6D`
- White: `#FFFFFF`

**Fonts** (use Google Fonts in HTML)
- Primary: Inter
- Alternative: Montserrat

**Tone**
- Simple, clear, positive, educational
- Written for parents of children aged 7–16
- No corporate jargon, no complex language
- Focus on what children will BUILD and CREATE

**Logo**
- Path: `assets/logo.png` (relative to repo root)
- Always place logo on last slide (CTA slide)

**Language**
- ALL text on slides must be in **Georgian only** — proper Georgian Unicode characters (ა ბ გ დ ე...)
- **NEVER use Cyrillic/Russian characters** under any circumstances — not even as a transliteration of Georgian words
- No English text on slides (except brand name "Robocode", level names like "Arduino Kids", "Pro Embedded", and tech terms like "IoT", "AI")

---

## Slide Structure (Classic)

Every carousel follows this structure:

| Slide | Purpose | Notes |
|-------|---------|-------|
| 1 | **Hook** | Bold statement or question. Large text. Grabs attention. |
| 2–N | **Content slides** | One idea per slide. Short text. Supporting point or tip. |
| Last | **CTA** | "რეგისტრაცია უფასო საცდელ გაკვეთილზე" + link + logo |

Number of content slides (2–N) is flexible — based on how much content the user provides.

---

## Images

**If the user provides photos:**
- Use them directly as slide backgrounds or side visuals
- Reference them by filename from the repo `assets/` folder

**If no photos are provided:**
- Generate images using **Replicate API** (model: `black-forest-labs/flux-schnell`)
- Image requirements:
  - Realistic, clean, modern photography style
  - Children must appear to be of **Georgian nationality** (dark hair, olive skin tones)
  - Subject: children coding, building robots, using computers, presenting projects
  - No text in the generated images
  - Bright, well-lit environments
- Save generated images to `assets/generated/`

**Replicate API call example (Node.js):**
```javascript
import Replicate from "replicate";
const replicate = new Replicate({ auth: process.env.REPLICATE_API_TOKEN });

const output = await replicate.run("black-forest-labs/flux-schnell", {
  input: {
    prompt: "Georgian child aged 10-12 building a small robot at a classroom desk, realistic photo, bright modern classroom, clean background, no text",
    num_outputs: 1,
    aspect_ratio: "1:1",
  }
});
// output[0] is the image URL — download and save to assets/generated/
```

---

## HTML Output Spec

Generate a single `carousel.html` file.

**Slide dimensions:** 1080×1080px (standard Facebook square carousel)

**Structure:**
```html
<!-- One .slide div per slide, all visible and stacked vertically -->
<!-- User scrolls down and screenshots each one -->
<div class="slide" id="slide-1"> ... </div>
<div class="slide" id="slide-2"> ... </div>
<!-- etc -->
```

**Design rules:**
- Each slide is exactly 1080×1080px
- Use brand colors — blue as dominant, accent with green/orange/yellow
- Font: Inter (import from Google Fonts)
- Text: bold headlines, large and readable
- Keep text minimal — 1 headline + max 2–3 short lines per slide
- Modern, energetic feel — not corporate
- Slide 1 (hook): Large bold text, strong color background, no image needed
- Content slides: can use image on one side + text on other, OR full background image with text overlay
- CTA slide: Blue background, white text, logo, registration link, phone number

**CTA slide always includes:**
```
რეგისტრაცია უფასო საცდელ გაკვეთილზე
+995 591 691 749   ← use phone number, NOT a form link (links don't work in screenshots)
პეკინის 34, თბილისი
[Robocode logo]
```

---

## Step-by-Step Workflow

1. **Read the user's text/topic** for the carousel
2. **Check if photos were provided** in the repo `assets/` folder
3. **If no photos** → generate images via Replicate API (see above)
4. **Write Georgian text** for each slide following the Classic structure
5. **Generate `carousel.html`** following the HTML Output Spec
6. **Save to repo root** as `carousel.html`
7. **Tell the user:** open the file in a browser, scroll through slides, screenshot each one at 1080×1080px

---

## Example Prompt → Output Mapping

**User says:**
> "კარუსელი გაიმეორე Unity კურსის შესახებ"

**You generate:**
- Slide 1: Hook — e.g. "შენი შვილი შექმნის თავის პირველ თამაშს"
- Slide 2: What they learn in Unity course
- Slide 3: Age group + what project they build
- Slide 4: CTA — free trial registration

---

## Photo & Overlay Rules

- **Never make overlays too dark** — max `rgba(0,0,0,0.80)` opacity. Above that the photo disappears and the slide looks completely black.
- Good overlay pattern: `linear-gradient(to top, rgba(0,0,0,0.75) 35%, rgba(0,0,0,0.1) 70%, transparent 100%)`
- **Use direct CSS background on the slide element** — layer the gradient and photo as multiple `background-image` values. Do NOT use child `.bg` / `.overlay` divs, as they cause black rendering bugs with `container-type: size`:
  ```css
  #c2-s3 {
    background-image:
      linear-gradient(to top, rgba(0,0,0,0.75) 35%, transparent 100%),
      url("assets/photos/photo.jpeg");
    background-size: cover;
    background-position: center;
  }
  ```
- When a photo has a lot of blank/white space at the top, move text to the top of the card (`justify-content: flex-start`) and flip the gradient direction to `to bottom`.

---

## HTML Tech Notes

- Slide size: **350×350px** fixed (`width: 350px; height: 350px`) in a horizontal scrolling row
- Use `container-type: size` on `.slide` so internal sizes use `cqh` units (1cqh = 1% of card height = 3.5px)
- All internal font sizes, padding, spacing use `cqh` units
- Each carousel = one `<section class="carousel-section">` with a `.section-label` + `.slides-row`
- Section label visibility: use `color: rgba(255,255,255,0.7)` minimum — not `0.3` (too faint)

---

## Notes

- Never ask the user to re-explain the brand — it's all in this skill
- Always write slide text in Georgian
- Keep "Robocode" spelled exactly as-is (not translated)
- Phone never changes: `+995 591 691 749`
- Address never changes: `პეკინის 34, თბილისი`
- **Do not put links on slides** — links are not clickable in screenshots. Use phone number instead.
- If unsure how many slides to make — fewer is better. 4–6 slides is ideal.
