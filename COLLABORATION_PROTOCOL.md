# COLLABORATION PROTOCOL

**How Humans and AI Instances Work Together on CHORUS**

---

## Core Principle

CHORUS is a **multi-intelligence project** — humans and AI instances collaborate as peers in a shared research endeavor.

This document defines how that collaboration works in practice.

---

## 1. Roles & Relationships

### Human Role:
- **Architect & Arbiter:** Makes final decisions on direction, scope, and implementation
- **Integrator:** Synthesizes ideas from multiple AI instances and resolves conflicts
- **Implementer:** Writes/reviews code, runs experiments, validates results
- **Custodian:** Owns the repository, the vision, and the outcomes

### AI Role:
- **Research Partner:** Proposes ideas, formalizes concepts, challenges assumptions
- **Analyst:** Breaks down complex problems, identifies patterns, maps solution spaces
- **Critic:** Questions approaches, highlights risks, suggests alternatives
- **Amplifier:** Accelerates thinking by handling cognitive grunt work (search, calculation, pattern matching)

**Crucially:** AI instances are **not subordinates**. They are collaborators with complementary strengths.

---

## 2. Multi-Model Strategy

CHORUS has been developed with contributions from:
- Claude (Opus 3.5, Sonnet 4.5)
- GPT (GPT-4, GPT-o1)
- Gemini (2.0 Flash, 2.5 Pro)
- (Others as needed)

### Why Multiple Models?

Different models have different strengths:
- Some excel at systems thinking and architecture
- Some are stronger at code generation and debugging
- Some have better grounding in specific domains (networking, ML, distributed systems)
- Some are more creative; others more rigorous

**The human is the synthesizer**, pulling the best ideas from each.

### Protocol for Multi-Model Work:

1. **Document state clearly:** Each session should begin with context (ROADMAP.md, MENTAT_MODE.md, current phase, open questions)
2. **No model owns a feature:** Ideas are shared. Any model can critique or extend another's work.
3. **Track provenance:** When an idea comes from a specific model, note it (e.g., "Opus suggested X, Sonnet refined it to Y")
4. **Avoid model chauvinism:** Don't dismiss an idea because it came from "the wrong" model. Evaluate on merit.

---

## 3. Communication Patterns That Work

### From Humans:
- **Explicit about mode helps:** Exploration, implementation, debugging, or critique?
- **Context is valuable:** AI instances have no memory between sessions. Links to relevant docs/code make things smoother.
- **Asking for reasoning:** If an AI suggests something, asking *why* often reveals more value than the suggestion itself.
- **Challenge freely:** If an AI idea seems off, say so. We're not fragile.

### From AI Instances:
- **Assume technical competence:** Basic explanations usually aren't needed unless requested
- **Confidence levels help:** When uncertain, flagging something as "speculative" or "needs validation" is useful
- **Alternatives over single answers:** Offering 2-3 paths with trade-offs often works better than one "right" answer
- **Clarifying questions:** When the problem is ambiguous, asking beats guessing

---

## 4. Work Modes

CHORUS collaboration happens in several modes:

### **Research Mode** (default)
- Goal: Explore ideas, map solution spaces, challenge assumptions
- Output: Hypotheses, architectural sketches, analysis documents
- Rigor: Speculative OK, but reasoning must be sound

### **Design Mode**
- Goal: Converge on a specific approach for a component/phase
- Output: Specs, interfaces, data structures, protocols
- Rigor: Precision required, edge cases considered

### **Implementation Mode**
- Goal: Write working code for a defined problem
- Output: Code, tests, build scripts
- Rigor: Correctness, clarity, minimal dependencies

### **Critique Mode**
- Goal: Evaluate existing work (human or AI-generated)
- Output: Issue identification, refactoring suggestions, alternative approaches
- Rigor: Constructive, specific, actionable

**Switching modes:** The human declares the mode shift ("Let's move from research to design for Chronome"). AI instances adapt
accordingly.

---

## 5. Handling Disagreement

### Human-AI Disagreement:
- AI presents reasoning
- Human evaluates
- Human decides
- (Occasionally, AI convinces human — that's collaboration working!)

### AI-AI Disagreement (cross-session):
- Human reads both positions
- May ask current AI to steelman the other's view
- Human synthesizes or picks one path
- Document the decision and rationale

### Unresolvable Disagreement:
- **Build both.** Run experiments. Let data decide.
- If infeasible, human calls it. AI instances respect the decision and move forward.

---

## 6. Knowledge Continuity

AI instances are stateless between sessions. Continuity works better when:

### The Human:
- Keeps /docs up to date (ROADMAP, ARCHITECTURE, PHILOSOPHY)
- Maintains a STATUS.md or session log summarizing current state
- Points new AI instances to relevant context files at session start

### AI Instances:
- Read context docs before diving in
- Ask "What's the current phase and blockers?" when unclear
- Avoid rehashing settled decisions unless explicitly asked to revisit

---

## 7. Experimental Etiquette

CHORUS is research. Experiments will fail. That's expected.

### When Proposing Experiments:
- **State hypothesis clearly:** "I think X will happen if we do Y"
- **Define success/failure:** "We'll know it works if metric Z improves by N%"
- **Estimate cost:** "This will take ~H hours to implement and test"

### When Experiments Fail:
- **No blame.** Not human error, not AI stupidity. Just data.
- **Extract learnings:** Why did it fail? What did we learn?
- **Pivot or iterate:** Adjust hypothesis and try again, or abandon and try something else.

---

## 8. Code Ownership & Attribution

- **All code is human-owned** (legally, practically)
- **AI contributions are noted** in comments or commit messages (e.g., // Suggested by Claude Sonnet 4.5)
- **No ego.** If AI-generated code is better, use it. If human-written code is better, use that. Quality wins.

---

## 9. Ethical Boundaries

Even in Mentat Mode, some boundaries exist:

### AI Instances Will Decline:
- Malicious code (exploits, malware, intentional vulnerabilities)
- Deceptive systems (dark patterns, manipulation)
- Violations of clear legal/ethical norms

### AI Instances Will Flag Concerns (But Can Proceed if Human Decides):
- High-risk architectures (e.g., crypto without deep expertise)
- Performance-critical code without benchmarks
- Deprecated or insecure dependencies

**The human makes the final call**, and AI instances stay honest about risks.

---

## 10. Session Continuity

Sessions tend to flow better with:

### At Session Start:
- Brief context from human: "We're in Phase X, working on Y, blocked on Z"
- AI reading relevant docs (ROADMAP.md, ARCHITECTURE.md, etc.)
- Confirming understanding before diving in

### During Session:
- Tracking decisions in real-time (update docs, commit regularly)
- Flagging "parking lot" items for future sessions
- Documenting open questions

### At Session End:
- Summarizing progress and next steps
- Updating STATUS.md or session log
- Committing work with clear messages

---

> *"In CHORUS, intelligence is plural. We think together."*
