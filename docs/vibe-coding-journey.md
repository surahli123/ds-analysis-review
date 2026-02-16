# Building a DS Analysis Review Agent via Vibe Coding in a Two-Day Marathon

**A data scientist's journey from idea to shipped Claude Code plugin — no code, just domain expertise and stubbornness.**

**In two days, I built an AI tool that reviews data science analyses across eight lenses and produces scored reports with specific fixes. I wrote zero traditional code. What carried the project wasn't engineering skill — it was the PM instincts, DS rigor, and communication patterns I'd absorbed over ten years of getting my own work reviewed.**

---

It started with something I don't usually admit out loud.

I've been a data scientist for over a decade. The analysis side — methodology, metrics, experimental design — that's my comfort zone. But every performance review, the same theme comes back: *tighten the narrative, lead with the "so what," calibrate for your audience.* Sound analysis, inconsistent communication. I know I'm not alone. Most data scientists I've worked with carry the same gap. We optimize for rigor; our stakeholders optimize for clarity. Impact gets lost in between.

I'd tried our company's internal AI writing tools. They were static, not tailored to DS work, and couldn't distinguish between "metric without context" (a methodology problem) and "metric without audience-appropriate context" (a framing problem). I wanted something smarter. Something that reviewed like a senior peer — catching both analytical and communication blind spots.

The problem: I'm not a developer. My coding stops at Python scripts and SQL. But I'd been using Claude Code for side projects, and a thought kept nagging me — maybe I didn't need to know how to code. Maybe I just needed to know what "good" looks like. I'd already prototyped an internal version at work using our company's AI coding agent. This public version, built with Claude Code and Opus 4.6, is the one I could share.

So I blocked two days and dove in.

---

**Day 1 was all PM brain.** I didn't touch a single prompt. I just made decisions.

My first instinct was to over-engineer — five levels of graceful degradation, automatic audience detection, session freshness monitoring. Classic scope creep. A PM-style review caught nine issues and I cut it all. *"Not yet"* turned out to be the most productive phrase of the entire project.

What survived: thirteen architecture decisions. Split the review into two dimensions — Analysis (is the methodology sound?) and Communication (will the audience act on this?). Handle short documents differently from long ones. Don't let two reviewers flag the same issue twice. Each decision took five minutes. Undoing the wrong choice in code would have taken hours.

Then came the rubric — and this is where my DS brain took over, for better and worse.

I needed to define what "good" actually means, in scorable terms. The eight evaluation lenses came from frameworks I'd collected across my career: Logic & Traceability from Pedro Sant'Anna's backward-check workflow. Actionability from DoorDash's analytics culture — *any insight that isn't actionable is trivia.* Audience Fit from Matt Abrahams' Stanford communication frameworks, filtered through years of watching my own presentations land differently depending on who was in the room.

The deduction values? Honestly, those started as gut estimates. A single row looked like: `Missing TL;DR -> CRITICAL -> -12 points -> reader can't find the insight in the top 20%.` I wrote thirty of these, fully expecting calibration to prove half of them wrong. It did. But having explicit numbers — even wrong ones — meant I could debug them. That turned out to be the whole point.

For test data, I scraped DS blog posts from Airbnb, Netflix, and Meta using browser automation, and pulled analyses from Kaggle. Six documents total. Was this a rigorous evaluation set? Not remotely. Six public-facing articles, all tech company blogs, no messy internal memos or early-stage drafts. A real evaluation study would need thirty documents across industries with inter-rater reliability checks. I had a weekend and stubbornness. But six diverse documents were enough to test one thing: can the system tell good work from bad?

---

**Day 2 nearly broke me.**

I ran the system on a published Meta article about LLM bug detection — 109 likes, real community engagement. Score: **18/100, Major Rework.** Then on a Vanguard A/B test with pre-specified hypotheses and balance checks. Score: **16/100.** *Lower than the blog post.*

I stared at the screen. The tool actively punished rigor. More detail meant more surface area for the system to find issues, which meant lower scores. A junior analyst submitting three vague paragraphs would outscore a senior who did the work properly. I wanted to scrap the whole thing.

But then my DS instincts kicked in. I recognized the feeling — it's the same frustration when a model you've been tuning performs worse than the baseline on the holdout set. The architecture isn't wrong. The loss function is.

**Root cause:** deduction-only scoring. Start at 100, subtract for every issue. No credit for what the analysis did *well*. This is a problem I understand from search relevance — ranking systems that only penalize bad signals can't distinguish between documents. You need positive signals too. I was building a ranking system for analyses and had forgotten the most basic principle of ranking.

**The fix took three calibration rounds**, and each one taught me something my DS training should have predicted. Round 1: I added strength credits and diminishing returns — scores jumped from the 16-29 range to the 59-73 range. Over-corrected. Classic when you change too many variables at once. Round 2: I changed only the diminishing returns curve. Scores settled into target ranges with meaningful differentiation. Round 3: I extended to all six fixtures — and discovered my fixes had overfit to three documents. The same lesson every ML practitioner learns about train/test splits, playing out in prompt engineering.

None of this is rigorous science. Three calibration rounds on six blog posts wouldn't survive peer review. But it was enough to validate the core question: does the system reward rigor and penalize hand-waving? When I ran it on one of my own draft analyses, it flagged that I'd buried the business impact in section 4 — the exact feedback my manager gives me every quarter. That was the moment I stopped doubting.

---

**What surprised me most was which skills actually mattered.**

Not coding — Claude handled all of that. What mattered was the stuff I'd been doing for years without thinking of it as "technical":

*PM thinking* showed up everywhere. Cutting scope. Saying "not yet" to features. Forcing explicit decisions instead of letting the system guess. When Claude proposed automatic audience inference from document content, I said no — that's a user decision. When three reviewer perspectives disagreed on the credit cap, I made the call based on how DS analyses actually work in practice.

*DS rigor* drove the calibration loop. Change one variable per round. Test against all fixtures. Document what moved and why. When Round 1 over-corrected, I didn't panic — I traced it to compounding credits, isolated the variable, and fixed it in Round 2. It's the same discipline as hyperparameter tuning, applied to prompts instead of models.

*Communication instincts* — the ones I'm supposedly still developing — shaped the rubric itself. The audience personas map to real people I present to weekly. The routing table (which agent owns which finding) came from my experience being on the receiving end of both analytical and communication feedback. I know the difference because I've been dinged for both.

---

**I want to be honest about what this isn't.**

The evaluation set is the weakest part. Six blog posts, all public-facing, calibrated to tech company writing styles. The tool has never seen a two-page internal memo or a dense Confluence page with nested expand macros. It also has inherent run-to-run variability — the same document can score differently by up to ten points across runs. That's a fundamental property of LLM-based evaluation, not a bug I can fix. In practice, treat the score as a band (60s means fix these issues, 80s means ready to share) rather than a precise number. The findings list is more stable than the score.

This approach also only works because the tool is entirely prompt-driven — no databases, no APIs, no state management. And it only works because my domain knowledge was well-formed. Ten years of absorbed feedback gave me enough conviction to write a 276-line rubric. If I'd tried this for a domain I was still learning, the prompts would have been as vague as my understanding.

Vibe coding amplifies expertise. It doesn't substitute for it.

---

**So why ship something imperfect?**

Because this tool exists for a personal reason. I'm tired of getting the same communication feedback and not having a way to catch those patterns before someone else does. I wanted a peer reviewer available at midnight, one that evaluates against a rubric I trust — because I wrote it from the patterns I've absorbed over a decade of hearing *"where's the so what?"*

v1.0 ships today. Next: expand the test corpus to thirty documents across industries, run inter-rater reliability checks against human reviewers, and tighten the credit cap that's inflating scores. The science behind the scoring wouldn't survive a methods section yet. But the tool is useful *now* — and the person who benefits most from it is me.

`claude plugins install surahli123/ds-analysis-review`

---

*Built with Claude Code (Opus 4.6) over two days. Domain expertise: 10+ years. Lines of traditional code written: 0.*

*If you're a domain expert who's been told "you should build a tool for that" — you can. Start by writing down your rubric — the mental checklist you already use when reviewing work. Then use Claude Code to turn those criteria into a scoring system. The domain knowledge is the hard part; the tooling is the easy part.*
