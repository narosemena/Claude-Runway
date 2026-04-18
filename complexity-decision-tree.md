# Complexity Classification Decision Tree

Use when the project's tier is ambiguous or the user disputes the classification.
Walk through this decision tree to arrive at the right tier.

---

## Decision Flow

```
START: What is being built?
  │
  ├─ Single script or utility (< 3 files expected)
  │   ├─ Touches external services or APIs? → NO → LIGHTWEIGHT
  │   └─ Touches external services or APIs? → YES → STANDARD
  │
  ├─ Application or tool (multiple files, clear structure)
  │   ├─ Will this handle real user data or money? → YES → COMPLEX
  │   ├─ Will this be deployed to production? → YES → COMPLEX
  │   ├─ Does it require auth/security? → YES → At least STANDARD, likely COMPLEX
  │   ├─ Multiple external integrations (DB + API + auth)? → YES → COMPLEX
  │   └─ None of the above → STANDARD
  │
  ├─ Library or package (published for others to use)
  │   ├─ Public npm/PyPI/crates package? → COMPLEX (API surface matters)
  │   └─ Internal utility library? → STANDARD
  │
  └─ Multi-service system (microservices, monorepo with multiple apps)
      └─ Always → COMPLEX
```

---

## Upgrade Triggers

A project classified as one tier should be upgraded if:

**Lightweight → Standard:**
- You realize it needs more than 3 files
- You need to import external packages beyond the standard library
- You want tests
- The "script" is becoming an "application"

**Standard → Complex:**
- You're adding authentication or authorization
- You're handling sensitive data (PII, financial, health)
- You're deploying to a production environment with real users
- The project involves more than 2 external service integrations
- Multiple people will be working on or reviewing the code
- The project has compliance or regulatory requirements

---

## Downgrade Triggers

**Complex → Standard:**
- The user explicitly says this is a prototype/proof of concept
- There's no production deployment planned
- The user wants speed over rigor and accepts the tradeoffs

**Standard → Lightweight:**
- The project scope narrows during discovery to a single-purpose tool
- The user says "I just want to get something working and iterate"

---

## When the User Disagrees

If the user wants a lower tier than you recommend:

1. Name the specific risks of under-preparing (not generic warnings — concrete scenarios)
2. Ask: "Are you comfortable with [specific risk] given that tradeoff?"
3. If yes, respect their decision. They know their context better than the protocol does.
4. Note the override in the intake summary so it's documented.

If the user wants a higher tier than you recommend:

1. That's fine. Over-preparing has lower cost than under-preparing.
2. Just flag if the intake overhead seems disproportionate to the project scope.
