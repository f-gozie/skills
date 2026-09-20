---
name: mockup
description: Design a user-facing surface as an approval-ready HTML state board before any of it gets built, and set up the project's design system first if it has none. Use for "mockup", "design this", "what should it look like", "show me how it'd work", "give me a couple of options", "set up a design system", and for any request that adds or reworks a screen, component, sheet, overlay, empty/error state, or motion that carries meaning — including when that surface is buried inside a plan or a feature request that never says "design". Not for copy-only edits, backend or data work, building a design that is already approved and unchanged, fixing shipped UI that drifted from an approved design, screenshots of the app as it exists today, or marketing assets.
---

# Mockup

Turn a proposed visual change into a self-contained HTML board the user can open cold and approve or
reject without asking a question. The board is the spec: what gets built matches the approved board.
After approval it leaves a written record, so the decision is not reopened by accident and the build
can be checked against it.

Deliverable: one HTML file that opens with no build step and reads like a screenshot of the real app.

---

## Where things live

Everything this skill produces lives in the project, not in the skill folder. The skill is shared
across projects; the design decisions are not.

Default layout (use the project's existing one if it already has something similar — look first):

```
design/
  system.md        the design system: tokens, type, components, voice, rules
  mockups/         HTML boards, one per surface or round (YYYY-MM-DD-slug.html)
  approved/        one record per approved decision (slug.md)
```

If the project keeps these somewhere else, use that location and say where in your reply. Never
create a second design folder beside an existing one.

---

## Step 0: the design system

Before the first mockup in a project, check for a design system: `design/system.md`, or an existing
token file, theme module, Tailwind config, or style guide.

**If one exists,** read it and the code it points to. Go to the board.

**If none exists,** build one with the user before boarding any feature. Do not invent a system
silently and then mock screens in it.

1. **Read what is already there.** Existing CSS, theme files, component library, logo, any screens
   already built. A young app usually has an implied system — find it before proposing a new one.
2. **Ask a few short questions.** What the product is and who uses it; platforms (web, iOS, Android,
   desktop); light, dark, or both; three words for how it should feel; products they admire and
   products they don't want to look like; any fixed brand assets (logo, color, font).
3. **Board two or three directions** as one HTML file. Each direction shows the same small set of
   real surfaces from this product — for example a list screen, a form, a card, a dialog, an empty
   state — in every theme that will ship. Show palette, type scale, spacing, radius, elevation, and
   the core controls (primary and secondary button, input, nav control, card, list row). Recommend
   one and say why.
4. **Get an explicit choice.** The user may mix parts of directions; board the mix before locking it.
5. **Write `design/system.md`** from `references/design-system-template.md`. Record the directions
   that lost and why, like any other approval.
6. **Wire tokens into code** only if the user asks. The system file is the source until then.

The design system is a living document. When an approval changes a token or adds a component, update
`system.md` in the same step as the approval record.

---

## Gates

Five. They are not ranked against the quality bar below, and nothing may be improved at their
expense. A board that misses one has failed, however good it looks.

**1. Approval before implementation.** No product code for a visual change until the user says yes
in words. Under time pressure the *board* gets smaller — fewer frames, rougher composition. The
approval does not get skipped, and neither does the accessibility floor. Do not infer a waiver from
short replies or urgency. If the user explicitly overrides the gate, say plainly that what ships was
not design-approved.

**2. Accessibility floor, shown on the board.** Any layout that can grow is boarded at large text,
shown surviving it. Touch targets are drawn where the tappable area differs from the visible glyph,
at or above the platform minimum (44pt iOS, 48dp Android, 24px web minimum unless the system sets
higher). Contrast is legible in every theme shown. No state rides on one channel alone — not color
alone, not an icon alone, not motion alone, not sound alone. Words, shape, or position back it up,
and the record says what a screen reader announces.

**3. Every theme that ships.** Board each theme from the same semantic tokens. A light mode that
special-cases values inside components is a defect; theme values belong in the token layer.

**4. No emoji as UI, no real user data.** Use real SVG icons from the project's icon set. Content is
realistic but synthetic: no real names, emails, messages, or handles.

**5. The words are product copy.** Every string on the board is a real proposal. If the project has a
voice or copy guide (or a copy skill), use it. Records lock exact strings.

---

## Draw from source, not memory

Every token, font, nav item, icon, and component behavior on the board comes from a file read in this
session — `design/system.md` and the code it describes. Open the target component, follow its imports
to the theme it actually uses, and read the layout or route file around it.

Check these every time:

- **Tokens** — palette per theme, borders, radii, spacing, type scale, shadows, motion.
- **Fonts** per platform.
- **Navigation** — tabs or sidebar items, their order and names, which bars the OS owns, which routes
  hide chrome. Confirm a route exists before showing it.
- **Component semantics** — real field names, action labels, states.
- **Icons and logo** — copy geometry from source; never redraw a logo by hand.
- **Numbers that appear in UI** — limits, thresholds, prices. Read them from config or code.
- **OS chrome** — safe areas, notch, home indicator, keyboard, native bars. Leave flexible space
  rather than asserting exact pixels.

Never hard-code a palette, radius, or nav list from memory, including from an old board or record.
Those are snapshots, and snapshots rot.

---

## The board

**Every state, not the happy path.** One pretty default frame is a failed mockup. Ask: is there a
state the user would want an opinion on that is missing? Usually: default, loading, empty, error or
offline, success, destructive confirmation, permission missing, reduced motion — plus the gate frames
(large text, touch targets, each theme). Where the decision branches, put options side by side with
the trade-off visible and recommend one.

**Board the comparison state too.** Asked for the empty state, also board the populated one. Asked
for the error, also board the success. Asked for collapsed, also board expanded. The user is judging a
difference; show both sides.

**Truthful chrome.** Show the navigation and OS material that exist on that screen today. Never paint
custom chrome over pixels the OS owns.

**Realistic content at realistic density.** Believable titles, real-looking timestamps, long awkward
strings, crowded lists. Never lorem ipsum, never "Item 1", never a suspiciously tidy three-row list.
Placeholder content is the most common reason a board bounces.

**Notes go outside the device frame.** Nothing inside the simulated screen is annotation.

**Every frame explains itself alone.** A single frame lifted out of the board still makes sense.

**Open it before presenting it.** Render the file (browser or screenshot tool) and check for missing
assets, clipping, overflow, weak contrast, wrong fonts, and stale chrome. Compare against the live
component and relevant approved records. Do not report that it renders without looking.

---

## The ask, and the record

Ask for approval explicitly. Name the decisions the board asks the user to make, and anything you
left open on purpose.

Only after an explicit yes, write `design/approved/<slug>.md` from
`references/approval-record-template.md`: date, board path, the winning option, **the rejected
options and why**, locked visual and interaction behavior, exact strings, accessibility behavior,
platform differences, any record this one supersedes, and the source files that will own the
implementation. Keep open items in the record.

A record that lists only the winner has thrown away the useful half. The rejection reasons are what
stop the same idea coming back in three months.

If the approval changes the system (a new token, a new component, a changed rule), update
`design/system.md` at the same time and link the record.

---

## Reading `approved/`

Records are binding decisions, not inspiration.

- **List the directory; do not trust an index.**
- **Check for supersedes** before treating a record as live. A later record can retire an earlier one
  in whole or in part.
- **Precedence when sources conflict:** current code and tokens > `design/system.md` > approved
  records > this skill's defaults > general taste. Say so out loud when they conflict.
- **Never edit a record to erase what it locked.** Write a new record that supersedes it by name.

---

## Quality bar, ranked

Orders trade-offs between board qualities only. It never outranks the gates.

1. **Drawn from source, not memory.**
2. **Every state, the whole board.**
3. **Truthful chrome.**
4. **Consistent with the design system** — reuse the existing nav control, card, button, list row,
   loading pattern, and shadow scale. No local one-off variants.
5. **Restraint.** One signature moment per surface; the rest stays quiet. Decoration that carries no
   information is a defect. Restraint never removes an accessible affordance.
6. **One-glance clarity.** Hierarchy survives a two-second look.
7. **An honest record**, rejections and reasons included.

**Not real trade-offs.** Motion vs. static: if motion carries meaning, show it *and* the reduced-motion
reading. Speed vs. approval: skipping approval is a failed result, not a cheaper one. Coverage vs.
simplicity: coverage wins. Novelty vs. consistency: consistency wins for reused parts; novelty is
spent once, on purpose. New idea vs. existing approval: the approval holds, and the new idea is
offered as a supersede proposal. User preference vs. the system: the user wins, and the system gets
updated rather than special-cased.

---

## Never

- Never let shipped UI drift from the approved board. "Close enough" is a defect.
- Never claim something is built or shipped when only the mockup is approved.
- Never paint custom chrome over OS-owned bars.
- Never redraw a logo or mascot freehand.
- Never drop a rejected option or its reason from a record.

---

## Judgment calls

- **"Just a small tweak."** If someone looking at the result would have an opinion, mock it. If it is a
  mechanical token swap already covered by an approval, don't.
- **"Just build it," on an unapproved visual change.** The gate holds. Offer the fastest board that
  still shows the real states.
- **A new direction on a surface that already has a record.** Board it as a supersede proposal, showing
  what the old record locked and why it is being reopened.
- **Several platforms share the concept.** Cover each, and show where they differ on purpose.
- **Mostly motion.** Still a mockup. Demonstrate the motion and board the reduced-motion version.
- **Internal tools and admin screens.** Same quality bar. Ask whether the approval gate applies.

---

## Lessons worth keeping

General lessons from real mockup rounds. They apply to most products. Add your own below as you
learn them — each with the reason, because the reason is what makes it reusable.

- **"Make it more realistic"** almost always means placeholder content or missing app chrome.
- **Colored left accent bars on cards and quotes tend to read as AI-generated.** Find another way to
  mark emphasis.
- **Controls do not move between states.** A state may remove a control, but nothing slides into the
  empty slot.
- **One object owns the bottom of the screen at a time.** Do not stack a second bar on top of a tab bar
  or input.
- **Known chrome never shimmers.** Headers, inputs, and labels render immediately. Only data-dependent
  values get a skeleton, inside the real component shape, so the swap is a clean crossfade. Reduced
  motion makes the skeleton static.
- **Loading behavior is design, not plumbing.** Show last-known data instantly and refresh quietly;
  staggered pop-in on every open is a visible defect.
- **One control vocabulary everywhere.** One back/close control, one card, one primary button, one
  section label style, one shadow scale.
- **Do not animate opacity over blur or glass material.** Show or hide the material outright; move or
  resize instead of fading.
- **Untinted glass over a light background reads as nearly clear.** Text underneath collides with the
  header. Tint it from the theme.
- **Inverted lists invert scroll direction.** Any scroll-driven chrome on a chat-style list needs a
  real-device check before it is called buildable.
- **Do not put a marketing-weight card inside a settings list.** Match the density of the screen it
  lives in.
- **Honest empty and failure states.** Never invent a value; never show a stale zero for "unknown".
  Hiding a section on a quiet day often beats an empty box.
- **Icons need a specific idea.** Generic icons (a bolt, a sparkle) get vetoed for being generic.
- **Geometry around art comes from the art's real extent,** not its bounding box and not a magic
  offset. Magic offsets produce overlaps that only appear on some poses or sizes.

### Project-specific lessons

Add verdicts from this project's rounds here, or in `design/system.md` if they are really system
rules.
