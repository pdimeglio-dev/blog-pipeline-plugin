# Voice samples: how Pablo actually writes

Read this before drafting or revising anything that goes out under Pablo's name
(Writer, Rewriter, Social Manager). The voice rules in dimeglio.dev's AGENTS.md say
what to avoid. These samples show what to aim for. When a sentence you wrote
would sound out of place next to them, rewrite it.

Each sample is real text Pablo wrote or approved after editing a draft. Add new
ones when he rewrites something by hand; that is the strongest signal there is.

---

## Sample 1: LinkedIn post, Building the Batcave Part 2 (2026-10-24)

Written by Pablo from a draft, then lightly copy-edited.

> Replaced MyFitnessPal with my own AI agent (and his own apps).
>
> Meet Gordon, the first agent in the Batcave to get his own apps. The Batcave is a set of AI agents I've been building, and now they're getting apps of their own. Two weeks before a family vacation I started logging all my food with him, and it stuck. Given how active I was going to be before the trip, we set calories and macros that worked with the little time we had. We also cleaned up the pantry so he could suggest meals only from what's there.
>
> He isn't just a chat box. I'd already solved that part with Telegram: everyone in the Batcave talks through it, and a small router picks the right agent to put on the line. Now Gordon has more ways to talk: a website on Vercel, a React Native iOS app with Siri (App Intents) and notifications so he can talk back, and even his own little Apple Watch app for a quick macros check or logging from my wrist.
>
> "Hey Siri, log food in Gordon" from the lock screen. Calorie ring and macros on my Apple Watch. Hold the mic and talk, snap a plate, scan a barcode, and he logs it. Things I always wanted from other tracking apps, but never quite got the way I wanted them.
>
> Next up: Ra's, the training agent, so food and training finally talk to each other.

## Sample 2: blog closing, same post

The draft closing was rejected twice ("does not read like myself"). This is the version Pablo approved, built from how he described it in chat.

> The trip was never the test. The goal was the two weeks before it: lose some weight and get my nutrition in order before getting on the plane.
>
> Being honest, a week at an all-inclusive is not getting tracked on Gordon. I'm not logging buffet plates on vacation. But I know it's going to be super helpful when I'm back, to get on track again right away instead of drifting for a month.
>
> One thing I noticed that I wasn't expecting: my recovery between workouts got faster once I was actually eating enough protein. Which makes the next step pretty obvious. Ra's, the workouts agent, needs some love.

## Sample 3: how he describes his own process (chat, 2026-10-22)

Raw, unedited. Useful for rhythm and word choice, not to copy verbatim.

> I am leaving this weekend, have been tracking on it for 2 weeks. Making changes on the go, fixing bugs, improving features. improving the agent, using evals to control we are not breaking shit.

> say briefly that I have found bugs on the mobile app ux and also some on the backend and agents code or tools, but have been working on fixing them, adding tests and evals everytime something new was found.

## Sample 4: LinkedIn post, Building the Batcave Part 3 (2026-11-04)

Drafted against Sample 1 after he rejected a template-shaped draft, then revised by Pablo and posted. He said the first version of the opener needed more explanation and didn't say where things ended up, then corrected "ready to open up to other users" because there's no signup yet.

> My AI nutrition agent, Gordon, started as a Telegram bot built for exactly one user: me. My user id was a constant in the code, so as far as he knew, nobody else could ever exist. Now he has a web app and an iPhone app, and the code is ready for other users.
>
> It started because I wanted to open Gordon from my phone without going through a chat app. That meant getting my user id out of the code, and a web app and an iPhone app talking to the same agent and the same diary, without breaking the bot.
>
> The cool part is that it's still one Gordon. Telegram and the apps only differ at the front door. Behind it, it's the same queue, the same worker and the same agent. I can send him a message, lock my phone, and his reply shows up on the lock screen when he's done.
>
> Being honest, most of the guardrails in there exist because I tripped over something first, like the eval run that wrote 13 rows into my real diary.
>
> There's no signup yet, so it's still just me using him, but the code doesn't assume that anymore.
>
> Part 3 of Building the Batcave. Next up: memory, where I thought Gordon had four tiers and then read the code.

## Sample 5: introducing the agent, Building the Batcave Part 4 (2026-11-10)

Added after Pablo said the post had to explain the Batcave as a multi-agent system and say more about Ra's before the details. He approved the post ("this reads great").

> That post also said Ra's would come last, "and it'll be honest about how simple he still is." So, honestly: until a couple of weeks ago Ra's was a 16-line prompt on Haiku at low effort, with two tools: one to log a set and one to read history back. They wrote to one flat table: exercise as free text, reps, weight, unit. He could write a row down and read it back, and that was about it.

## Sample 6: explaining the core idea in plain words, same post

Added after he asked "are we explaining how evals work and what they are... and more important what the golden sets are and why we need those".

> A golden set is a list of real inputs, each with the right answer written next to it by someone who knows. For Ra's that's me.
>
> With normal code I can read the function and write a test for it. With a model there's nothing to read, and the prompt tweak that fixes the workout in front of me can quietly break three I wasn't looking at.

---

## What his edits consistently did

Patterns from the Part 2 revision loop. Each one came from a specific edit.

- **Plain over clever.** He cut "made peace with that", "earn its keep", "slightly awkward", "the only place my protein and my training meet is my head". Lines that sound written get replaced with lines that sound said.
- **"We" for him and the agent.** "We set calories and macros", "we cleaned up the pantry". Gordon is a collaborator, not a tool.
- **His own phrases.** "Being honest", "super helpful", "give it some love", "on the go", "the cool part". Use them; don't polish them away.
- **Personal process is the interesting detail.** Planning macros around the time he had, cleaning the pantry so suggestions come from it, faster recovery from more protein. These beat implementation detail for a general reader.
- **Lead with what works.** He asked not to frame posts around issues he had. Bugs are texture, briefly mentioned, unless the post is about the bug.
- **Technical depth in one place.** He cut the SQL migration, the TestFlight story, the eval-drift details and the voice-guard list as "too much detail a blog reader wouldn't care about", then asked for more on search because that part was interesting. One deep section, chosen for interest, beats depth everywhere.
- **Facts about his life stay his.** Family vacation (not "last-minute trip"), no body-weight number, no invented times of day. When the draft guessed, he corrected it.
- **Honest scope.** "The Batcave is a set of AI agents ... now they're getting apps", not "a suite of apps running in production". He corrected framing that overstated what exists.
- **Social copy is a story, not filled-in slots.** The rejected Part 3 draft opened "This post is the plumbing", listed topics in the link line, added a parenthetical aside and a closing question. The version he liked told what he wanted, what it can do now (lock the phone, the reply still arrives) and one honest line.
- **Say where it ended up.** He sent back an opener that stopped at the problem ("needs more explanation"). Openers say what was true then and what's true now, including what isn't there yet.

Patterns from the Part 4 revision loop (Ra's golden sets, 2026-11-10). The full checklist is in dimeglio.dev's AGENTS.md under "Written for a reader who hasn't seen the repo".

- **He gives the spine; build the post on it.** He reframed a trivia title (halt clean vs pause clean) into the process: how the golden sets were built from real data and how their evals drove development. The story is the method, not one example.
- **Set up the world before the detail.** He asked twice for more context: first what golden sets and evals are and why they're needed, then the Batcave as a multi-agent system and what Ra's was and should become. A series post still has to stand alone.
- **Say the design split out loud.** "Code to discover the exercises, the LLM to find the structure" had to be its own section, not one sentence in the parser part.
- **Repo words are not reader words.** "Why we saying things like ADR... or too many details?" Plain names for things, and only the numbers that carry the point.
- **The explainers he asked for stay.** The Rater kept suggesting cuts to them; he kept them. Long is fine when it's earned.
