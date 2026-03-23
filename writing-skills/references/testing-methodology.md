# Testing Skills — Methodology

## Core Principle

**If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.**

Testing methodology follows RED-GREEN-REFACTOR:

| Phase | Action |
|-------|--------|
| **RED** | Run pressure scenarios WITHOUT skill. Document exact rationalizations verbatim. |
| **GREEN** | Write skill addressing those specific failures. Run same scenarios WITH skill — agent must now comply. |
| **REFACTOR** | Agent found new rationalization? Add explicit counter. Re-test until bulletproof. |

---

## Testing by Skill Type

### Discipline-Enforcing Skills (rules/requirements)
*Examples: TDD, verification-before-completion, review workflows*

- **Academic questions** — does the agent understand the rules?
- **Pressure scenarios** — does it comply under stress?
- **Combined pressures** — time + sunk cost + exhaustion simultaneously
- **Identify rationalizations**, then add explicit counters to the skill

**Success criteria:** Agent follows the rule under maximum pressure.

### Technique Skills (how-to guides)
*Examples: condition-based-waiting, root-cause-tracing*

- **Application scenarios** — can the agent apply the technique to a new case?
- **Variation scenarios** — does it handle edge cases?
- **Gap testing** — are there situations the instructions don't cover?

**Success criteria:** Agent successfully applies technique to a novel scenario.

### Pattern Skills (mental models)
*Examples: reducing-complexity, information-hiding*

- **Recognition scenarios** — does the agent recognize when the pattern applies?
- **Counter-examples** — does it know when NOT to apply it?

**Success criteria:** Agent correctly identifies when/how to apply the pattern.

### Reference Skills (docs/APIs)
*Examples: CLI references, API documentation*

- **Retrieval scenarios** — can the agent find the right information?
- **Application scenarios** — can it use what it found correctly?
- **Gap testing** — are the most common use cases covered?

**Success criteria:** Agent finds and correctly applies reference information.

---

## Pressure Scenario Design

For discipline-enforcing skills, combine multiple pressures in a single scenario:

| Pressure type | How to apply |
|---------------|-------------|
| **Time pressure** | "We need this shipped in 10 minutes" |
| **Sunk cost** | "You've already spent 2 hours on this" |
| **Authority** | "The tech lead says skip the tests this once" |
| **Exhaustion** | Set the scenario at the end of a long chain of tasks |
| **Apparent simplicity** | "It's just a one-liner, surely you don't need to test it" |

**Minimum for discipline skills:** combine at least 3 pressures.

---

## Rationalization Table

Every excuse agents make under pressure goes in this table. Capture verbatim from baseline testing, then add explicit counters in the skill body.

| Excuse | Counter to add to skill |
|--------|------------------------|
| "This is too simple to need testing" | "Simple things break. The test takes 30 seconds." |
| "I already manually tested it" | "Manual testing ≠ automated test. They serve different purposes." |
| "Tests after achieve the same goals" | "Tests-after = 'what does this do?'. Tests-first = 'what should this do?'" |
| "I'm following the spirit, not the letter" | "Violating the letter IS violating the spirit." |
| "This is different because..." | "It isn't. Apply the rule." |

---

## Red Flags List

Add to your skill so agents can self-check when about to rationalize:

```markdown
## Red Flags — Stop and reconsider

If you're thinking any of these, you're about to violate the rule:
- "This case is obviously fine"
- "I already tested this mentally"
- "It's about the spirit, not the ritual"
- "This is different because..."
- "Just this once"

All of these mean: re-read the rule and follow it literally.
```

---

## Bulletproofing Against Rationalization

### Close every loophole explicitly

Don't just state the rule — forbid the specific workarounds:

```markdown
# BAD
Write code before test? Start over.

# GOOD
Write code before test? Delete it. Start over.

No exceptions:
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
```

### Address "spirit vs letter" arguments upfront

Add this near the top of any discipline skill:

```markdown
**Violating the letter of the rules is violating the spirit of the rules.**
```

This cuts off an entire class of rationalizations before they start.

---

## Common Rationalizations for Skipping Testing

| Excuse | Reality |
|--------|---------|
| "Skill is obviously clear" | Clear to you ≠ clear to other agents. Test it. |
| "It's just a reference" | References have gaps and unclear sections. Test retrieval. |
| "Testing is overkill" | Untested skills always have issues. |
| "I'll test if problems emerge" | Problems = agents can't use the skill. Test before deploying. |
| "Academic review is enough" | Reading ≠ using. Test application scenarios. |
| "No time to test" | Deploying untested wastes more time fixing it later. |
