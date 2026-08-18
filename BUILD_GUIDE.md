# Build guide — Operations Control Plane

How to build this control panel from an empty file to a live URL. Every step maps to
code that is actually in `index.html`, so you can follow it start to finish or jump to
the section you are changing.

**What you are building:** a single-page, seven-tab operations console — a market
dependency graph, six telemetry panels, a four-lane architecture diagram, a named
automation layer, a thirteen-stage change-management pipeline, a control-anatomy view
that computes which controls are unprovable, and an interactive change simulator that
computes the blast radius of a regulatory change.

**What you need:** a text editor, a browser, and git. That is all. No Node, no npm, no
build step, no framework, no dependencies. The entire panel is one HTML file with
inline `<style>` and `<script>`.

**Why one file:** the whole thing has to be readable by someone who did not build it,
deployable by dropping it on a static host, and openable by double-clicking it offline.
Any of those goes away the moment you add a build step. A ~700-line file is still under
the size where splitting starts paying for itself — and the data that drives the newer
views lives in plain arrays at the bottom of the script, which is the part you will edit
most.

---

## Step 0 — Decide the argument before you write markup

This panel is a design exercise, not a dashboard. The layout exists to carry seven
claims, one per tab:

| Tab | Claim it has to make |
|---|---|
| Control plane | Shared controls are the propagation risk, and the negatives (silent controls, missing evidence) are the finding |
| Architecture | Judgement visibly leaves the automation and reaches a person with authority |
| Automation layer | Where the workflow tool actually sits — and, more convincingly, where it stops |
| Change management | The stages people skip are the stages where change management fails |
| Control anatomy | A row is not a control; a control that cannot be proved is indistinguishable from absent |
| Change simulator | A register turns "what does this affect?" from an investigation into a query |
| Approach | The reasoning, including what the author would need to learn |

Write these down first. Every visual decision later — node size, red edges, a warning
row that appears after the table — is answerable by "which claim does this serve?" If a
panel serves none of them, cut it.

---

## Step 1 — Skeleton

Create `index.html`. The name matters: static hosts serve `index.html` at the directory
root automatically, and anything else 404s.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Compliance Operations Control Plane — Design Exercise</title>
<style>/* Step 2 */</style>
</head>
<body>
<div class="wrap">
  <header></header>   <!-- Step 3 -->
  <div class="strip"></div>
  <nav role="tablist"></nav>  <!-- Step 4 -->
  <section class="view on" id="plane"></section>
  <section class="view" id="arch"></section>
  <section class="view" id="auto"></section>   <!-- Step 8 -->
  <section class="view" id="chg"></section>    <!-- Step 9 -->
  <section class="view" id="anat"></section>   <!-- Step 10 -->
  <section class="view" id="sim"></section>
  <section class="view" id="approach"></section>
  <footer></footer>
</div>
<script>/* Steps 8-11 */</script>
</body>
</html>
```

The `viewport` meta is not optional — without it the SVG diagrams render at desktop
width on a phone and the whole thing is unreadable.

---

## Step 2 — Design tokens

Define the palette once, in `:root`, and never write a raw hex value in a rule again.
This is the single highest-leverage step: it is what makes six panels built at
different times look like one system.

```css
:root{
  --void:#0b0e14;   /* page ground        */
  --panel:#12161f;  /* raised surface     */
  --stroke:#232b3a; /* hairline border    */
  --lit:#2d3f56;    /* active border      */
  --hi:#c9d4e4;     /* primary text       */
  --txt:#8b97ab;    /* body text          */
  --dim:#5f6b80;    /* labels, captions   */
  --ok:#5dcaa5;     /* healthy / scoped   */
  --warn:#efaa3c;   /* attention / active */
  --crit:#e8615f;   /* risk / global      */
  --mono:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
  --sans:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
}
```

Three rules that keep it coherent:

1. **Colour carries meaning, not decoration.** Green is market-scoped and healthy, amber
   is the active scenario or partial state, red is *global scope* and risk. Once that
   mapping is set, a red line in the graph and a red `HIGH` in the simulator table mean
   the same thing without a legend telling you.
2. **Four greys, in order.** `--void` → `--panel` → `--stroke` → `--lit` is a depth
   ladder. Panels are one step off the ground; borders are one step off the panel.
3. **Monospace only for data.** Log lines, counters, telemetry readouts use `--mono`.
   Prose uses `--sans`. The eye then sorts machine output from human writing before
   reading a word.

Base type is small — `font-size:13px` on `body`, 9–11px inside panels. That is
deliberate: a control plane should read as instrumentation, dense and scannable. Keep
`line-height:1.6` so density never becomes cramped.

Add the system fonts stack and `-webkit-font-smoothing:antialiased`, then the primitives:

```css
*{box-sizing:border-box}
body{margin:0;background:var(--void);color:var(--txt);
     font-family:var(--sans);font-size:13px;line-height:1.6}
.wrap{max-width:1180px;margin:0 auto;padding:18px 16px 40px}
svg{display:block;width:100%;height:auto}
```

That `svg` rule is what makes every diagram responsive later — set a `viewBox`, never a
fixed `width`, and the browser scales it.

---

## Step 3 — Header and the status strip

The header is three lines: what this is, who made it, and the provenance disclaimer.
Put the disclaimer in the header, not just the footer — a reader who screenshots the top
of the page should still see "built from public sources only."

The strip below it is the at-a-glance state of the system:

```html
<div class="strip">
<span>MARKETS <b>11</b> verified</span>
<span>REQUIREMENTS <b>17</b></span>
<span>CONTROLS <b>16</b></span>
<span>UNPROVABLE <b class="cr" id="upv">5</b></span>
<span>GAPS <b class="cr">1</b></span>
<span>PARTIAL <b class="wn">3</b></span>
<span>STALE <b class="wn">4</b></span>
<span>SCENARIOS <b>14</b></span>
<span>SCENARIO <b class="ok">KE-2025</b></span>
</div>
```

```css
.strip{display:flex;gap:16px;flex-wrap:wrap;font-size:10px;letter-spacing:.07em;
       color:var(--dim);border-top:1px solid var(--stroke);
       border-bottom:1px solid var(--stroke);padding:9px 0;margin:14px 0 0}
.strip b{color:var(--hi);font-weight:500}
```

`flex-wrap` does the responsive work for free. The label stays dim, the number goes
bright — so the row scans as numbers first, labels second.

**Include the bad numbers.** `GAPS 1`, `PARTIAL 3`, `STALE 4`, `UNPROVABLE 5` are the
credible part. A strip showing all green reads as a mock-up; a strip admitting that five
of sixteen controls could not be evidenced in an audit reads as a system someone actually
runs. Note the `id="upv"` — that one is computed in Step 10, never typed.

---

## Step 4 — Tabs

Seven buttons, seven sections, one class toggle. No router, no framework.

```html
<nav role="tablist">
<button role="tab" aria-selected="true"  data-v="plane">CONTROL PLANE</button>
<button role="tab" aria-selected="false" data-v="arch">ARCHITECTURE</button>
<button role="tab" aria-selected="false" data-v="auto">AUTOMATION LAYER</button>
<button role="tab" aria-selected="false" data-v="chg">CHANGE MANAGEMENT</button>
<button role="tab" aria-selected="false" data-v="anat">CONTROL ANATOMY</button>
<button role="tab" aria-selected="false" data-v="sim">CHANGE SIMULATOR</button>
<button role="tab" aria-selected="false" data-v="approach">APPROACH</button>
</nav>
```

```css
.view{display:none}
.view.on{display:block}
nav button[aria-selected=true]{color:var(--hi);border-color:var(--lit);background:var(--panel)}
```

```js
Array.prototype.forEach.call(document.querySelectorAll("nav button"), function(b){
  b.addEventListener("click", function(){
    Array.prototype.forEach.call(document.querySelectorAll("nav button"), function(x){
      x.setAttribute("aria-selected","false"); });
    Array.prototype.forEach.call(document.querySelectorAll(".view"), function(v){
      v.classList.remove("on"); });
    b.setAttribute("aria-selected","true");
    document.getElementById(b.dataset.v).classList.add("on");
  });
});
```

Two things worth keeping: `data-v` ties the button to its section id, so adding a fifth
tab is one button plus one section and no JS change. And `aria-selected` drives *both*
the accessibility tree and the CSS — the styling cannot drift out of sync with what a
screen reader announces, because there is only one source of truth.

---

## Step 5 — The market graph

This is the centrepiece, and it is hand-authored SVG. No graph library. Eleven markets
with fixed positions do not need a force simulation, and a library would add 40kB to
draw eleven circles.

```html
<svg viewBox="0 0 660 210" role="img"
     aria-label="Market graph showing shared control dependencies between markets">
```

The `viewBox` gives you a 660×210 coordinate space that scales to any width. The
`role`/`aria-label` pair is what a screen reader gets instead of the picture — write it
as a sentence, not a filename.

Build it in four layers, in this order (SVG paints in document order, so later elements
sit on top):

**Layer 1 — base edges.** Every shared control, thin and low-opacity:

```html
<g stroke="#2d3f56" stroke-width="0.6" fill="none" opacity=".5">
<path d="M330 105 L175 58"/><path d="M330 105 L145 125"/>
...
</g>
```

**Layer 2 — global control edges**, red and animated. These are the argument of the
whole view:

```html
<g class="dash" fill="none" stroke="#e8615f" stroke-width="1.2" opacity=".85">
<path d="M330 105 L472 52"/><path d="M330 105 L515 112"/>
</g>
```

**Layer 3 — market-scoped edges**, green, same animation at a slower duration so the two
kinds of traffic are distinguishable in motion, not just colour.

**Layer 4 — nodes and labels.** Kenya is the active scenario: larger radius, amber, plus
a faint halo ring to mark focus.

```html
<circle cx="330" cy="105" r="17" fill="none" stroke="#efaa3c" stroke-width="0.6" opacity=".35"/>
<circle class="nd" cx="330" cy="105" r="9.5" fill="#efaa3c" style="animation-duration:2.1s"/>
<text x="330" y="134" text-anchor="middle" font-size="10" fill="#efaa3c">kenya</text>
```

The animations are three CSS keyframes, reused everywhere:

```css
@keyframes puls{0%,100%{opacity:.4}50%{opacity:1}}   /* nodes breathing   */
@keyframes trav{to{stroke-dashoffset:-40}}           /* traffic along edges */
@keyframes bl  {0%,100%{opacity:.45}50%{opacity:.95}} /* heat bars         */
.nd{animation:puls 3s ease-in-out infinite}
.dash{stroke-dasharray:3 6;animation:trav 2.4s linear infinite}
.hb{animation:bl 1.9s ease-in-out infinite}
```

**Vary the durations per element** with an inline `style="animation-duration:2.7s"`. If
every node pulses on the same clock the page throbs and looks broken; at 2.1s / 2.7s /
3.2s / 3.5s the nodes drift out of phase and it reads as independent systems running.

Encode information in geometry, not just colour: node radius is requirement count
(Kenya 9.5, Ghana and Nigeria 6.5, the rest 5.5). Then state it in the legend —
`node size = requirement count` — because an encoding nobody is told about is decoration.

Finally, `+4 verified` in the corner. Drawing 11 nodes in 660×210 would be mush; drawing
7 and admitting the remainder is honest and legible.

---

## Step 6 — The six telemetry panels

One responsive grid, one panel component, repeated six times.

```css
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(158px,1fr));
      gap:8px;margin-top:12px}
.pnl{background:var(--panel);border:1px solid var(--stroke);border-radius:5px;
     padding:9px 11px;min-width:0}
.lb{font-size:9px;letter-spacing:.09em;color:var(--dim);margin:0 0 7px;text-transform:uppercase}
.rw{font-family:var(--mono);font-size:9.5px;line-height:1.6;color:var(--txt);word-break:break-word}
.ok{color:var(--ok)}.wn{color:var(--warn)}.cr{color:var(--crit)}.h{color:var(--hi)}
```

`repeat(auto-fit,minmax(158px,1fr))` is the entire responsive strategy: six across on a
desktop, two on a phone, no media query. `min-width:0` on the panel stops long monospace
strings from forcing the grid track wider than its share.

The six panels, and why each exists:

1. **Change log** — timestamped lines proving the system detected, assessed, warned,
   classified, sequenced. It shows the machine doing process work.
2. **Blast radius** — the simulator's answer, pinned where it is always visible, with a
   `.bar` progress element for proportion.
3. **Exception heat** — six flex divs with percentage heights. A bar chart with no
   library, no axis, no labels; you only need the shape and the outlier.
4. **Control heartbeat** — five controls and their liveness. `CTL-ADV-03 silent 41d` is
   the point of the whole panel.
5. **Escalation** — raised, routed to human, auto-resolved, and the rate, with the
   caption `falling rate = warning`. That caption inverts the usual dashboard reflex and
   is the thesis in five words.
6. **Register health** — the meta-panel: does the register itself hold up? No owner, no
   evidence, unmapped controls, and how much of the data is verified versus illustrative.

Close the view with the interpretive note — a `.note` block with a left border:

```css
.note{font-size:10px;color:var(--dim);margin:14px 0 0;padding-left:9px;
      border-left:1px solid var(--stroke)}
```

> **The negatives are the finding.** A control with no evidence that it operated is
> indistinguishable from a control that has stopped operating.

Do not make the reader infer the conclusion from the panels. State it under them.

---

## Step 7 — The architecture diagram

A 660×400 SVG in four named lanes, left to right:

```
INGEST  ──▶  TOOLING          ──▶  SYSTEM OF RECORD  ──▶  HUMAN LAYER
             Make.com scenarios     requirements register
             SCN-01 … SCN-14        evidence store
                                    immutable log
```

**Do not draw a box labelled "rule engine."** A reader cannot see where the workflow tool
sits, what it does, or where it stops — which are the only three things the diagram is
for. Name the lane, list the scenarios inside it, and label every path with the SCN
number that runs it. That is the difference between a diagram of a system and a diagram
of an idea: a viewer should be able to point at any line and read `SCN-03 document
follow-up`.

**7a. Lane labels** — small, dim, letter-spaced, so the structure is readable before any
box is.

**7b. Boxes.** Group the ones that share styling so the attributes are written once:

```html
<g fill="#12161f" stroke="#232b3a" stroke-width="0.6">
<rect x="14" y="44" width="88" height="26" rx="4"/>
...
</g>
```

**7c. An arrowhead marker**, defined once in `<defs>` and referenced by every connector:

```html
<marker id="ar" viewBox="0 0 10 10" refX="8" refY="5"
        markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.4" stroke-linecap="round"/>
</marker>
```

`stroke="context-stroke"` makes each arrowhead inherit the colour of the line it
terminates — one marker serves the grey connectors, the green write-back path, and the
red routing bus.

**7d. The divider.** A dashed red vertical line at x=472, running the full height, with a
rotated label:

```html
<path d="M472 18 L472 372" stroke="#e8615f" stroke-width="1" stroke-dasharray="4 4"/>
<text x="464" y="200" font-size="8" fill="#e8615f" text-anchor="middle"
      transform="rotate(-90 464 200)">AUTOMATION STOPS HERE</text>
```

Everything right of that line is a person. One red routing bus crosses it, carrying the
label `SCN-02·03·04·06·07·08·09·10 / routing only — no disposal`. Nothing automated
crosses back.

**7e. The escalation ladder** lives in the human lane: five tiers, each `rect` indented
7px further and 7px narrower than the one above, so authority visibly descends. Tier 3 —
the designated officer — gets `stroke="#e8615f"`.

The one path that does return leftward is the human decision: an orthogonal green route
from `decision + rationale` back into the immutable log, labelled `human decision written
back · SCN-05 records who, what, when and why`. Label it explicitly as human-originated,
or it reads as automation reaching back across its own boundary.

**7f. Travelling particles.** Define invisible paths in `<defs>`, then animate circles
along them:

```html
<defs><path id="q1" d="M102 57 L136 72" fill="none"/></defs>
...
<circle r="2.5" fill="#5dcaa5">
  <animateMotion dur="2.5s" repeatCount="indefinite"><mpath href="#q1"/></animateMotion>
</circle>
```

Stagger them with `begin="0.8s"`, `begin="1.4s"` and so on. Four particles is enough to
read as flow; more looks like a screensaver. The one on the routing bus is red and larger
(`r="3.5"`) — the eye follows it, which is exactly where you want the eye.

**The trap in that snippet:** a `<circle>` driven by `animateMotion` has no `cx`/`cy`, so
between page load and its `begin` time it paints at the SVG origin — a stray coloured
quarter-disc in the top-left corner, on every load, for as long as the delay lasts. It is
easy to miss because it disappears by the time you look. Hide it until its animation
starts:

```html
<circle r="3.5" fill="#e8615f" visibility="hidden">
  <set attributeName="visibility" to="visible" begin="1.1s"/>
  <animateMotion dur="3.4s" begin="1.1s" repeatCount="indefinite"><mpath href="#q4"/></animateMotion>
</circle>
```

Setting `cx`/`cy` instead does not work — `animateMotion` translates *relative* to them,
so the whole path shifts.

---

## Step 8 — The automation layer

The view that answers "where does the workflow tool actually fit." One row per scenario,
five columns: scenario, trigger, what it does, what it writes to, and **where it stops**.

Hold the scenarios in an array, not in markup — you will add to them, and a table you
hand-write in HTML is a table you stop maintaining:

```js
var SCN = [
["SCN-01","case creation","New case event","Create record, assign owner…","Register: new row",
 "No human — no judgement present", 0,
 "Webhook case.created → Router by market → Airtable create record → …",  // modules
 "A new market missing from the routing table: the case is created with no owner.", // failure mode
 "No cases are created. SCN-08 sees no evidence inside the interval and raises an alert."],
…
];
```

Index 6 is the flag that colours the stop point: `0` means the scenario completes with no
human in it (green), `1` means it stops at a person (red). Only SCN-01 and SCN-05 are
green, and that ratio is the argument.

**The vertical line.** The whole view exists to render one graphic: a hard divider between
"writes to" and "stops at", labelled `automation stops here`. Because the table is
`table-layout:fixed` with declared column widths (14 + 15 + 31 + 18 + 22), the boundary
sits at a known 78% — so the divider is an absolutely positioned element, not a guess:

```css
.tw{position:relative;min-width:730px}
.stopv{position:absolute;left:78%;top:0;bottom:0;border-left:1px dashed var(--crit)}
.stoplbl{position:absolute;left:78%;top:34px;margin-left:-13px;
         writing-mode:vertical-rl;font-size:8px;color:var(--crit)}
```

Wrap that in `.tscroll{overflow-x:auto}` so a wide table scrolls inside its own box
instead of pushing the whole page sideways on a phone.

**Expandable rows.** Each scenario row is followed by a hidden detail row carrying three
things: the modules in order, the failure mode, and what happens if the scenario itself
breaks. Toggle with a class, and bind `keydown` as well as `click` with `tabindex="0"` so
the rows are reachable without a mouse:

```js
function tog(){ dt.classList.toggle("open"); }
tr.addEventListener("click", tog);
tr.addEventListener("keydown", function(e){
  if(e.key === "Enter" || e.key === " "){ e.preventDefault(); tog(); }
});
```

The "if the scenario itself breaks" line is the one worth writing carefully. It is what
turns a diagram of an idea into a description of a system: SCN-05 failing puts a hole in
every control's evidence chain, and SCN-08 is the scenario that catches other scenarios
failing — which means it needs its own watcher.

**The never-automate panel.** Seven items, rendered next to the table. Naming the
exclusions is more convincing than listing the inclusions, and it is the only part of the
view that cannot be mistaken for a feature list.

---

## Step 9 — Change management, thirteen stages

Two components, and the split between them is the point: **the strip is what someone
remembers, the table is what they interrogate.**

The strip is ten flex items, no library:

```css
.steps{display:flex;flex-wrap:wrap;gap:4px}
.steps span{flex:1;min-width:70px;text-align:center;font-size:8.5px;
            text-transform:uppercase;padding:8px 3px;background:var(--panel);
            border:1px solid var(--stroke);border-radius:3px}
```

Understand → Map → Design → Assign → Build → Test → Train → Launch → Monitor → Evidence.

The table beneath carries all thirteen stages with owner, automation and evidence
produced. Four of them — 8 SOP update, 9 training, 11 effective-date tracking, 13
post-launch monitoring — carry a `skip` flag in the data:

```css
tr.skip td{background:#161311}
tr.skip td:first-child{border-left:2px solid var(--warn)}
```

plus an `OFTEN SKIPPED` chip rendered from the same flag. Do not shade them by hand in
markup; drive both the shading and the chip off one boolean, or they will drift apart.

Then say why each one matters in prose underneath. The shading raises the question; only
the prose answers it — that regulations have commencement dates, and being ready two
weeks late is a breach even if the work was excellent.

---

## Step 10 — Control anatomy

The most data-heavy view, and the one that produces the uncomfortable number.

**The ten questions** render as cards from a `QS` array of `[question, field,
why it matters]`. Give question 9 a red border in code, not by hand:

```js
if(i === 8){ d.style.borderColor = "#3a2226"; }
```

**Controls** are `[id, description, answers[10]]`, where an empty string means the
question is unanswered. That single convention drives everything downstream — the dot
matrix, the missing-question list, and the computed flag:

```js
function provable(a){ return !!(a[3] && a[7] && a[8]); }
```

Evidence location, logged events, assurance method. If any of the three is missing the
control cannot be evidenced retrospectively, which is what makes `provable` a computation
rather than an opinion. Count the failures and write the number into both the status strip
and the view:

```js
document.getElementById("upv").textContent = un;
document.getElementById("upv2").textContent = un;
```

**Never hard-code that count in two places.** It appears in the strip and in the question-9
callout, and a register that contradicts itself about how many controls are unprovable
undermines the whole argument. One computation, two write targets.

**Work one control in full.** CTL-KYC-01 answers all ten questions concretely — that is
the reference for what "finished" looks like. Every other control then renders as one row
with ten dots, green for answered and red for not, expanding to name exactly which
questions are missing and why that makes the control unprovable. The gaps are the finding,
exactly as with the register.

---

## Step 11 — The change simulator

The only genuinely interactive view. Three parts: a data model, a render function, and
a staged reveal.

**8a. Data model.** One object, keyed by scenario. Rows are positional arrays — surface,
change required, class, scope, risk:

```js
var SC = {
  ke: {
    d: "Came into force 26 August 2025, replacing the Betting, Lotteries and Gaming Act…",
    r: [
      ["Licence structure","Separate online licence types…","Structural","market","low"],
      ["Advertising controls","Messaging, age warnings…","Tightening","global","high"],
      …
    ],
    w: "The two high-risk rows are the finding. Advertising runs through a shared creative pipeline…"
  },
  fmt: { … }, dep: { … }
};
```

Keep `d` (description), `r` (rows) and `w` (the propagation warning) separate. The
warning is the analysis and must not be derivable from the table — if the reader could
compute it themselves, it is not adding anything.

Label scenarios honestly in the `<option>` text: `(real, sourced)` versus `(synthetic)`.

**8b. Escape everything you inject.** Any string that reaches `innerHTML` goes through:

```js
function esc(s){var d=document.createElement("div");d.textContent=s;return d.innerHTML}
```

The data is currently a hard-coded literal, so nothing hostile can reach it today. The
habit matters for when this is wired to a real register — at that point the strings come
from an API and the escaping is the only thing between you and injected markup.

**8c. Render, with a staged reveal.** Rows are appended dimmed (`tr.className="dim"`),
then lit one at a time on a 160ms cadence:

```js
s.r.forEach(function(row,i){
  var tr = document.createElement("tr");
  tr.className = "dim";
  var cls  = row[2]==="Structural" ? "c-t" : (row[2]==="New obligation" ? "c-o" : "c-m");
  var risk = row[4]==="high" ? '<span class="cr">HIGH</span>'
           : (row[4]==="med" ? '<span class="wn">med</span>' : '<span class="ok">low</span>');
  var scope= row[3]==="global" ? '<span class="cr">global</span>'
                               : '<span class="ok">market</span>';
  tr.innerHTML = "<td>"+esc(row[0])+"</td><td>"+esc(row[1])+"</td>"
               + "<td><span class='chip "+cls+"'>"+esc(row[2])+"</span></td>"
               + "<td>"+scope+"</td><td>"+risk+"</td>";
  tb.appendChild(tr);
  if(row[3]==="global") g++;
  setTimeout(function(){ tr.className = row[4]==="high" ? "lit" : ""; }, 160*(i+1));
});
```

The staging is not decoration — it makes the blast radius look *computed* rather than
retrieved, and high-risk rows stay highlighted (`.lit`) after the sweep finishes while
everything else settles to normal. That is the finding surviving the animation.

**8d. The counter,** fired after the last row lands:

```
1 change → 6 surfaces affected → 2 shared controls at propagation risk
         → 2 markets that could be silently changed
```

Written as a chain, in that order, because the last clause is the one that matters and
you want the reader to arrive at it having accepted each step. Then reveal the `warn`
note with the analysis.

**8e. Reset on change.** When the `<select>` changes, clear the table, description,
warning and counter back to `Press RUN to compute the blast radius.` Otherwise the
reader sees Kenya's table under a Nigeria heading and trusts nothing afterwards.

---

## Step 12 — The approach tab

Plain prose, `max-width:74ch`, section headings at 12px uppercase. No diagrams. Seven
sections: the framing, three obligation buckets, two principles, design versus operating
effectiveness, the exception/incident/breach distinction, what the author brings and
would need to learn, and tooling.

```css
h2{font-size:12px;font-weight:500;color:var(--hi);letter-spacing:.06em;
   text-transform:uppercase;margin:22px 0 8px}
p.body{font-size:12.5px;max-width:74ch;margin:0 0 10px}
```

The `74ch` cap is doing real work: on a 1180px page, full-width prose is unreadable. Let
the diagrams use the width; keep the text in a column.

Keep the "what I would need to learn" section. A design exercise that claims no gaps is
less credible than one that names them.

---

## Step 13 — Accessibility and motion

Four things, none optional:

```css
@media (prefers-reduced-motion:reduce){ .nd,.dash,.hb{animation:none} }
@media (max-width:560px){ .strip{gap:10px;font-size:9px} h1{font-size:14px} }
```

1. **Respect reduced motion.** Users who set it get the full page, statically. Note the
   SMIL `<animateMotion>` particles in Step 7 are *not* covered by that CSS rule — if you
   want them stopped too, gate them in JS on
   `window.matchMedia("(prefers-reduced-motion:reduce)").matches`.
2. **`role="img"` plus a descriptive `aria-label`** on both SVGs.
3. **`role="tablist"` / `role="tab"` / `aria-selected`** on the nav, kept in sync by the
   same code that drives the styling.
4. **Contrast.** `--dim:#5f6b80` on `--void:#0b0e14` is the lightest text on the page;
   check anything dimmer against WCAG AA before shipping it.

---

## Step 14 — Preview locally

```bash
python3 -m http.server 8000     # then open http://localhost:8000
```

Opening the file directly with `file://` works too, since there are no fetches. Serve it
over HTTP anyway — it is the same protocol GitHub Pages uses, so path bugs surface now
rather than after deploy.

Check, in order: all four tabs switch; the graph and architecture SVGs scale down to a
360px-wide viewport without clipping; RUN populates the table, lights the high-risk rows,
prints the counter and reveals the warning; changing the `<select>` clears everything;
the console is clean.

---

## Step 15 — Deploy to GitHub Pages

```bash
git add index.html README.md
git commit -m "Add operations control plane"
git push -u origin main
```

Then in the repository: **Settings → Pages → Source: Deploy from a branch →
`main` / `(root)` → Save.** The site appears at
`https://<user>.github.io/<repo>/` within a minute or two.

**The one thing that breaks this:** the file at the repository root must be named
exactly `index.html`. A file named `index (1).html` — the name a browser gives a second
download — will publish successfully and 404 at the site URL, because Pages has no
`index.html` to serve for the root path. If your live demo link is dead, check the
filename first.

---

## Step 16 — Extending it

**Add a market.** Pick coordinates in the 660×210 viewBox, add a `<circle>` sized to its
requirement count plus a `<text>` label, draw its edges in the layer matching their
scope (grey base, red global, green market-scoped), and update `MARKETS` in the strip and
the `+N verified` counter.

**Add a simulator scenario.** Add an `<option>` to `#pick` and a matching key in `SC`
with `d`, `r` and `w`. No other change — `run()` reads whatever is in the object. Say in
the option label whether it is real or synthetic.

**Add an automation scenario.** Append one row to `SCN` with all ten fields, including the
human-stop flag, the modules, the failure mode and what happens if the scenario breaks.
Then add its number to the tooling lane in the architecture SVG and to any path it runs,
and update `SCENARIOS` in the strip. A scenario that exists in the table but not in the
diagram is exactly the undocumented automation the Approach tab warns about.

**Add a control.** Append `[id, description, answers[10]]` to `CTL`. Leave a string empty
where the question is genuinely unanswered — the dot matrix, the missing-question list and
the `provable` count all derive from that, so faking an answer to make the number look
better corrupts the one metric the view exists to produce.

**Add a telemetry panel.** Copy one `.pnl` block. The grid absorbs it with no CSS change.

**Wire it to real data.** The structure already assumes a register: markets, requirements,
controls, evidence, exceptions. Put that in Airtable, expose a view, replace the `SC`
literal with a `fetch`, and keep `esc()` on every field — that is the moment escaping
stops being a habit and starts being load-bearing. The four views are all derivable from
one table set:

- graph edges = controls mapped to more than one market
- blast radius = requirements joined to controls joined to markets, filtered by scenario
- control heartbeat = last evidence timestamp per control
- register health = null counts on owner, evidence and control mapping
- control anatomy = the ten fields on the control record, with `provable` computed
- automation layer = the scenario table, with its stop point as a required field

**Reconcile your numbers before shipping.** The strip, the panels and the views all state
counts, and readers do add them up. An earlier version of this file said `REQUIREMENTS 16`
in the strip while Register health said `verified source 11/17` and `illustrative 6` —
11 + 6 = 17, not 16. On a design exercise arguing for register discipline, an unreconciled
count is the most expensive kind of typo. The durable fix is not to check harder but to
compute: `UNPROVABLE` in the strip is written from `provable()` at load, so it cannot
disagree with the control-anatomy view no matter how the register changes.

---

## Checklist

- [ ] File is named `index.html` at the repository root
- [ ] `viewport` meta present
- [ ] No raw hex values outside `:root`
- [ ] Both SVGs have `viewBox`, `role="img"` and a descriptive `aria-label`
- [ ] Animation durations vary per element
- [ ] No `animateMotion` circle paints at the origin before its `begin` time
- [ ] `prefers-reduced-motion` honoured
- [ ] Every injected string passes through `esc()`
- [ ] Expandable rows respond to Enter and Space, not only click
- [ ] Wide tables scroll inside `.tscroll`, not by moving the page
- [ ] Selector change resets the simulator
- [ ] No automation path in any diagram is unlabelled
- [ ] Every scenario row states a stop point
- [ ] `provable` is computed, never typed, and written to every place it appears
- [ ] Counts reconcile across strip, panels, graph and anatomy
- [ ] Real versus synthetic data labelled everywhere
- [ ] Provenance disclaimer in both header and footer
- [ ] Renders at 360px wide
- [ ] Console clean

---

## Be able to explain it without notes

The panel is only half the deliverable; the other half is being able to talk through it.
Three questions it should let you answer cold:

1. **Where Make sits, what it does, and the one thing it must never do.** It sits in the
   tooling lane between ingest and the system of record. It creates, routes, chases,
   watches, reconciles and logs. It must never dispose — never decide whether a
   discrepancy is acceptable, whether an alert is a true match, or whether a risk can be
   accepted.
2. **Why training and effective-date tracking are stages, not admin.** A control that
   depends on people behaving differently does not exist until those people know, and
   training completion is the evidence an auditor asks for. A regulation has a
   commencement date, and being ready two weeks late is a breach even if the work was
   excellent — so the countdown is a tracked stage with a blocking alert, not a line in a
   project plan.
3. **How you would prove, six months later, that a given control operated.** Query the
   immutable log for the period, join to the evidence store on the requirement tag, and
   produce every instance with its actor and its dated artifact. That answer only exists
   if what happened, who did it and what it produced were all captured at the moment the
   control ran — which is why `provable` is a computed field and why five of sixteen
   controls currently fail it.
