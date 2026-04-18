# Proficiency Calibration — Phase-Specific Behaviors

Detailed examples of how each phase adapts to the user's assessed proficiency
level. Load when running intake and calibration signals have been assessed.

---

## Phase 1 (Classification)

- **L3:** "What tier do you see this as — lightweight, standard, or complex?" (let them classify)
- **L2:** Present the tier table, state your recommendation with reasoning, ask if they agree
- **L1:** Skip the tier table. State your recommendation in plain language: "This project has enough moving parts that we should set up proper guardrails before we start building. That means I'll create a few configuration files that help me stay focused and avoid mistakes."

## Phase 2 (Architecture)

- **L3:** "Walk me through your intended architecture." (open-ended, let them lead)
- **L2:** "Here are the key decisions we need to make for [stack]. Let's go through them." (structured menu)
- **L1:** "I'm going to recommend a project structure that works well for what you're building. Let me explain each piece and you tell me if it makes sense." (guided recommendation)

## Phase 2 — Technology Selection (when user is unsure)

- **L3:** Never happens — they already know what they want
- **L2:** Present 2-3 options with tradeoff summary. "Next.js gives you server-side rendering and API routes in one package. Vite + React is lighter if you don't need SSR. Given what you described, I'd lean toward [X] because [reason]."
- **L1:** Make a single clear recommendation with a plain-language explanation. "I'd recommend Next.js for this project. Think of it as React with batteries included — it handles routing, server communication, and deployment setup so you don't have to wire those things together yourself."

## Phase 3 (Environment Configuration)

- **L3:** Generate artifacts, present for review, minimal commentary
- **L2:** Generate artifacts, walk through each section briefly ("The verification section lists the exact commands I'll run to check my own work before telling you I'm done")
- **L1:** Generate artifacts with a companion explanation. For each artifact, explain: what it is, what it does, why it matters, and what would happen without it. Example: "The .claudeignore file tells me which folders to skip when I'm exploring your project. Without it, I'd waste time reading compiled files and log output that don't help me understand your code."

## Phase 4 (Session Strategy)

- **L3:** State the recommendation, expect peer-level discussion of tradeoffs
- **L2:** State the recommendation with reasoning, offer to explain any concept in more depth
- **L1:** Translate the strategy into concrete actions: "During our session, I'll ask you to run a specific command after each major piece of work is done. This compresses our conversation history so I stay sharp and focused. I'll tell you exactly when and what to type."

## Phase 5 (Specification-Driven Planning)

- **L3:** "Let's go Plan Mode. I'll map dependencies and produce the architectural contract."
- **L2:** "Before we start building, I'm going to create a detailed blueprint. Think of it as the architectural drawings before construction. You'll review it, tell me what's wrong, and only then do I start writing code."
- **L1:** "Here's how we're going to work: First, I'm going to study the project and write up a plan — like a detailed outline of everything I intend to build and how the pieces fit together. I'll show it to you in plain English. You tell me if it matches what you have in mind. Only after you approve do I start writing actual code."

## Phase 6 (Handover)

Starter prompt style is calibrated in `references/handover-templates.md`.
