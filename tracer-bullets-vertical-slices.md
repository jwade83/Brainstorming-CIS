# Tracer Bullets & Vertical Slices for AI Coding Agents

> "AI loves to code horizontally — layer by layer. Phase one: all the database stuff. Phase two: all the API stuff. Then the front end on top. What's wrong with that picture? You don't have the whole feedback loop."
> — Matt Pocock, *AI Engineer Europe* (clipped by [@thedailyai.news](https://www.tiktok.com/@thedailyai.news))

Why AI coding agents stall on multi-phase plans, and the planning technique that fixes it. The idea is old (it comes from *The Pragmatic Programmer*, 1999), but it turns out to matter *more* when the thing executing the plan is an agent that can't feel its way through a codebase the way a person can.

---

## Quick answer

- A **layer** is a unit of the system that other units depend on: database, API, front end — or, inside one service, the quiz service, team service, user service, coupon service.
- A **horizontal plan** works one layer at a time: all the schema, then all the endpoints, then all the UI. Nothing end-to-end runs until the last phase.
- A **vertical slice** (a.k.a. **tracer bullet**) is one thin piece of *user-visible* functionality that cuts through every layer it needs. It is small, but it is complete.
- Horizontal plans fail with agents because the **feedback loop is broken**: the agent writes hundreds of lines "blind" and only discovers the integration mistakes at the end.
- Vertical plans succeed because at the end of phase one, or *during* phase one, the agent can run the entire flow and get real feedback.
- Rule of thumb: **every task you hand an agent should be something you could demo.** If it can't be exercised end to end, it's a layer, not a slice.

---

## Deeper detail

### Where the term comes from

Hunt and Thomas coined *tracer bullets* in *The Pragmatic Programmer*. Tracer rounds are loaded among ordinary ammunition so a gunner can see, in real time, where the shots are actually landing and adjust — instead of calculating a firing solution in advance and hoping.

Applied to software: rather than fully building each component in isolation, you write the *thinnest possible path* that goes all the way from the user's input to the system's output, through every architectural layer. It's ugly, it handles one case, but it **runs**. Then you thicken it.

Two things distinguish a tracer bullet from a prototype:

| | Prototype | Tracer bullet |
|---|---|---|
| Purpose | Explore one question, then throw it away | Become the skeleton of the real system |
| Code quality | Disposable | Production-grade, just narrow |
| Scope | Usually one layer or concern | All layers, minimal width |
| What happens next | Rewrite | Incrementally extend |

*Vertical slice* is the same idea from the agile world (and *walking skeleton* / *steel thread* are close cousins). A slice is a unit of work defined by a **user outcome**, not by an architectural tier.

### Why horizontal plans are the natural default — for humans and agents

Layered plans *look* rigorous. They map one-to-one onto the architecture diagram, each phase has an obvious owner, and "finish the schema before the API" feels like sound engineering. Humans fall into it constantly; LLMs fall into it even harder because a plan grouped by layer is the most statistically likely shape for a plan to have.

The cost is invisible until the end:

1. **No integration signal.** The database layer can be "done" and still wrong for the API that hasn't been written yet — wrong nullability, wrong ID type, missing an index the query will need. Nothing detects it until phase two, and nothing detects the API/front-end mismatch until phase three.
2. **Nothing to test.** You can unit-test a layer, but the assertions are guesses about how the layer will be consumed. The true test — *does the feature work?* — is impossible until the last phase.
3. **Rework compounds.** A mistake in phase one is discovered in phase three, by which time two layers were built on top of the mistaken assumption.
4. **Zero demo value until the end.** Nothing can be shown, reviewed, or shipped in the meantime.

### Why it hurts agents specifically

Everything above applies to a human team too, but an agent has a few extra weaknesses that horizontal work aggravates:

- **Agents don't get intuition for free.** A senior engineer building "the database phase" carries a mental model of the API and UI that will consume it. An agent has only what's in context and what it can *observe*. Take away the ability to run the flow, and it's coding on assumptions.
- **Agents drift.** The longer they work without a hard check, the further small misreadings compound. A slice gives a hard check (a test, a running request, a rendered page) every few hundred lines instead of every few thousand.
- **Agents verify by running things.** Their strongest correctness tool is executing code and reading the output. A horizontal phase produces code that *cannot be run meaningfully* until other phases exist, so that tool is switched off exactly when it's needed most.
- **Context is finite.** By phase three the details of phase one have been summarised away. If phase one was already validated end to end, that's fine. If it wasn't, the agent is now debugging integration issues in code it barely remembers.
- **Self-directed task selection needs a good unit of work.** Pocock's point is that tracer bullets change how you let the AI *choose its own next task*. If the backlog is a list of thin, demonstrable slices, "pick the next one and make it pass" is a safe instruction. If the backlog is a list of layers, "pick the next one" yields hours of unverifiable output.

### What a good slice looks like

A slice is defined by an observable behaviour, and it names every layer it must touch:

> **Slice:** A user can apply a valid coupon code at checkout and see the discounted total.
> **Touches:** `coupons` table (one row, one column set) → coupon service `validate(code)` → `POST /checkout/apply-coupon` → checkout page input + total re-render.
> **Done when:** an end-to-end test enters `SAVE10` and asserts the total dropped by 10%. Invalid codes, expiry, stacking, and admin UI are **out of scope** for this slice.

Compare the horizontal version of the same feature:

> Phase 1: design and migrate the full coupons schema (codes, expiry, usage limits, stacking rules, audit log).
> Phase 2: build the coupon service and all endpoints (CRUD, validate, redeem, report).
> Phase 3: build the checkout UI and the admin UI.

The first version produces a working, testable, demoable feature in one sitting. The second produces three sittings of untested code and a fourth sitting of integration debugging.

### Slicing heuristics

- **Start with the happy path.** One valid input, one correct output. Error handling, edge cases, and admin tooling are later slices.
- **Hard-code what you can.** If the slice needs a "current user", pass a fixed user ID. Auth is its own slice.
- **Fake the expensive layer, but only on the edge.** A slice may stub a third-party payment API. It should *not* stub your own database or your own API — those are precisely the layers whose integration you're trying to prove.
- **Prefer width over depth for the first slice.** It should touch *every* layer with the *least* logic per layer. Subsequent slices add depth.
- **Each slice ends with a runnable check.** A test, a `curl`, a screenshot. If there's no check, the slice isn't finished.
- **Order slices by risk, not by layer.** Do the slice that proves the scariest integration first (e.g. the one that crosses the service boundary you've never crossed).

### Anti-patterns to catch in an agent's plan

Signs a plan has gone horizontal even when it claims to be sliced:

- Phase headings that are architectural nouns: *"Database"*, *"Backend"*, *"Frontend"*, *"Models"*, *"Services"*.
- A phase whose deliverable can't be exercised by a user or a test.
- "We'll write the tests in the final phase."
- A first phase that creates every table / type / interface the feature will *ever* need.
- Any phase described with the word "all" ("all the endpoints", "all the schema").

If you see these, send the plan back: *"Re-plan as vertical slices. Each task must be a user-visible behaviour that crosses every layer it needs and ends with a passing end-to-end check."*

### Relationship to nearby ideas

| Idea | Relationship |
|---|---|
| **Walking skeleton** (Cockburn) | The *first* tracer bullet: the tiniest end-to-end implementation, wired into the real build/deploy pipeline. |
| **Steel thread** | Same concept, common in large-system and defence contexts. |
| **MVP** | Product-level scope; a tracer bullet is engineering-level scope. An MVP is usually several slices. |
| **Feature flags** | Let you ship thin slices to production without exposing half-finished features. |
| **TDD** | Complementary. The end-to-end test that defines "done" for a slice is the outer loop; unit tests are the inner loop. |
| **Spike / prototype** | Throwaway learning. A tracer bullet is kept. |

---

## How to apply it in this repo

This repo has no test stack yet. When one gets picked, the first task should itself be a tracer bullet: **one trivial test, running locally and in CI, in the chosen language.** Not "set up the whole tooling" — just the thinnest path from "commit pushed" to "green check". Everything else (linting, coverage, fixtures) is a later slice.

Prompt template to reuse when planning with an agent:

```
Plan this feature as vertical slices, not layers.
Each slice must:
  1. describe one user-visible behaviour,
  2. list every layer it touches (db → service → api → ui),
  3. end with a runnable end-to-end check that proves it works,
  4. leave out everything not needed for that behaviour.
Order the slices by integration risk. Start with the happy path.
Do not group work by layer.
```

---

## Likely follow-up questions

- How thin is *too* thin? (If a slice has no observable behaviour, it's too thin. If it touches a layer with more than the minimum logic, it's too thick.)
- What if the layers are owned by different teams or different deployables? (Slices still cross them; that's the point. Coordinate the *slice*, not the layer.)
- Does this mean no upfront schema design? (No. Design the direction; *implement* only what the current slice needs. Migrations are cheap; unverified assumptions aren't.)
- How does this interact with agent context limits? (Favourably: each slice is a bounded, self-verifying unit, so context can be reset between slices with little loss.)
- What's the difference between a tracer bullet and a prototype? (Tracer bullets are kept and extended; prototypes are discarded.)
