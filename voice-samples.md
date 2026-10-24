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
