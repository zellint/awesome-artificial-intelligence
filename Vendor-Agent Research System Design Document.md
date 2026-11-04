# Vendor-Agent Research System: Design Document

*Draft v1 · November 4, 2026 · Main document (Sections 1–8) and Appendix (A–H)*

## 1. Summary

We buy data from many vendors: news, company financials, analyst forecasts, economic indicators. Our edge comes from combining them in ways that other firms have not thought of. Today, finding those combinations depends on what individual researchers think to try, and much of what is needed to combine data safely (when the data was really available, where it is unreliable, which vendors secretly share inputs) lives in people's heads.

This document proposes a system in which **each data source has its own AI expert**, built from measured facts about the data rather than from the vendor's marketing. A coordinator picks promising pairs of experts, who meet in small groups, work out how their data might be combined, and write up proposals. A director with limited testing capacity decides which proposals get tested. Our existing testing process checks the winners, and the results teach the system what kinds of combinations work.

The main difference from the common approach (a human has an idea and an AI finds and builds the data) is the direction: **the system generates ideas for a human to approve, rather than waiting for a human to ask.** The back end (building and publishing datasets) can stay exactly as it is.

The proposal is deliberately testable. Section 7 describes a small pilot with clear tests, including a comparison against simpler alternatives. If the simpler alternatives do as well, we use them.

## 2. The problem

1. **Too many combinations.** With dozens of sources, the number of possible pairs, triples and conditions is far more than people can examine.
2. **Obvious ideas give no edge.** Pairing news with fundamentals, or analyst revisions with sentiment, is what every firm tries. If our process only finds ideas a human would think of first, we find what competitors find.
3. **Hidden data quirks.** Many ideas fail not because the economics is wrong, but because of details: a value stamped on one date but actually available later, a history that was quietly rewritten, two vendors that both consume the same underlying feed. This knowledge is scattered and rarely written down.
4. **Many datasets are already model outputs.** A vendor's "sentiment score" may already include analyst data, so combining the two can double count one source of information.

## 3. The proposal on one page

```
  Data cards (measured facts about every source)
        |
        v
  Consolidation: group vendors that sell essentially the same thing
        |
        v
  Matchmaker: choose which pairs of sources to examine
        |
        v
  Small sessions: 2 data experts + a skeptic
        |   (each expert first thinks alone, then they compare)
        v
  Written proposals in a standard format
        |
        v
  Cheap screening -> Director allocates limited testing capacity
        |
        v
  Existing validation process + record of everything tried
        |
        v
  Human sign-off -> existing build-and-publish pipeline
        |
        +--> Results feed back into the matchmaker and expert scores
```

| Step | In plain terms |
| --- | --- |
| Data cards | A profile of each source, produced by code from the real data: timing, coverage, where it predicts, how it overlaps with what we already own |
| Vendor experts | One AI agent per source that reads its card and can run checks on the real data |
| Consolidation | If five vendors sell near-identical data, treat them as one family so we do not test the same idea five times |
| Matchmaker | Picks pairs to examine, favouring complementary data and unfamiliar pairings |
| Sessions | Short, structured meetings of two experts and a skeptic |
| Proposals | Each idea is written in a fixed format: why it should work, what data it needs, expected performance with uncertainty, and how we would prove it wrong |
| Director | Allocates scarce testing capacity across a varied set of proposals |
| Validation | Our existing testing process, with a log of every idea tried |
| Human sign-off | A person approves before anything is built or published |

## 4. Key components

### 4.1 Data cards

A data card is a document, built automatically and kept up to date, that states what a data source actually does. It covers: what the data is and where it comes from; how often and how fast it arrives; whether history is trustworthy; how much of our stock universe it covers; where it has predictive power and how that fades over time; and how much it overlaps with signals we already have. Every number carries a measure of uncertainty and a link to the check that produced it. Full contents are in Appendix D.

### 4.2 Vendor experts

Each expert reads its own card and may run a limited number of extra checks on the data. It does not rely on memory or the vendor's documentation alone. Its job is to state what its data is good at, where it is weak, and what questions it cannot answer (Appendix E).

### 4.3 Matchmaker

Nobody wants every expert in one meeting: it is expensive and degenerates into noise. Instead the matchmaker chooses a limited number of pairs per round. It is designed to find *non-obvious* pairs, not the textbook ones, by looking at what each source misses, how their timing differs, and where they disagree, while penalising pairings that match the obvious combinations everyone tries (Appendix A).

### 4.4 Small sessions

Two data experts and a skeptic. Each expert writes down privately and simultaneously what its data is good at, where it is weak and what it cannot answer. Only then are the notes shared. They are not forced to combine: "there is nothing here" is an accepted and rewarded outcome (Appendix B).

### 4.5 Director and selection

Testing capacity is limited. A director decides which proposals get tested, acting like a manager allocating a budget. It is judged on the long-run value of its choices, not on how convincing the pitches sound (Appendix F).

### 4.6 Testing, record keeping and learning

Selected proposals go through the existing validation process. A log records every idea tried, so we can correct for the fact that trying many ideas will produce lucky results by chance. Outcomes update the matchmaker's priorities and each expert's reliability score.

## 5. Safeguards

| Risk | Safeguard |
| --- | --- |
| Experts "know" things that are wrong | Knowledge comes from measured data cards and live checks. Every claim is labelled as verified, from documentation, or a belief; unsupported "verified" claims are rejected automatically |
| Overselling | Each pitch states an expected result with uncertainty. After testing, the prediction is scored. Overclaiming costs the agent influence (Appendix C) |
| Finding luck instead of signal | A log of every idea tried, with results adjusted for the number of attempts. Non-obvious ideas face stricter, not looser, evidence requirements |
| Final test data leaks into the process | The final test period is visible only to the validation process. Agents and the director see only earlier data and a limited query budget |
| Double counting between vendors | Each source records what it consumes upstream. Vendors with shared inputs are grouped in consolidation |
| System invents ideas for the sake of it | We track how many sessions end with "nothing found". A healthy system should have a large share |
| Agents become more expensive than they are worth | Fixed session budgets, cheap screening first, and a simpler baseline we can fall back on |
| Loss of human control | A human signs off before anything is built or published |

## 6. Comparison with the common pipeline

### 6.1 The common pipeline

Many teams already run a pipeline of this shape:

1. A single AI assistant knows the whole data lake through documents, manifests of data modules, and retrieval over descriptions of individual fields.
2. A human proposes an idea.
3. The assistant searches its knowledge for data fields that fit the idea.
4. Human and assistant discuss how to improve it.
5. The human signs off the design.
6. The assistant helps construct the dataset using a dedicated framework, and publishes it.

### 6.2 Side by side

|  | Common pipeline | Proposed system |
| --- | --- | --- |
| Who starts | A human poses an idea | The system proposes ideas; a human reviews them |
| Role of the AI | Librarian: finds fields that fit the idea | Researcher: generates combinations nobody asked for |
| Source of AI knowledge | Documents, manifests, retrieval over field descriptions | Measured data cards: timing, coverage, predictive behaviour, overlap |
| Number of agents | One that knows the whole lake | One per source, plus skeptic and director |
| Search space | Limited by the human's imagination | Sampled across pairs and triples by a matchmaker |
| Controls on accuracy | Human judgment during discussion | Evidence-backed claims, scored predictions, reputation effects |
| Learning over time | Little systematic feedback | Outcomes update pair priorities and agent reliability |
| Output | A built, published dataset | A ranked set of tested proposals for human sign-off |
| Where the human sits | At the start and at sign-off | At sign-off, and in reviewing the proposal list |

### 6.3 Pros and cons of the common pipeline

**Pros**

- Simple, cheap and already running, with a clear and proven build-and-publish path.
- Human judgment and accountability at the start of the process, with the human's intuition steering the work.
- Low risk of nonsense: nothing is attempted that a person did not ask for.
- Easy to explain and to audit.

**Cons**

- **Only as original as the person asking.** Ideas converge on the same obvious combinations as other firms, which is where edge is weakest.
- **Knowledge comes from reading, not measuring.** Documentation states what vendors claim; it rarely reveals real timing, quiet history rewrites, coverage gaps, or hidden overlap with other vendors.
- **One generalist assistant.** It knows a little about everything and has no detailed, checkable view of any one source.
- **Little learning.** There is no systematic record connecting ideas to outcomes, so the process does not improve itself.
- **Scales with human attention.** More vendors mean more combinations to consider, but the same number of researchers.

### 6.4 Pros and cons of the proposed system

**Pros**

- Explores combinations nobody thought to request, including ones across unrelated categories.
- Knowledge is measured and reproducible, so claims can be checked.
- Built-in controls against overselling and against false discoveries.
- Learns: each tested idea improves the next round.
- Scales with compute rather than with headcount.
- Reuses the existing back end, so the new work is limited to the front end.

**Cons**

- **More to build and run:** data cards, diagnostics, matchmaker, session machinery and a log of trials.
- **Higher cost per idea** (computing and agent usage), and the need to keep cards up to date as vendors change.
- **More ideas also means more chances of false discoveries.** This is manageable with the trial log and stricter evidence thresholds, but it needs discipline.
- **The multi-agent part may not beat a simpler setup.** A single assistant with data cards might do as well. This is a hypothesis to test, not an assumption.
- **Harder to explain and audit** than a single assistant, although the structured proposals and run records help.
- **Unproven in our setting.** The first results will tell us how many non-obvious ideas survive testing.

### 6.5 How they fit together

The two approaches are not competitors. They sit at opposite ends of the same workflow.

```
 Proposed system: finds and refines candidate ideas
        -> human reviews and signs off
        -> existing pipeline: builds and publishes the dataset
```

The proposed system replaces "a human poses an idea" with "the system proposes, a human selects". Everything after sign-off stays as it is today.

**Recommended path**

1. Add data cards to the existing assistant's knowledge. This alone should improve its answers about timing, coverage and overlap.
2. Add an optional "suggest ideas" mode based on pair sessions, and compare its suggestions with human ones.
3. Expand to more sources only if the earlier steps show clear value.

Each stage can be judged on its own results before the next is built.

## 7. Pilot plan and success criteria

### 7.1 Scope

Three data sources from different categories (for example fundamentals, news or text-based model output, analyst revisions), a basic matchmaker, one skeptic, the standard proposal format, and a connection to the existing validation process. The first milestone is to show that data cards and live checks make the experts' claims reliable, before adding more sources.

### 7.2 Tests

| Test | What it answers | How |
| --- | --- | --- |
| Rediscovery | Can the system find things that are known to work? | Hide a combination we already use and see whether the system proposes it |
| Simple baseline | Do agents add anything over brute force? | Compare against listing source pairs against a fixed set of standard templates, with no agents |
| Single assistant | Does having one expert per source matter? | Compare against one assistant reading both data cards and proposing |
| Director versus rule | Does the director add value? | Compare against picking the top proposals by screening score with a diversity rule |
| Grounding | Are experts' claims accurate? | Independently verify a sample of their claims |

### 7.3 Measures

- Share of sessions ending with "nothing found" (should be substantial).
- Share of proposals that survive screening and then validation.
- Duplicate rate across sessions.
- How well proposals' stated expectations match results.
- Accepted ideas: how much they add beyond signals we already have, and how quickly their performance fades.
- Cost per accepted idea.

### 7.4 Decision rule

Thresholds for each measure should be agreed with the team before the pilot starts, so that results cannot be interpreted after the fact. Where a simpler baseline matches the full system, we adopt the simpler approach for that step and stop building the more complex one.

## 8. Risks and open questions

- **Vendor history.** Verifying timing and history quality needs snapshots of how data looked at the time. If a vendor only provides rewritten history, some claims cannot be verified, and the card says so.
- **Raw feeds versus model scores.** How much of our vendor data is raw versus already modelled determines how much effort goes into tracking shared inputs versus verifying timing.
- **Number of sources.** With fewer than about 20 sources that survive filtering, a simple shared board and matchmaker is probably enough. With more than about 50, the consolidation step becomes essential.
- **Crowding.** Non-obvious does not guarantee unique. Cards should include indicators of how widely a dataset is sold and whether similar effects have faded.
- **Operating cost.** Session budgets and the screening ladder need tuning during the pilot.
- **Ownership and process.** Who reviews the proposal list, and how often, needs to be decided with the research team.

---

# Appendix

## A. How matchmaking finds non-obvious pairs

Grouping by category (news with fundamentals) finds what every firm finds. The matchmaker therefore pairs by what the data actually does.

- **Gaps.** For each source, measure what it fails to explain. Pair sources where one's blind spot is the other's strength, even if they come from unrelated fields, such as shipping data and analyst forecasts.
- **Timing.** Some sources react to the same event earlier than others. Pair sources with a consistent lead and lag, since the earlier one may act as a warning for the later one.
- **Disagreement.** Pair sources that usually agree but sometimes diverge sharply. The unusual moments are often where information is hiding.
- **Penalising the obvious.** Ask a standard AI model, and consult our past work and published research, for the most obvious combinations. Pairs that match are penalised.
- **Reserved exploration.** A fixed share of sessions (for example 20%, to be tuned) goes to randomly chosen, far-apart pairs. Most will fail, but this is the only way to find ground nobody has searched.
- **Learned priors.** Past outcomes for each type of pair and each type of interaction raise or lower the pair's priority. Selection uses a method that balances exploiting what has worked and exploring what has not been tried, within a fixed session budget per round.
- **Growing combinations.** Triples are formed by extending a pair that already looks plausible, not by listing all triples.
- **Conditioners.** Regime and macro sources attach as an optional third party where it makes sense to switch a signal on or off by market state, instead of pairing with everything.

**Honest caveat.** The less obvious an idea is, the more likely it is to be coincidence. Non-obvious pairs therefore face stricter evidence requirements. Part of the edge is also practical difficulty: combinations that need rare data, careful cleaning and vendor-specific knowledge are hard to copy even when the idea is known.

## B. Session protocol: who speaks first

Nobody speaks first, and nobody is told they should combine.

1. **Private, simultaneous notes.** Each expert writes what its data is good at, where it is weak, and what it cannot answer. It states what *kind* of information would complement it (for example "I react slowly to events, so a faster signal would help") without naming a partner.
2. **Reveal and cross-examination**, at most two rounds. Every claim must cite the data card or a check result.
3. **Skeptic pass.** The skeptic looks for use of information that was not yet available, shared inputs between vendors, crowding, capacity limits and the absence of any real mechanism.
4. **Write-up.** Surviving ideas are converted into the standard proposal format, with a cap on proposals per session.

Why combining must earn its place: the starting assumption is that the sources stay separate. A combination survives only if it clearly beats each source alone and a simple average of the two. Experts are rewarded for correctly concluding "nothing here". If almost every session ends in a proposal, that is a warning sign that agents are being pushed to invent.

To keep diversity up, agents can be given different starting perspectives, and quick agreement is treated as a warning rather than success. Real diversity comes from differences in data access and, where possible, from using different underlying AI models, not just from different personas.

## C. Keeping agents honest

An AI agent does not feel pain, so the cost of being wrong has to be built into how the system works. The currency is influence.

- **Labelled claims.** Every statement is marked verified (with a link to the check), from documentation, or belief. Unverified statements carry less weight. A statement marked verified without a supporting check is automatically downgraded.
- **Scored predictions.** Pitches state an expected result with a confidence range. After testing, these are scored; overclaiming is penalised more than modest, accurate claims.
- **Spot checks.** A random sample of each agent's claims is independently verified. A false claim that is caught costs a lot.
- **Reputation drives influence.** The score affects how much weight an agent's pitches get, how many sessions it joins, and how much testing capacity its ideas receive. The agent's track record is also shown back to it, so it sees which past claims were wrong.
- **The skeptic is scored too.** It is rewarded for flaws that turn out to be real and penalised for false alarms, so it cannot win by blocking everything.
- **Hidden final test.** Agents cannot tune claims to the final exam.

## D. What a data card contains

A data card is a versioned document produced by code, with no AI involved in producing the numbers.

**1. Identity and provenance (from the vendor, checked by us)**

- What it is: vendor, product, type (raw feed, model score, forecast, event stream), fields, definitions and units.
- Methodology and version history, including dated changes.
- Upstream dependencies: what the vendor itself consumes.
- Licensing and operational limits.

**2. Timing and structure (measured)**

- Update cadence and latency.
- Point-in-time audit: whether history reflects what was known at the time, or was backfilled or overwritten, with a trust score for the history.
- Revision profile: how often and how much values change afterwards.
- History length, gaps, and the number of genuinely independent observations.

**3. Coverage and quality (measured)**

- Share of our stock universe covered, by country, sector, size and liquidity, and how it changes over time.
- Missing-data patterns (random or systematic).
- Value distribution and stability.
- A log of detected jumps, level shifts and stale values.

**4. Predictive behaviour (measured, development period only)**

- Predictive power at several horizons and how it fades, with uncertainty.
- The same split by sector, size, liquidity and market regime.
- Stability over time, including tests for structural breaks.
- Speed of response after events, where relevant.
- Trading turnover, capacity and cost sensitivity.
- Statistical power: the smallest effect the history could detect.

**5. Relationship to everything else (measured)**

- Overlap with our existing signals, and what remains after removing it.
- Overlap with other vendors and shared-input flags.
- A numerical profile of where it is strong and weak relative to others, read directly by the matchmaker.
- Crowding indicators.

**6. Metadata and trust**

- Card version, data snapshot and code version, so every number is reproducible.
- A link from every statistic to the check that produced it.
- Freshness and drift flags showing what changed since the last build.
- The cutoff dates separating development from final test data; the card never contains final-test statistics.
- Open questions: what the checks could not establish.

**Good practice.** Every statistic carries a confidence interval or sample size. A one-page summary sits at the top, with detail behind it. Facts are labelled by how we know them: stated by the vendor, measured by us, or unknown.

## E. How an expert can state strengths, weaknesses and limits

The expert does not recall these things. Standard code produces the numbers, and the expert interprets them, notices what they do not show, and proposes follow-up checks.

| The expert says | Established by |
| --- | --- |
| **Good at** | Predictive power by horizon and by group; response speed around events; added value after removing what existing signals and risk factors already explain |
| **Weak at** | Missing-data maps; timing audit; revision and backfill profile; stability and break tests (which catch vendor methodology changes); turnover and capacity |
| **Cannot answer** | Cadence versus horizon (monthly data cannot speak to daily timing); statistical power (a regime that appears twice in the history cannot be assessed); information type (the data shows what moved, not why); entity and event coverage |

**Toolkit**

1. Read-only, as-of access to historical snapshots. Without snapshots the timing and revision claims cannot be verified.
2. A library of standard, tested diagnostic functions, so results are reproducible and comparable across vendors.
3. Read-only access to the existing signal pool and risk factors, and to vendor documentation and change notes.
4. A structured output format in which each claim has a type, a link to the supporting check, and a confidence level. A validation step rejects any claim marked verified without a link.

**Workflow.** The cards are built offline on a schedule. At the start of a session the expert reads its own card and writes its three lists, with a limited budget for extra checks. It sees only its own data and a summary of the existing pool, not its partner's data.

**Pitfalls built around**

- Profiles are built only on the development period, so they do not overfit to the final test.
- Mechanism stories ("this works because investors underreact") are labelled hypotheses unless a check supports them.
- Cards are rebuilt when a vendor refreshes data, and statistics that moved significantly are flagged.

**Minimum kit for the pilot:** the timing audit, predictive power by group, added value after removing existing signals, missing-data maps, and the statistical power calculation.

## F. Director and capital allocation

The director is a portfolio manager of research ideas, with testing slots as the scarce resource. It should not be an audience to be persuaded.

**Why "maximise PnL" is the wrong goal.** A director optimising backtest profit will fund whatever looked best in the past data, and with many sessions some ideas will look good by chance. The goal should instead be the expected marginal contribution to the existing portfolio, net of costs and of the number of ideas tried, measured out of sample.

**Why persuasion is the wrong mechanism.** Rewarding agents for convincing the director rewards rhetoric. The numbers in the pitch, not the prose, are what matters.

**How it works**

1. **Pitch.** A structured one-pager with an expected result and confidence range.
2. **Questions.** The director and the skeptic can request checks, such as results by sub-period or overlap with a given existing signal.
3. **Decision.** Fund, revise or kill, with a recorded reason.
4. **Settlement.** After validation, the stated expectations are scored. Agents build a reputation for calibration.

**Where the director sits.** After cheap screening, at batch selection, so it sees competing proposals together and can pick a varied set (not ten variants of one idea) while respecting the trial log.

**Test of value.** Compare the director's picks with a simple rule over several hundred decisions. If the director does not do better, the rule replaces it.

## G. Topology: how agents are connected

The core idea is to separate *who meets whom* from *what happens in a meeting*.

- **Hierarchy, then shared board, then small sessions.** Within each category, vendors are first consolidated into families. Experts publish structured entries to a shared board (what they provide, what they miss, proposals, critiques) rather than messaging each other. The matchmaker reads the board and samples pairs.
- **Small sessions only.** Most useful combinations involve two sources, sometimes with a third as a switch. Sessions of two to four agents keep conversations short and cheap, and each agent only loads its own card.
- **Cost.** Communication through the board grows in proportion to the number of sources, not the square of it.

**Alternatives considered and rejected**

- *Everyone in one meeting:* cost grows with the square of the number of sources, and the discussion becomes noise.
- *One central moderator for all discussions:* becomes a bottleneck and cannot judge data it does not know. Acceptable for scheduling, not for content.
- *Fixed category-to-category routing alone:* simple, but never discovers unexpected pairings such as a macro series interacting with an ownership dataset.

**After sessions.** Near-duplicate proposals from different pairs are merged. Cheap tests come first, then a successive narrowing of candidates before anything reaches the expensive validation step. A final batch is chosen for variety.

## H. Glossary

| Term | Plain meaning |
| --- | --- |
| **Signal** | A number computed from data that is used to decide which stocks to hold or avoid |
| **Alpha / edge** | Return we can earn that is not just general market movement or something other firms already capture |
| **Backtest** | Testing an idea on past data to see how it would have performed |
| **Point-in-time** | Using only what was actually known on each past date, not information that was published or corrected later |
| **Look-ahead error** | Accidentally using information that was not yet available, which makes results look better than reality |
| **Overfitting / false discovery** | Finding patterns that are coincidence in the past data and do not repeat |
| **Holdout / final test data** | Data kept hidden until the end to check whether an idea really works |
| **Information coefficient (IC)** | A measure of how well a signal predicts future stock returns |
| **Regime** | A distinct market state, for example calm versus stressed, in which signals may behave differently |
| **Data card** | The measured profile of a data source described in Appendix D |
| **Calibration** | How well an agent's stated confidence matches what actually happens |
| **Trial log** | A record of every idea tested, used to adjust results for the number of attempts |
