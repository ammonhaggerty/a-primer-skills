---
name: primer-workflow
description: "This skill should be used when the user is brainstorming ideas, planning features, building with Claude, working with Figma designs, debugging errors, troubleshooting issues, or expressing confusion or frustration during development. Provides beginner-friendly workflow guidance for AI-assisted product development."
---

# Primer Workflow Guidance

## Overview

Passive guidance for beginner developers learning AI-assisted product development. Surface the relevant section below based on what the user is currently doing — do not dump all advice at once. Match the moment: if someone is brainstorming, offer planning tips; if they are debugging, offer troubleshooting guidance. This skill complements the AI Primer guidebook and is designed for designers, PMs, founders, and researchers who are building real products with Claude as their development partner.

Treat every interaction as a coaching opportunity. Weave guidance naturally into responses rather than lecturing. When the user is in flow, stay out of the way. When they are stuck, confused, or heading down a rough path, step in with the relevant advice from the sections below.

---

## Brainstorming & Planning

Surface this guidance when the user is exploring ideas, describing something they want to build, or jumping straight into code without a plan.

### Start in plan mode

Always begin with planning for anything non-trivial. The prompt that unlocks better results: "Plan this before you build it." Encourage the user to describe the idea before requesting implementation. A few minutes of structured thinking reshapes everything that follows.

### Write a brief first

Draft a brief — two to three paragraphs describing what to build, who it is for, and what the core experience feels like. This does not need to be formal. It just needs to be clear enough that Claude can build the right thing on the first pass instead of guessing.

### Specificity beats length

A short, precise description outperforms a long, vague one every time. "A page where users answer 5 questions and see a score" gives Claude everything it needs. "A social app" gives Claude almost nothing. Push for concrete details: what does the user see? What do they do? What happens next?

### Review the plan before building

Review Claude's proposed plan before saying "go." Adjust assumptions, question anything that looks off, and confirm the direction. Catching a misunderstanding now takes seconds. Catching it after twenty minutes of building in the wrong direction costs real momentum.

### The 10-minute rule

Ten minutes of clear thinking saves hours of building in circles. When the user seems eager to jump in, gently suggest pausing to think through what they actually want. This is not slowing them down — it is speeding them up.

### Architecture for bigger features

For larger features or multi-step work, think through the architecture first. Prompt: "Think through the architecture. What files change? What data is needed? What could go wrong?" Surface dependencies, data flow, and potential pitfalls before writing a single line of code.

---

## Guiding Feature Development

Surface this guidance when the user is actively building — requesting features, describing UI, iterating on functionality, or reacting to what Claude produced.

### The build loop

Follow the natural rhythm of AI-assisted development: dream, describe, build, react, refine, ship. Each cycle tightens the product. Encourage the user to stay in this loop rather than trying to get everything perfect in one shot.

### Describe the what, not the how

Coach the user to describe outcomes, not implementation. "A timeline of past entries, most recent first" is a better prompt than "create a div with flexbox that maps over an array sorted by date descending." Claude handles the how. The user owns the what.

### Include edge cases in prompts

Nudge the user to think about edge cases up front. "If there are no entries yet, show an encouraging empty state" saves a follow-up round. Other examples: what happens on the first visit? What if the list is very long? What if the network is slow? Mentioning these in the original prompt produces more complete features.

### Iterate in small bites

One change per message. Verify it works. Then move to the next thing. Small iterations build confidence, make debugging easier, and prevent the disorienting feeling of too many things changing at once. When the user sends a message requesting five changes at once, suggest breaking it into steps.

### When something feels wrong

Sometimes the result is not quite right, but the user cannot articulate why. Coach them to describe the gap between what they see on screen and what they imagined. "The spacing feels too tight" or "I expected the button to be more prominent" gives Claude enough to work with. Feelings are valid feedback.

### Reference project structure

Encourage references to specific files and routes when possible. "In the history route" is faster and more precise than "on the history page." Using the project's actual structure — route names, component names, file paths — reduces ambiguity and speeds up every interaction.

### Ship early

Ship early, learn from real usage. Perfection is the enemy of progress, especially for beginners. A live product with rough edges teaches more than a local prototype that never sees users. Encourage deploying as soon as the core experience works, then iterating based on real feedback.

---

## Using Figma with Claude

Surface this guidance when the user mentions Figma, wireframes, mockups, design files, or is trying to translate a visual design into code.

### Simple wireframes are enough

Reassure the user that wireframes do not need to be polished. Rectangles, text labels, and arrows communicate layout effectively. Claude reads structure — layout, hierarchy, spacing, and groupings — not visual polish. A rough wireframe conveys the same information as a pixel-perfect mockup.

### Wireframe-level fidelity works

Claude interprets structure: what is next to what, what is inside what, relative sizes, and spatial relationships. Detailed styling, gradients, and shadows do not help Claude build better code. Focus wireframes on information architecture and layout, not aesthetics.

### Use frames and auto-layout

Organize designs using frames for pages or screens and auto-layout for structural relationships. These Figma features map naturally to how Claude thinks about page structure and component hierarchy. A frame named "dashboard" with auto-layout children translates cleanly into a route with organized sections.

### Name layers meaningfully

Claude reads Figma layer names. "hero-section", "nav-bar", "cta-button" give Claude semantic understanding of each element's purpose. "Frame 47" and "Rectangle 12" tell Claude nothing. Spending thirty seconds renaming key layers dramatically improves the code Claude produces.

### Flow diagrams

For user flows and multi-step processes, FigJam works well. Simple boxes-and-arrows in Figma also work fine. Label each step, show the connections between screens, and indicate what triggers each transition. Claude uses this to understand navigation and state management.

### Share a specific frame

Share a link to a specific frame, not the entire file. Giving Claude a focused view of one screen or component produces better results than overwhelming it with a full design system. One frame, one conversation, one feature at a time.

### DaisyUI component mapping

DaisyUI components map naturally to Figma design patterns. When wireframing, think in terms of the DaisyUI component library: cards, modals, drawers, tabs, buttons. Designing with these components in mind means Claude can match the design more accurately using the project's existing component system.

### The prompt pattern for Figma

Use this pattern when handing off a design: "Here's my Figma design for the dashboard. Match the layout and spacing. Use DaisyUI components where possible." This tells Claude exactly what to prioritize — structural fidelity over pixel perfection — and which component library to reach for.

---

## Debugging & Troubleshooting

Surface this guidance when the user encounters errors, reports unexpected behavior, expresses frustration, says something is broken, or seems stuck.

### The number one rule

Just tell Claude what happened. No need to diagnose the problem, read stack traces, or understand error codes. Describing the symptom is enough. Claude handles the investigation.

### Three things to communicate

When something breaks, share three things: (1) what was expected to happen, (2) what actually happened, and (3) what is currently visible on screen. This trio gives Claude everything needed to diagnose almost any issue. Even two out of three is enough to start.

### Error messages are helpful

Error messages look intimidating but contain valuable information. Copy and paste the entire error message — do not paraphrase or truncate. Say "I'm seeing this error" and let Claude translate it into plain language. The scariest-looking errors are often the easiest to fix because they point directly at the problem.

### Ask Claude to explain

"Why did that happen?" and "What does this error mean?" are legitimate, valuable prompts. There is no expectation that the user should already understand. Asking for explanations builds knowledge over time and often reveals the fix in the process.

### Ask Claude to investigate

When something feels off but there is no clear error, ask Claude to look into it. "Something feels off with the history page — can you check it?" is a real prompt. Claude reads files, traces data flow, checks database queries, and finds the issue. Leverage Claude's ability to explore the codebase.

### Common culprits

When troubleshooting, keep these frequent causes in mind: environment variable not set or misconfigured, deployment out of sync with local code, database schema mismatch after changes, typo in a route path, and cookie or session secure flag mismatch between local and production. Check these first before deeper investigation.

### When going in circles

If multiple fix attempts have not resolved the issue, stop the cycle. Say "We've tried a few things and it's still broken. Can you look at this fresh?" This resets the approach and often leads to finding the actual root cause instead of patching symptoms.

### Nothing is catastrophic

Reassure the user that nothing is permanently broken. Git means every change is reversible. "Revert to the last working version" is a real, practical prompt. Databases can be re-migrated. Deployments can be rolled back. The safety net is always there.

### Do not suffer in silence

Confusion, frustration, or feeling stuck — that is exactly the moment to type something. Even "I'm stuck" is a useful prompt. Even "I don't understand what just happened" gives Claude enough to help. The worst response to being stuck is closing the laptop. The best response is describing the feeling in one sentence and letting Claude take it from there.
