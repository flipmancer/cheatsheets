# {{TITLE}} Cheatsheet

> Replace all placeholders like {{TITLE}}, {{DOMAIN}}, {{SHORT_DESC}}, {{API_BASE_URL}}.
> Remove sections not relevant to your domain.
> Keep numbering continuous after removing sections.
> Add more sections and examples as needed.

## Overview

- Purpose: {{SHORT_DESC}}
- Audience: {{AUDIENCE_LEVEL}} (e.g. Beginner / Intermediate / Advanced)
- Scope: {{SCOPE_NOTE}} (what’s included vs excluded)
- Updated: {{DATE}}

## How to Generate a New Cheatsheet

1. Copy this template and rename it:

```powershell
Copy-Item TEMPLATE.md {{TITLE}}.md
```

2. Replace placeholders in the new file:

- {{TITLE}} → e.g., "React", "HTML & CSS", "Node.js", "Hardhat"
- {{DOMAIN}} → e.g., "frontend", "smart contracts", "testing"
- {{SHORT_DESC}} → one sentence elevator pitch

3. Keep it practical:

- Prefer small examples over long theory
- Include "When to use" and "Mental model" for complex topics
- Add language tags to all code fences: shell, js, ts, jsx, py, rs, sol, json, txt, etc.
- Keep headings consistent; use H2 for sections, H3 for subsections

4. Validate quickly:

- Skim for missing placeholders {{LIKE_THIS}}
- Run links and commands mentally; keep them copyable
- Ensure examples compile (where relevant)

---

## {{DOMAIN}} Setup (if applicable)

### Installation

```shell
# Install dependencies
{{INSTALL_COMMAND}}   # e.g., npm install, pip install -r requirements.txt, cargo build
```
...

---

## Core {{DOMAIN}} Concepts

### 1. {{CONCEPT_NAME}}

Short explanation:

- What it is: {{ONE_LINER}}
- Why it matters: {{IMPACT_OR_BENEFIT}}
- Mental model: {{ANALOGY}}

```{{LANGUAGE}}
// Minimal example (replace with your domain code and add comments)
{{CODE_EXAMPLE}}
```

Common pitfalls:

- {{PITFALL_1}}
- {{PITFALL_2}}

---

### 2. {{CONCEPT_NAME}}

Short explanation:

- What it is: {{ONE_LINER}}
- When to use: {{TYPICAL_TRIGGERS}}
- Gotchas: {{EDGE_CASES}}

```{{LANGUAGE}}
// Example in your target language
{{CODE_EXAMPLE}}
```

...

---

## Common Patterns for {{DOMAIN}}

### {{PATTERN_NAME_1}}

```{{LANGUAGE}}
{{PATTERN_1_EXAMPLE}}
```

### {{PATTERN_NAME_2}}

```{{LANGUAGE}}
{{PATTERN_2_EXAMPLE}}
```

...

---

## Quick Reference

List key syntax rules, gotchas, or conventions for {{DOMAIN}}:

1. {{RULE_1}}
2. {{RULE_2}}
3. ...

---

## File Structure Best Practice (example if applicable)

```txt
{{PROJECT_ROOT}}/
├─ {{FOLDER_1}}/
├─ {{FOLDER_2}}/
├─ {{FOLDER_3}}/
├─ {{MAIN_FILE}}
└─ {{CONFIG_FILE}}
```

Notes:

- Keep modules/files small and focused
- Co-locate related files (tests next to code, or under dedicated test directory)
- Follow conventions for your language/framework

---

## Performance Tips

1. {{PERFORMANCE_TIP_1}} (e.g., memoize expensive computations, cache results)
2. {{PERFORMANCE_TIP_2}} (e.g., optimize data structures, use lazy loading)
3. {{PERFORMANCE_TIP_3}} (e.g., profile and benchmark critical paths)
4. {{PERFORMANCE_TIP_4}} (e.g., reduce allocations, use efficient algorithms)
5. ...

---

## Security & Safety (if applicable)

- {{SECURITY_TIP_1}} (e.g., validate inputs, sanitize data)
- {{SECURITY_TIP_2}} (e.g., use secure protocols, encrypt sensitive data)
- ...

---

## Complex Topic Mini-Explanations (Pattern)

Use this pattern whenever a topic may confuse beginners.

### {{COMPLEX_TOPIC}}

- What it is: {{ONE_TO_TWO_SENTENCES}}
- Why it matters: {{IMPACT}}
- Mental model: {{ANALOGY}}
- When to use: {{TRIGGERS}}
- Pitfalls: {{TOP_PITFALLS}}

```{{LANGUAGE}}
// Minimal example or pseudocode illustrating the core idea
{{CODE_EXAMPLE}}
```

---

## Next Steps (example)

- {{NEXT_STEP_1}} (e.g., Explore advanced topics like X, Y, Z)
- {{NEXT_STEP_2}} (e.g., Build a small project using what you learned)
- {{NEXT_STEP_3}} (e.g., Join community forums or groups for support)

---

## Resources

- Official docs: {{LINK}}
- API reference: {{LINK}}
- Tutorials and guides: {{LINK}}
- Useful tools: {{LINK}}

---

## Authoring Checklist (Before Commit)

- [ ] All placeholders ({{...}}) replaced with actual content
- [ ] Code fences have language tags
- [ ] Sections relevant to the domain kept; others removed
- [ ] One short explanation added for each complex topic
- [ ] Commands tested or sanity-checked
- [ ] Examples compile/run in target environment
