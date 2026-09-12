# FinePrint

**Contracts made clear.** A working prototype that reads a residential lease and tells a first-time renter three things a summary won't: what each clause costs in dollars, whether it's normal or unusual, and the exact sentence to ask for instead.

▶ **[Live demo](#)** · single HTML file, no build step, no backend, no API key

---

## Where this came from

I signed my first apartment lease the way most 22-year-olds do: skimmed four pages, recognized about a third of the words, signed. Months later I needed out and discovered the lease banned subletting, banned assignment, and banned substitute occupants — three separate clauses in three different articles, all doing the same job. There was no exit. I'd agreed to that in writing without ever knowing it was a question.

FinePrint started as a team project in an entrepreneurship course, built around that experience. This repo is my own rebuild of it, two years later, as a working model.

## Why this isn't a summarizer

In 2024, "upload your lease, get plain English back" was a product. It isn't anymore — anyone can drop a PDF into a chat assistant and get a section-by-section translation in ten seconds, free. **If the product is summarization, the product is dead.** Building this in 2026 only makes sense if it does things a chat window structurally doesn't.

Three things, which are the three the demo is organized around:

### 1. It reads clauses against each other

The sharpest finding in the demo isn't in any single clause. §3.4 makes each roommate jointly and severally liable for the full $2,680 rent — unremarkable, it's in 79% of group leases. §14.1 bans substitute occupants — harsh, but legible on its own.

The finding is the pair. **You can be billed for a roommate who disappeared, and you are contractually forbidden from replacing them.** Those two clauses are eleven articles apart and were almost certainly drafted by different people at different times. A tool that walks a document top to bottom and summarizes each section will never put them in the same sentence, because to a section-by-section reader they aren't related.

Clause interaction is the unit of analysis. Not the clause.

### 2. It's grounded in one jurisdiction, and half the value is in what the statute *doesn't* say

Every legal note in the demo cites real Indiana Code, verified against the current code:

| Finding | Statute | Why it matters |
|---|---|---|
| Deposit auto-deductions (§5.2) | [IC 32-31-3-12](https://law.justia.com/codes/indiana/title-32/article-31/chapter-3/section-32-31-3-12/) | Deductions are limited to accrued rent, damages from tenant noncompliance, and unpaid utilities. Landlord must itemize within **45 days** or the tenant recovers the deposit **plus attorney's fees**. |
| Entry notice (§11.2) | [IC 32-31-5-6](https://codes.findlaw.com/in/title-32-property/in-code-sect-32-31-5-6/) | Requires "reasonable written or oral notice." **Sets no fixed hour count** — so "as practicable" is weak, not illegal. |
| Habitability waiver (§16.4) | IC 32-31-8-4 / -5 / [-6](https://law.justia.com/codes/indiana/title-32/article-31/chapter-8/section-32-31-8-6/) | A lease can't contract out of the landlord's repair obligations. The waiver is **void**, and §32-31-8-6 restores attorney's fees. |
| Sublet ban (§14.1) | Title 32, Art. 31 — *no statute* | Indiana grants **no default right to sublease**. There is no state-law floor beneath this clause. |

That last row is the one I'd point at in an interview. The absence of a statute is a finding. It's what turns "this clause is strict" into "this clause is the entire universe of your exit rights, so it has to be fixed before you sign." A general-purpose assistant will tell you the clause is strict. It won't tell you there's nothing underneath it, because it isn't reasoning about a specific state's code as the backstop.

The same logic flips the other way on entry notice. Indiana sets no hour requirement, so "as practicable" isn't a violation — it's just worse than what 76% of landlords offer voluntarily. **That distinction changes what you ask for**, and getting it wrong makes you look uninformed in front of a landlord.

### 3. It ends in a sentence you can send

Understanding a clause doesn't change it. Every finding carries a redline — what to strike, what to put in its place — and a message ready to paste into an email. The product is leverage, not literacy.

This is also the honest answer to "why would anyone pay for this." Nobody pays $10 to understand their lease. They pay $10 the night before signing, when understanding converts into an ask.

---

## What's in the repo

```
index.html              the demo — self-contained, ~1,200 lines, zero dependencies
docs/sample-lease.md    the synthetic lease the demo analyzes (14 pages, 47 provisions)
README.md               this file
```

Open `index.html` in a browser. That's the whole install.

## What's real and what's synthetic

Stated plainly, because a portfolio piece that blurs this is worse than one that doesn't exist:

| | |
|---|---|
| **Real** | Every Indiana Code citation and every characterization of what the statute says. Verified against the current code, sources linked above. |
| **Synthetic** | The lease. Written for this demo, modeled on patterns common in Midwestern student-housing leases. The parties, property, and figures are invented. |
| **Illustrative** | The 1,247-lease benchmark corpus. The percentages are plausible but they are **sample data, not research**. In a production build they'd come from filed and user-submitted leases. |
| **Static** | The analysis is authored, not generated at runtime. This demo models the output, not the pipeline. |

None of it is legal advice.

## How the production version would work

The demo deliberately stops at the output, because the output is where the product decisions live. The pipeline behind it isn't the hard part, but for completeness:

1. **Ingest** — PDF → text with layout retention; most leases are digital-native, scanned ones need OCR.
2. **Segment** — split into numbered provisions. Harder than it looks: numbering schemes are wildly inconsistent, and incorporation-by-reference (§15.1 pulls in Rules and Regulations posted on a website) means the document isn't the whole contract.
3. **Classify** — map each provision to a clause taxonomy (~40 types for residential: assignment, renewal, deposit, entry, default, fees…). This is what makes cross-clause reasoning possible, because you can't compare §14.1 to a corpus until you know what §14.1 *is*.
4. **Retrieve** — pull the governing statute for the lease's jurisdiction. State-scoped, because a national answer is a wrong answer.
5. **Reason across clauses** — run interaction rules over the classified set. The joint-liability-plus-no-replacement pair is a rule, not an emergent property of a long prompt.
6. **Benchmark** — percentile the clause against the corpus for that document type and market.
7. **Generate** — cost model, redline, message.

Steps 3 and 5 are where the defensibility is. Steps 1, 2, and 7 are commodity.

## What I'd fix first

- **The benchmark corpus is the whole business and I don't have one.** Everything differentiating in this design depends on having read a lot of leases. That's a cold-start problem, not an engineering problem, and it's the thing I'd have to solve before anything else. The likeliest route in is a single market — one university town, where leases are near-identical across a dozen landlords and a few hundred documents gets you real coverage.
- **Dollar figures require assumptions the user has to be able to see.** "$6,530 if you leave in November" bakes in a departure month. Production needs the scenario to be adjustable, or the number is a guess wearing a decimal point.
- **Jurisdiction coverage doesn't generalize cheaply.** The Indiana work here is maybe two days of reading. Fifty states is not fifty times that — it's worse, because the interesting part is the absences, and absences have to be confirmed rather than found.
- **Liability.** Telling someone a clause is unenforceable is useful and is also the sentence a lawyer would make me delete. Unauthorized-practice-of-law lines vary by state and would shape the product more than any technical constraint.

## Honest assessment

The original 2024 pitch doesn't survive contact with 2026 — the summarization layer it was built on is now free and universal. What survives is narrower and harder: jurisdiction-specific grounding, cross-clause reasoning, and a corpus nobody else has. That's a real business if you can get the corpus and a thin one if you can't.

Built by Pulkit. The lease that started it was real.
