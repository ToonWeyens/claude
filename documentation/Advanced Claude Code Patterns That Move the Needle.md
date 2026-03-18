# **Advanced Claude Code Patterns That Move the Needle**

*I've spent 2000+ hours building with LLMs this year. These are the patterns that really work. I GUARANTEE that you have not heard of at least one of these tips. Enjoy\!*

---

## **Motivation**

Contrary to popular belief, LLM assisted coding is an unbelievably difficult skill to master due to combining software engineering fundamentals with all of the agentic harnesses and context engineering nuances. It can be incredibly overwhelming. I made my channel with a premonition that learning agentic coding as a discipline to master, not just as a tool to use was a mental reframe that is severely lacking in the software engineering and founder communities. I think Andrej Kaparthy’s tweet on this echoes the sentiment that those in the loop have been feeling lately.

“I've never felt this much behind as a programmer. The profession is being dramatically refactored as the bits contributed by the programmer are increasingly sparse and between. I have a sense that I could be 10X more powerful if I just properly string together what has become available over the last \~year and a failure to claim the boost feels decidedly like skill issue. There's a new programmable layer of abstraction to master (in addition to the usual layers below) involving agents, subagents, their prompts, contexts, memory, modes, permissions, tools, plugins, skills, hooks, MCP, LSP, slash commands, workflows, IDE integrations, and a need to build an all-encompassing mental model for strengths and pitfalls of fundamentally stochastic, fallible, unintelligible and changing entities suddenly intermingled with what used to be good old fashioned engineering. Clearly some powerful alien tool was handed around except it comes with no manual and everyone has to figure out how to hold it and operate it, while the resulting magnitude 9 earthquake is rocking the profession. Roll up your sleeves to not fall behind.” \- Andrej Kaparthy (one of the GOATs of AI research)

## **The Core Philosophy**

Before we begin, I would like you to fully embrace the idea that any issue in LLM generated code is solely due to YOU, the user. With LLMs in the advanced state that they are now, errors that come from AI generated code are very frequently traceable to a user error, either due to improper **prompt engineering** (not clearly and succinctly expressing what you wanted the LLM to build), or improper **context engineering** (not adequately controlling what information does and does not go into the model’s context window. Context rot is a real thing and model performance significantly decreases as more context is shoved into the window.)

---

## **Pattern 1: The Error Logging System**

Since we're treating Agentic Coding as a **skill to learn**, we have an issue. Learning is done best by **tightening and clarifying the loop** between what you input to a system and what you get out. The tighter this loop, the faster you improve. The more convoluted it gets, the harder the skill is to learn. Agentic coding has one of the most convoluted input-output loops of any skill I've encountered. First, **the output is qualitative**. In basketball, either the shot goes in or it doesn't. In agentic coding, it's "do I like this output," "is there a bug," "did the model hallucinate." Not binary. Not clean. Second, the middle is a black box. You put in a prompt, stuff happens, output appears. You can't see the intermediate states. **You can't trace causality.** Third, non-determinism adds noise. You can add role assignment to your prompts as a new tool in your toolkit, and it may have the exact same behavior as if you didn't. There's no verifiable difference unless you're running split tests at scale. People have tried solving this with benchmarks. SWE-bench and the like. Except those don't properly reflect how models perform in the field \- they measure synthetic tasks, not your actual workflow with your actual prompts. So here's my solution: **Log errors in time**. When something goes wrong \- hallucination, bug, anti-pattern \- I capture exactly what my input was (**prompt, context, harness**) and exactly what the output was. I trace the triggering prompt verbatim. I categorize the failure. I ask: what did I do wrong? Over time, **patterns emerge**. This reconstructs the input-output loop that agentic coding normally hides from you. It turns the black box into something you can actually learn from. **It puts the power back in your hands** instead of just reading tips and tricks online about prompt engineering and hoping they work.

### **What Triggers an Error Log**

Any time:

* Claude hallucinates something that doesn't exist  
* Claude does something I didn't like  
* Claude builds something I didn't ask for  
* An anti-pattern occurs  
* A bug appears in something Claude built  
* An instruction gets ignored or misinterpreted  
* Context gets lost  
* Claude gets stuck in a loop

Basically: anything that could be attributed to a misuse of context, prompting, or harnesses.

### **The Workflow**

1. Something goes wrong (hallucination, wrong build, ignored instruction, anti-pattern)  
2. Invoke `/log_error` \- this forks the conversation  
3. Claude interviews me about what happened with specific questions  
4. It captures:  
   * The exact triggering prompt (verbatim \- this is critical)  
   * Failure category (hallucination type, instruction ignored, context lost, etc.)  
   * Root cause analysis (surface cause AND deeper cause)  
   * Prevention strategy  
   * What was added to CLAUDE.md (if anything)  
   * Whether this pattern has been seen before  
   * Impact (time wasted, quality impact, downstream effects)  
5. Gets logged to a queryable database  
6. Double-escape to rewind the main conversation and continue working

### **The Interview Questions**

The /log\_error command has Claude ask me 5-8 clarifying questions that are SPECIFIC to what actually happened. Not generic. Examples:

* "I suggested using localStorage for the token. What made you catch that as a security issue?"  
* "The loop I wrote ran 47 times before you stopped it. What should the exit condition have been?"  
* "I missed that edge case with empty arrays. Was this something in the requirements I should have inferred?"

The log captures:

* **What my prompt was** \- the exact words that led to failure  
* **What went wrong** \- specific, not generic  
* **How to prevent it** \- actionable change

Over time, Claude (or you) can analyze the logs and detect common failure patterns. This allows you to improve at agentic engineering by enabling you to try out tips and see what went wrong, learn by building, and establish strong feedback loops.

### **I Also Log Successes**

When something works unusually well, I typically log this too\! This works especially well when trying out tips from online and noting if they work really well so you can be reminded to integrate them into daily workflows.

---

## **Pattern 2: /Commands as Lightweight Local Apps**

Slash commands are secretly one of the most powerful parts of Claude code. Most people think of /commands as saved prompts. I think of them as a method of utilizing Claude as a Service, which I define as building workflows with the power and complexity of a SaaS, but much quicker to build, much more dynamic and flexible, and utilizes the intelligence and filebase/knowledge base understanding of claude code. 

### **Why /Commands Over Skills (Sometimes)**

* Skills cannot be invoked deterministically. You can tell Claude 'use the X skill' and it might, or it might not. I have had many times where I tell Claude to invoke a skill in my prompt only to have it completely ignored. On the other hand, /commands can be invoked deterministically.  
* Since skills can't be launched deterministically but contain valuable knowledge, I use this pattern:  
1. Skill contains the knowledge/instructions  
2. /Command is the deterministic trigger  
3. /Command tells Claude to use the skill

### **How I Think About Them**

/Commands are:

* Easier to build than full localhosted applications  
* More dynamic than applications (they take arguments, can be altered mid-workflow)  
* Capable of launching and orchestrating the work of parallel sub-agents.  
* Has access to your files, repos, browsers, github, and pretty much anything that you do.  
* The deterministic launcher for any workflow you want (including Claude skill usage)

Think of /commands as CLI tools you're building for yourself. They take arguments, they have specific behaviors, they're testable.

### 

### **How I Use Them**

In order to crystallize why this is important and how to use these, here is an example of one of my most complex commands:

/presentation-to-shorts command: It takes a recorded presentation video and its Remotion codebase, then outputs four polished short-form clips with synchronized graphics, background music, and captions.

![][image1]

  The architecture:

  \- Phase 1: Opus refactors the presentation into reusable components WHILE WhisperX transcribes. Parallel because neither depends on the other.

  \- Phase 2: Opus picks the 4 best 20-60 second moments from the transcript.

  \- Phase 3: Four Opus agents build vertical compositions simultaneously.

  \- Error Gate: Playwright MCP validates all clips load. Fail → retry.

  \- Phase 4: Four Sonnet agents sync animation timing to word-level timestamps.

  \- Phase 5: Sequential GPU rendering, then captions via Watchdog.

  The patterns that make it work:

  \- Model routing: Opus for creative decisions, Sonnet for focused tasks, GPU for compute.

  \- Parallel where independent: Phases 1, 3, and 4 spawn multiple agents in one message.

  \- Sequential where dependent: Rendering runs one at a time (GPU memory is finite).

  \- Deterministic scripts for deterministic work: Transcription, video cutting, audio mixing are Python/FFmpeg—not LLM calls. The model orchestrates when to run them.

  Why not build a localhost app for this? The command already knows my file system. It makes 15+ LLM calls across different models. It adapts at runtime based on content. When I want to change something, I edit a markdown file or tell the model mid-process. For turning my own presentations into content, this is exactly the tool I want.

Check out my video on this for the six levels of slash command usage: [The Six Levels of Claude Code Slash Commands (Most are stuck at Level 1)](https://youtu.be/t9FPfbZ3t2A)

---

## **Pattern 3: Hooks for Deterministic Safety**

Hooks are code that runs before/after Claude takes actions. I use them as guardrails.

### **The Setup**

`dangerously-skip-permissions` \+ hooks that prevent dangerous actions \= flow state without fear (use at your own risk\!\!)

"Dangerously-skip-permissions \+ Claude hooks which prevent Claude from doing things you don't like is a beautiful life hack for not having to sit there and press continue (use at your own risk)"

Example: If Claude is about to run something like `rm -rf`, a hook intercepts it deterministically. No more sitting in the loop on experimental applications waiting to accept a cd statement from Claude.

### **Why This Matters**

**A core belief that I have about working with LLMs while coding is that knowing when to force guardrails and determinicity into your workflows is one of the top skills in building proper agentic harnesses.**

---

## **Pattern 4: Context Hygiene**

Context rot is real. Every irrelevant token degrades performance. Coding models typically get injected thousands of tokens immediately before you even start prompting due to [CLAUDE.md](http://CLAUDE.md) bloat, unruly MCP usage, unused skills, repo reading, and more. Some models begin to degrade performance by up to 50%+ at just 50k tokens in context, which means that our coding agents frequently are not performing at the level that they could before our prompt even starts.

### **CLAUDE.md Discipline**

**Keep it hyper lean.** I prefer to get context into the model just-in-time rather than front-loading everything into [CLAUDE.md](http://CLAUDE.md). My global CLAUDE.md is nearly at defaults. Project-level ones are minimal. Every token in the context must earn its place.

Recently, I was working with Claude and I was reading the [CLAUDE.md](http://CLAUDE.md), And I realized that Claude had added thousands of tokens of repo-specific instructions into my global [CLAUDE.md](http://CLAUDE.md) without me even noticing. This means that every prompt that I gave Claude for at least a couple of weeks was including context about a repo that made no sense to the model and likely deteriorated my model's performance without me even knowing. So here’s an actionable step: 

Check your [CLAUDE.md](http://CLAUDE.md) file right now with the strict mindset that every token in this file must earn its place and truly deserve to be in there. Be ruthless and at very least move to repo-specific [CLAUDE.md](http://CLAUDE.md) files.

Signs your CLAUDE.md is too bloated:

* It has content about multiple unrelated projects  
* There are instructions you don't remember adding  
* You haven't reviewed it in over a week  
* It's longer than \~50 lines

### **Compaction (One of the most important variables of Agentic Coding)**

Here’s an actionable step: 

1. Disable autocompact  
2. Add a context status line (e.g \[Opus 4.5\] 55%) to keep you in the loop  
3. From now on, compaction is done when and how you choose.

Now that the ball is in our court for context management, here are some RAPID FIRE TIPS for context management:

* /clear \+ repo-specific [CLAUDE.md](http://CLAUDE.md) is our best method of compaction for very clear break points. Start fresh sessions more often than you think. Context rot is insidious and comes on slowly. [https://www.youtube.com/watch?v=IMWpVV-VtnI](https://www.youtube.com/watch?v=IMWpVV-VtnI) for more on context rot.  
* Treat Claude code as an orchestrator and have him launch opus subagents for isolated tasks.  
* If you have non-critical changes and Opus’ context is full, treat Sonnet\[1m\] as the break glass in case of emergency to get those done.  
* /compact manually (at the right times) is still useful for when you just want a quick solution to tone down context and you don’t want to restate to the model what you are up to.  
* Create a custom slash command /handoff {NOTES} where NOTES are all of the notes that you want the model to know on what you are doing, what to focus on in compacting the conversation, etc. This is a very strong and quick way of managing context once the model really begins to fill up with too much context. This method has the best results but takes the most time and brainpower.  
* Context trimming via double escape (see below)

### **The Double-Escape Time Travel**

This is the most underutilized feature in Claude Code.

Double-pressing escape lets you jump to any point in the conversation and choose: **restore code and conversation** or **just conversation**. Bar none, my favorite thing to do in Claude code is finding the right time to hit Restore Conversation (not Restore Conversation and Code). Treat this as a crucial method for context trimming. Whenever the conversation goes off of the main topic, we can recenter without hurting good code changes. Here’s the most common example:

**The Bug Fix Pattern:**

1. Claude builds an app  
2. Claude introduces a bug  
3. You work with Claude to fix it \- maybe 5-10 turns of debugging  
4. Bug is fixed, code works  
5. Double-escape → restore **only conversation**, not code  
6. Now in Claude's mind, the bug never existed

Why this matters: You keep the working code, but you don't dilute context with debugging history that's no longer relevant. Claude continues with a clean mental model.

**The Runaway Recovery:**

When Claude starts looping or going off the rails:

1. Double-escape → restore **both code and conversation**  
2. You're back to a known good state  
3. Try a different approach

**Use it aggressively.** Most people treat conversation history as sacred. It's not. It's context, and stale context is harmful context.

You may find this video interesting for exploring context engineering more: [Why context engineering is the most important skill...](https://youtu.be/IMWpVV-VtnI)

---

## **Pattern 5: Subagent Control**

Subagents are incredibly powerful tools when utilized right. One thing that I noticed that most people don’t know is that Claude code consistently spawns **Sonnet** and **Haiku** subagents, even for knowledge tasks. Also, I’m sure that you’ve noticed that the CLI is severely lacking in the ability to tell what spawned agents are doing in real time. Luckily there is a solution for this\! I have a localhosted agent dashboard automatically spawn any time an agent is launched which allows you to monitor all active agents closely and see what they are saying, what their prompts were, and what tools they run. So here are your actionable steps for this section:

1. Add the following to your global [CLAUDE.md](http://CLAUDE.md) file: “Always launch opus subagents unless specified otherwise”  
2. Join my free skool community: [https://www.skool.com/the-agentic-lab-6743/about?ref=6be3bb2df7b744df8202baebef624812](https://www.skool.com/the-agentic-lab-6743/about?ref=6be3bb2df7b744df8202baebef624812) and grab my exact code for monitoring your subagents. Enjoy\!

### **My Subagent Philosophy**

* **Keep subagents simple** \- give them very specific patterns to follow, plus skills to access, in my experience it is better to give them specific isolated work rather than abstract roles.  
* **Parallelize aggressively** \- if tasks have isolated context, run them in parallel  
* **More Subagents is Better (If done right) \-** One rule I learned early on is that more subagents \>\> more tasks per subagent. Break up complicated tasks into isolated parts so that subagents can focus on their roles.  
* **The downside of subagents** \- If utilizing multiple subagents for a workflow, be careful for any hallucinations, because if one subagent relies on the output of another, a single hallucination anywhere in the chain can poison the whole workflow. Define clear boundaries and clear checks (leaning towards deterministic checks or Agent X checks Agent Y’s work).

---

## **Pattern 6: My Lean Tool Stack**

By now you know that context is absolutely sacred, and every token of context must fight for its place in my coding agents. As a result, it should come as no surprise that I very rarely use any MCPs outside of the essential Context7 MCP. My actionable step is for you to go download these two and at least try them out in your workflows.

### **Context7 MCP**

Due to the limitations in the training data of large language models always lagging by a couple of months, it is important to have access to up to date and stable documentation, which is where Context7 MCP comes into play. This MCP literally allows our models to look at the documentation of practically any project or framework out there, meaning it can stay up to date and understand the ins and outs of these frameworks. This is absolutely essential for anyone who codes with LLMs.

### **Dev Browser / Playwright MCP**

For those of you who have not explored the idea of browser automation, you absolutely must get and try out one of these. This allows Claude code to easily control your web browser, look for console errors in UIs for debugging, and allows it to take screenshots so it can utilize its multimodality for improved understanding of designs. The potential in these is endless. I highly suggest [dev-browser](https://github.com/SawyerHood/dev-browser), which is quicker and more context efficient than Playwright MCP.  
---

## **Pattern 7: Prompt Engineering on Steroids**

Prompt engineering is obviously a crucial element to all agentic coding workflows, and I noticed two things very early on in my LLM coding journey:

1. Due to the speed of agentic coding, in many cases, your bottleneck is your typing speed  
2. Many parts of prompt engineering are pretty easy and automatable (XML tags, prompt structuring, role assignment).

So obviously I created the following system:

### **The Reprompter System**

This is how I generate high-quality prompts fast:

1. Press a keybind  
2. Dictate what I want (speaking, not typing)  
3. The system asks me clarifying questions based on my dictation  
4. I answer the questions (still voice)  
5. It generates a thorough prompt with XML tags, role assignment, and utilizes all of the literature on good prompting in order to restructure a messy dictated prompt into something easy for the LLM to follow.

This means I can prompt at high quality, quickly, without the friction of typing out XML structures and remembering prompt engineering best practices every time. Once again, find it for free here: [https://www.skool.com/the-agentic-lab-6743/about?ref=6be3bb2df7b744df8202baebef624812](https://www.skool.com/the-agentic-lab-6743/about?ref=6be3bb2df7b744df8202baebef624812)

**Ask the Model to Ask You Questions**

If you don’t want to commit to a full on reprompter, at the very least as an actionable step:

**Have the models you are working with interview you way way more than you are now. Many times, even the couple of questions they ask in plan mode are insufficient for truly extracting exactly what you want.**

---

## **Quick Reference**

| Situation | Action |
| ----- | ----- |
| Claude does something wrong | /log\_error → fork → interview → capture verbatim prompt → rewind |
| Something worked unusually well | /log\_success → capture what made it click |
| Need reliable workflow execution | /command (deterministic) wrapping skill (knowledge) |
| Want a complex workflow fit to your file system | /command with parallel subagents \+ sequential dependencies |
| Claude asks for too much permission | Hooks \+ dangerously-skip-permissions |
| Context filling up too fast | Disable autocompact, add status line \[Opus 4.5\] 55%, manual compact |
| Bug is fixed but context is polluted | Double-escape → restore conversation only (keep code) |
| Claude is looping/runaway | Double-escape → restore both code and conversation |
| CLAUDE.md feels bloated | Weekly review, repo-specific files, ruthlessly trim, check for Claude additions |
| Need clean handoff between sessions | /handoff {NOTES} custom command |
| Clear breakpoint reached | /clear \+ repo-specific CLAUDE.md |
| Subagents using wrong model | Add "Always launch opus subagents" to global CLAUDE.md |
| Can't see what subagents are doing | Agent monitoring dashboard (localhost) |
| Hallucination poisoning subagent chain | Isolated tasks, deterministic checks, Agent X validates Agent Y |
| Typing prompts is slow | Reprompter: voice → clarifying questions → structured prompt |

### If you got value from this in any manner at all or learned something new, I promise that the value you get out of joining my free skool community and subscribing to my youtube will be much more.

### Youtube: https://www.youtube.com/channel/UCD-gasIQYzXqQ4dr7mGPRfw 

### FREE skool community (master agentic coding): [https://www.skool.com/the-agentic-lab-6743/about?ref=6be3bb2df7b744df8202baebef624812](https://www.skool.com/the-agentic-lab-6743/about?ref=6be3bb2df7b744df8202baebef624812)

### Regardless, thank you so much for taking the time to read this.

---

## **Appendix: The Log Error Command (Full)**

# **Log Error**

Can you make a custom slash command called /log-error that says this: 

You are helping the user log an error/failure that just occurred during agentic coding. The PRIMARY goal is to identify **what the USER did wrong** in their prompting, context management, or harness configuration. This is about building USER skill, not cataloging model failures.

## **Core Philosophy**

Errors in agentic coding are almost always traceable to:

1. **Bad Prompt** \- Ambiguous, missing constraints, too verbose, wrong structure  
2. **Context Rot** \- Didn't /clear, conversation too long, stale context polluting responses  
3. **Bad Harnessing** \- Wrong agent type, didn't pass context to subagents, missing guardrails

The model is the constant. The user's input is the variable. Focus on the variable.

## **Logs Directory**

All logs are stored in: **Create directory for this**

Errors: `/errors/error-XXX.md`

* Metadata (for ID tracking): `/metadata.json`

## **Your Task**

1. **Review the conversation** to identify what went wrong.  
2. **Ask 5-8 pointed questions** focused on USER behavior:  
   * "Your prompt was 4000 words. What were the 3 most important requirements?"  
   * "Did you specify what NOT to do, or only what to do?"  
   * "When did you last /clear? How full was context?"  
   * "Did you verify the subagents received the critical context?"  
   * "Was this reference material or explicit requirements?"  
   * "What constraints were in your head but not in the prompt?"  
3. **Trace the triggering prompt** \- Get the EXACT prompt that led to failure.  
4. **Be critical of the user** \- They asked for this. Don't soften it.  
5. **Log the error** with the template below.

## **Log Template**

markdown  
\# Error \#\[ID\]: \[Short Descriptive Name\]  
**\*\*Date:\*\*** \[Date\]  
**\*\*Project/Context:\*\*** \[What were you working on\]

\#\# What Happened  
\[2-3 sentences \- what went wrong specifically\]

\#\# User Error Category  
**\*\*Primary cause:\*\*** \[Pick ONE\]

\#\#\# Prompt Errors  
\- \[ \] **\*\*Ambiguous instruction\*\*** \- Could be interpreted multiple ways  
\- \[ \] **\*\*Missing constraints\*\*** \- Didn't specify what NOT to do  
\- \[ \] **\*\*Too verbose\*\*** \- Buried key requirements in walls of text  
\- \[ \] **\*\*Reference vs requirements\*\*** \- Gave reference material, expected extracted requirements  
\- \[ \] **\*\*Implicit expectations\*\*** \- Had requirements in head, not in prompt  
\- \[ \] **\*\*No success criteria\*\*** \- Didn't define what "done" looks like  
\- \[ \] **\*\*Wrong abstraction level\*\*** \- Too high-level or too detailed for the task

\#\#\# Context Errors  
\- \[ \] **\*\*Context rot\*\*** \- Conversation too long, should have /cleared  
\- \[ \] **\*\*Stale context\*\*** \- Old information polluting new responses  
\- \[ \] **\*\*Context overflow\*\*** \- Too much info degraded performance  
\- \[ \] **\*\*Missing context\*\*** \- Assumed Claude remembered something it didn't  
\- \[ \] **\*\*Wrong context\*\*** \- Irrelevant information drowning signal

\#\#\# Harness Errors  
\- \[ \] **\*\*Subagent context loss\*\*** \- Critical info didn't reach subagents  
\- \[ \] **\*\*Wrong agent type\*\*** \- Used wrong specialized agent for task  
\- \[ \] **\*\*No guardrails\*\*** \- Didn't constrain agent behavior appropriately  
\- \[ \] **\*\*Parallel when sequential needed\*\*** \- Launched agents that had dependencies  
\- \[ \] **\*\*Sequential when parallel possible\*\*** \- Slow execution due to unnecessary serialization  
\- \[ \] **\*\*Missing validation\*\*** \- No check that agent output was correct  
\- \[ \] **\*\*Trusted without verification\*\*** \- Accepted agent output without review

\#\#\# Meta Errors  
\- \[ \] **\*\*Didn't ask clarifying questions\*\*** \- Could have caught this earlier  
\- \[ \] **\*\*Rushed to implementation\*\*** \- Skipped planning/verification  
\- \[ \] **\*\*Assumed competence\*\*** \- Expected Claude to infer too much

\#\# The Triggering Prompt  
\`\`\`  
\[Exact prompt \- verbatim\]  
\`\`\`

\#\# What Was Wrong With This Prompt  
\[Be specific and critical. What should have been different?\]

\#\# What The User Should Have Said Instead  
\`\`\`  
\[Rewritten prompt that would have prevented this error\]  
\`\`\`

\#\# The Gap  
\- **\*\*What user expected:\*\*** \[Expected outcome\]  
\- **\*\*What user got:\*\*** \[Actual outcome\]  
\- **\*\*Why the gap exists:\*\*** \[Direct connection to user error above\]

\#\# Impact  
\- **\*\*Time wasted:\*\*** \[X minutes\]  
\- **\*\*Rework required:\*\*** \[What needs to be redone\]

\#\# Prevention \- User Action Items  
1\. \[Specific action user should take next time\]  
2\. \[Another specific action\]  
3\. \[Consider adding to personal CLAUDE.md or workflow\]

\#\# Pattern Check  
\- **\*\*Seen this before?\*\*** \[Yes/No \- if yes, this is a habit to break\]  
\- **\*\*Predictable?\*\*** \[Should user have anticipated this?\]

\#\# One-Line Lesson (for the USER)  
\[Actionable takeaway about prompting/context/harnessing \- NOT about model behavior\]

\---

*\*Logged on \[timestamp\]\**

## **Important**

* Be CRITICAL of the user \- they're logging this to learn, not to feel good  
* Focus 80% on user error, 20% on model behavior  
* The goal is to improve USER skill at agentic coding  
* If the user can't identify their mistake, help them find it  
* Sanitized logs are useless \- be specific and honest

---

## **Appendix: The Log Success Command (Full)**

\# Log Success

You are helping the user log a success/win that occurred during agentic coding. Most people skip this \- they only log failures. But understanding WHY things work is just as important as why they fail. Capture what went RIGHT.

\#\# Logs Directory  
All logs are stored in: \`/home/\[user\]/claude\_accessible/agentic\_practice\_logs/\`  
\- Successes: \`/successes/success-XXX.md\`  
\- Metadata (for ID tracking): \`/metadata.json\`

\#\# Your Task

1\. \*\*First, review the recent conversation context\*\* to understand what went notably well. Look for:  
   \- What task was accomplished smoothly  
   \- What approach was used that worked well  
   \- Any moments where something just clicked  
   \- Unusually fast completion  
   \- First-try successes  
   \- Elegant solutions  
   \- Minimal intervention needed  
   \- Good tool/command usage

2\. \*\*Ask 4-6 clarifying questions\*\* to extract WHY it worked. Be SPECIFIC to what actually happened. Examples:  
   \- "That auth flow came together in under 20 minutes. What about the prompt setup made it work so smoothly?"  
   \- "You didn't have to correct me once during the refactor. Was that luck or did the context in CLAUDE.md help?"  
   \- "The solution I suggested was cleaner than what you initially had in mind. What made it click?"

   Questions should cover:  
   \- \*\*What specifically went well:\*\* Not generic "it worked" but precise win  
   \- \*\*Why it worked:\*\* The contributing factors  
   \- \*\*The setup:\*\* What context/prompt/approach was used  
   \- \*\*Key ingredient:\*\* What was the one thing that made the difference?  
   \- \*\*Reproducibility:\*\* Could you do this again? Should this become standard practice?

3\. \*\*Trace the triggering prompt\*\*: After the interview, identify and quote the EXACT user prompt(s) that led to this success. This is critical for understanding what instruction produced the win. Ask the user to confirm or paste the exact prompt if you can't find it in context.

4\. \*\*After getting answers\*\*, read metadata.json to get the next success ID, then create the log file.

5\. \*\*Update metadata.json\*\* with the incremented counter.

[image1]: <data:image/png;base64,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>