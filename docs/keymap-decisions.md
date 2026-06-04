# Keymap Design Decisions

A running log of *why* the `config/corne.keymap` is shaped the way it is. Update
this whenever a deliberate layout decision is made, so future-you (and the layout
editor) has the reasoning, not just the result.

> Status legend: ✅ decided · 🔬 proposed (not yet in keymap) · ❓ open question

---

## Motivation (2026-06)

Two dissatisfactions with the current keymap drove this round of changes:

1. **Symbol access is awkward.** Symbols are currently trapped behind *mod-taps on
   the number layer* (e.g. `&mt AMPS NUMBER_7`, `&mt ASTERISK NUMBER_8`). That means
   getting a symbol is a *hold-on-hold*: hold a thumb to reach the num layer, then
   hold a number key to emit the symbol. Timing-sensitive and slow.
2. **No dedicated Shift.** Shift was scattered as hold-mods on `A`/`Z`/`/`, inspired
   by — but not faithful to — home-row-mod conventions.

The reference for the redesign is the **Anymak** layout
(<https://github.com/rpnfan/Anymak>), specifically its *principles* rather than its
literal key placement (Anymak targets a 4-row board with a fat spacebar and an
intentionally-empty B position; we are on a 3-row 5-column Corne with 6 thumb keys).

### 🐛 Bug found during review

The `sym_layer` (5th layer defined, **layer index 4**) is **orphaned** — nothing in
the base layer activates it. There is no `&lt 4` / `&mo 4` / `&sl 4` anywhere. This
is why the "real" symbol layer was unreachable. Fixing this is the first task.

### Clarification: the number-layer mod-taps were intentional

The `&mt EXCL NUMBER_1`, `&mt AT NUMBER_2`, … mod-taps on `num_layer` were **not a
mistake** — they deliberately mirror the **MacBook's shift+number symbols** in their
number-row positions (shift+1=`!`, shift+2=`@`, …). The real pain was the
*hold-on-hold* ergonomics. The redesign keeps the Mac positional mapping but makes it
**flat**: hold NUM, then Shift+number gives the symbol — exactly like macOS (see D7).

---

## Decisions

### ✅ D1 — Dedicated Shift = true one-shot (`&sk LSHFT`)
Anymak-style. **Pure modifier, no layer involved:**
- **Tap** → arms Shift for exactly the *next* keypress, then auto-disarms. Tap, then
  `a` → `A`. One capital, no holding.
- **Hold** → behaves as a normal held Shift while held (multi-key Shift, ⌘⇧ chords).
- Explicitly **not** a "tap=layer / hold=Shift" key. That combined behavior was
  considered and rejected as more error-prone and contrary to the Anymak design,
  which keeps Shift a pure one-shot. (A layer would instead use `&sl`/`&lt`.)
- Pairs cleanly with the existing `caps_word` combo for ALL-CAPS runs.
- Removes the scattered home-row/bottom-row Shift hold-mods (`Z`, `/`).
- **Lives on the left-inner thumb, adjacent to the NUM thumb (left-mid).** To type a
  symbol you fat-finger both left thumbs (NUM + Shift) and tap the number with the
  right hand → Shift+number = the symbol. Held together, it Shifts a whole run of
  symbols. The user confirmed fat-fingering the two adjacent thumb keys is fine.
- For `⌘+Shift+letter`: hold the Cmd thumb (right), tap one-shot Shift, then letter.

### ✅ D2 — No dedicated symbol layer; symbols come from Shift+key (Mac-native)
The orphaned `sym_layer` is **deleted**. With the NUM layer holding *plain* keycodes,
Shift reproduces every macOS symbol in its native position:

| NUM key | + Shift | NUM key | + Shift |
|---|---|---|---|
| `1`–`0` | `! @ # $ % ^ & * ( )` | `-` | `_` |
| `=` | `+` | `[` `]` | `{` `}` |
| `` ` `` | `~` | `\` | `\|` |

The base layer covers the rest: `;`→`:`, `,`→`<`, `.`→`>`, `/`→`?`, `'`→`"`. So
**100% of symbols are reachable exactly as on the MacBook**, with no symbol layer.

### ✅ D3 — NUM layer hosts numbers (right hand) + F-keys (left hand)
Numbers on the right hand in number-row order (`1-5` / `6-0`); the freed left hand
holds **F1–F12** (F-row sits above the number row on a Mac); extra symbol keys
(`- = [ ] \ ``) on the bottom so their shifted forms work (D2).

### ✅ D4 — Anymak principles adopted (the ones that fit a 3-row Corne)
- One-shot Shift (D1).
- Symbols via Shift, in their Mac-native positions (D2) — no held symbol layer.
- **Opposite-hand rule:** held layers (NUM, NAV) sit on *left* thumbs, so numbers and
  arrows are typed with the *right* hand.
- Consolidated Navigation layer (arrows + editing + media) on one thumb hold.
- **Space stays a pure key** (not a layer-tap) — it is the most-pressed key, so we
  avoid accidental layer activation there.

### ✅ D5 — Modifier placement (Cmd / Ctrl / Option) — RESOLVED
**Goal:** minimize divergence from the MacBook so switching between the Corne and the
laptop is seamless.

- **Ctrl → home-row mod on `A`** (D5a). On the MacBook, Caps Lock is remapped to
  Ctrl and sits at the left of `A`; the closest Corne equivalent is a home-row-mod
  Ctrl on the `A` key itself. Keep the existing `&hml LEFT_CONTROL A`.
- **Cmd → right INNER thumb**, as `&mt LGUI RET` (tap = Enter, hold = ⌘). Cmd is a
  hold-only key, so it sits on the least-comfy inner thumb by design (see the comfort
  principle in D8); `⌘C/⌘V/⌘D` stay opposite-handed (right thumb holds, left hand
  types). Tap = Enter. *(Start with `balanced` flavor so the Enter tap doesn't
  misfire as ⌘.)*
- **Option/Alt → dropped.** User confirmed they don't use it.

### ✅ D8 — Thumb comfort ranking drives placement; Backspace is pure
**User's thumb comfort ranking:** outer = most comfy, mid = homing (resting)
position, inner = least comfy.

**Principle that follows:** *holds* (layer/modifier keys you just press and keep down)
go on the **least-comfy inner** thumbs — comfort barely matters when you only hold.
*Frequent taps and pure keys* go on the **comfy outer / homing mid** thumbs.

- **Backspace** = pure `&kp BSPC` (a layer-tap would break hold-to-repeat delete). It
  earns the **right-outer (most comfy)** thumb because it is frequent and held to
  repeat.
- **NUM** and **Cmd** are holds → the two **inner** thumbs (left = NUM, right = Cmd).
- **`'`** rides as the tap of the NAV thumb (left-outer, most comfy) — good for
  Python strings; **`"`** = Shift+'. **Tab** rides as the tap of the NUM thumb
  (left-inner) since Tab is infrequent.
- **Shift** (one-shot) sits on the **left homing (mid)** thumb — most-frequent left
  tap, adjacent to NUM for the fat-finger symbol trick. *Caveat:* if resting causes
  stray capitals, add `require-prior-idle-ms` to the `&sk`.

### ✅ D7 — Layer set collapses to three (base / num / nav)
With symbols handled by Shift (D2) and media folded into NAV's free left hand, the
layout drops from 7 layers to **3**:

| Idx | Layer | Held by | Typed with |
|---|---|---|---|
| 0 | base | — | both |
| 1 | num  | left-mid thumb (NUM/BSPC) | right hand (numbers); left hand = F-keys |
| 2 | nav  | left-outer thumb (NAV/TAB) | right hand (arrows); left hand = media/BT |

Both held layers are on **left** thumbs → numbers and arrows on the **right** hand
(arrows in the conventional right-hand position). The SYM and MEDIA layers are gone.

### 🔬 D6 — Free up alpha keys used as layer/mod hosts
Today `T` is `&lt 3` (media) and `B` is `&lt 2` (nav), and Shift/Ctrl ride on
`A`/`Z`/`/`. Moving layer access to thumbs and Shift to a one-shot returns these
alphas to plain letters — better for the alpha layout you're happy with.

---

## Proposed layer maps (🔬 not yet written to `corne.keymap`)

Legend: `XXX/YYY` on a thumb = `tap XXX` / `hold YYY`. Alphas unchanged (QWERTY).
`td_q` = tap Q / double-tap or hold Esc (existing tap-dance, kept).

### Layer 0 — BASE
```
 Q*   W    E    R    T        Y    U    I    O    P
 A    S    D    F    G        H    J    K    L    ;
 Z    X    C    V    B        N    M    ,    .    /
        ┌───────┬──────────┬─────────┐   ┌─────────┬───────┬──────┐
        │ NAV/' │ ⇧ oneshot│ NUM/TAB │   │ RET/CMD │ SPACE │ BSPC │
        └───────┴──────────┴─────────┘   └─────────┴───────┴──────┘
        outer    mid(home)   inner       inner    mid(home) outer
   (* Q = td_q: tap Q / hold Esc)
```
Thumb rationale (comfort: outer > mid/homing > inner — see D8):
- **NAV/'** (left outer, most comfy) — hold = Nav (arrows on the RIGHT hand); tap =
  `'` (frequent Python char gets the best thumb). `"` = Shift+'.
- **⇧ one-shot** (left mid, homing) — most-frequent left tap; adjacent to NUM so
  NUM+⇧ fat-finger → symbols (D1/D2).
- **NUM/TAB** (left inner, least comfy) — hold = Num (numbers on the RIGHT hand); a
  hold belongs on the worst thumb; tap = Tab (infrequent).
- **RET/CMD** (right inner, least comfy) — hold = ⌘ (a hold, so worst thumb is fine;
  `⌘C/⌘V/⌘D` opposite-handed); tap = Enter (D5).
- **SPACE** (right mid, homing) — pure key, rests under the thumb (D4).
- **BSPC** (right outer, most comfy) — pure key, frequent + holds to repeat (D8).
- Home row keeps `&hml LEFT_CONTROL A` for Ctrl (mirrors MacBook Caps→Ctrl, D5a).

### Layer 1 — NUM  (hold left NUM/TAB; numpad on the RIGHT hand)
```
 F1   F2   F3   F4   F5       [    7    8    9    -
 F6   F7   F8   F9   F10      ]    4    5    6    =
 F11  F12  \    ─    ─        \    1    2    3    ENTER
                              (right thumbs: SPACE→. (DOT)   BSPC→0)
```
**Updated post-build via the keymap editor** (commit "move delete to num layer"):
the right hand is now a **numpad** (7-8-9 / 4-5-6 / 1-2-3, `0` and `.` on the right
thumbs, numpad `Enter`), with brackets `[ ]` and `- =` down the inner column and
`\` on the left. Numbers are the regular `N1`–`N0` keycodes, so **Shift still yields
the macOS symbols** (`Shift+7`=`&`, `Shift+[`=`{`, …, D2) — only the *spatial*
arrangement changed from a number row to a numpad. F-keys remain on the left hand.

### Layer 2 — NAV  (hold left TAB/NAV; arrows on the RIGHT hand)
```
 BT0  BT1  BT2  BT3  BTCLR    HOME PGDN PGUP END  DEL
 VOL- VOL+ PREV NEXT PLAY     LEFT DOWN UP   RGHT INS
 ─    ─    ─    ─    ─        ⌘Z   ⌘C   ⌘X   ⌘V   ⌘⇧Z
```
Arrows on the right home row, paging on top, clipboard on the bottom. Media +
Bluetooth fill the free left hand, so no separate media layer (D4/D7).

---

## Open items before writing `corne.keymap`
- [x] **D1**: one-shot Shift = pure modifier, left-inner thumb (fat-finger NUM+⇧ for symbols).
- [x] **D2**: no symbol layer — symbols via Shift+key (Mac-native).
- [x] **D5/D5a**: Ctrl = HRM on `A`; Cmd = `&mt LGUI RET` (right INNER); Option dropped.
- [x] **D7**: collapse to 3 layers (base/num/nav); arrows + numbers on the right hand.
- [x] **D8**: comfort outer>mid>inner; holds on inner; Backspace pure on right-outer;
      `'` = tap of left-outer NAV thumb; Tab = tap of left-inner NUM thumb.
- [x] `\` lives on the NUM-layer left-bottom row (`Shift+\` → `|`).
- [x] Keep `td_q` (tap Q / Esc) for now.
- [x] Drop the `copy`/`paste` combos — ⌘ is on the right thumb, so `⌘C/⌘V` are native.
- [ ] Confirm `&mt LGUI RET` flavor/timing so the Enter tap doesn't misfire as ⌘.
- [x] Update `AGENTS.md` layer/combo tables to match the implemented keymap.
- [x] NUM layer re-tuned to a numpad via the keymap editor (see Layer 1 above).
```
