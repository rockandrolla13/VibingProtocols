# Refactor Skill Setup Guide

## What You're Installing

A Claude skill that helps you systematically reduce codebase size through verified, iterative deletions.

```
refactor-skill/
├── SKILL.md              # Overview — Claude reads this first
├── plan.md               # Phase 1: Generate the refactor plan
├── execute.md            # Phase 2: Execute each iteration
└── templates/
    └── progress.md       # Template for tracking progress
```

---

## Step-by-Step Setup

### Step 1: Locate Your Skills Directory

Claude looks for skills in `/mnt/skills/user/`. You need to copy the skill there.

**In Claude Desktop/Web:**
Your skills directory is managed by Claude. Ask Claude:
> "Where is my user skills directory?"

**In Claude Code / API:**
Skills typically live in a mounted volume. Check your configuration.

---

### Step 2: Create the Skill Folder

Ask Claude to create the skill for you:

```
Please create a new skill called "refactor" in my user skills directory 
with these files:
- SKILL.md (overview)
- plan.md (plan generator)  
- execute.md (iteration executor)
- templates/progress.md (progress template)
```

Or manually create the folder structure:

```bash
mkdir -p /mnt/skills/user/refactor/templates
```

---

### Step 3: Copy the Skill Files

Copy each file to the skill directory:

| Source | Destination |
|--------|-------------|
| `SKILL.md` | `/mnt/skills/user/refactor/SKILL.md` |
| `plan.md` | `/mnt/skills/user/refactor/plan.md` |
| `execute.md` | `/mnt/skills/user/refactor/execute.md` |
| `templates/progress.md` | `/mnt/skills/user/refactor/templates/progress.md` |

---

### Step 4: Verify Installation

Ask Claude:
> "List the skills in my user skills directory"

You should see `refactor` listed.

Then verify Claude can read it:
> "Read the SKILL.md file in my refactor skill"

---

## How to Use the Skill

### Usage 1: Generate a Refactor Plan

```
Using the refactor skill, create a plan to reduce my codebase.

Parameters:
- Language: Python
- Current LOC: 50,000
- Target LOC: 10,000
- Test command: pytest -q
- Lint command: ruff check .
- LOC command: cloc . --quiet
- Backtest command: python -m backtest.run --config base.yaml
- Untouchable paths: carry_strategy/, core/engine.py
- Progress log: docs/reduce/progress.md

Here's my repo structure:
[paste tree output or describe structure]
```

Claude will output:
1. Baseline assessment
2. Strategy overview (8-12 bullets)
3. Iteration plan (8-20 iterations)
4. Initial progress.md content
5. Risk register

---

### Usage 2: Execute an Iteration

After you have a plan:

```
Using the refactor skill, execute iteration 1.

The plan is:
[paste the relevant iteration from the plan]
```

Claude will:
1. Run pre-flight checks
2. Make the specified changes
3. Run all verification commands
4. Check all gates
5. Update progress.md
6. Report results

---

### Usage 3: Continue After Failure

If an iteration fails:

```
Iteration 3 failed because [reason].

Options:
1. Adjust the iteration scope
2. Split into smaller iterations
3. Address the underlying issue first

Which approach should we take?
```

---

## Quick Reference Commands

| Task | Prompt |
|------|--------|
| Generate plan | "Using refactor skill, create a plan for [params]" |
| Execute iteration | "Using refactor skill, execute iteration N" |
| Check progress | "Show me the current progress.md" |
| Adjust plan | "Iteration N needs adjustment because [reason]" |
| Skip iteration | "Mark iteration N as skipped, reason: [reason]" |

---

## Customization

### Adding Language-Specific Commands

Edit `plan.md` and `execute.md` to include your tooling:

```yaml
# For TypeScript
TEST_CMD: npm test
LINT_CMD: npm run lint
TYPE_CMD: npx tsc --noEmit
LOC_CMD: cloc src/ --quiet

# For Rust
TEST_CMD: cargo test
LINT_CMD: cargo clippy
LOC_CMD: tokei .

# For Go
TEST_CMD: go test ./...
LINT_CMD: golangci-lint run
LOC_CMD: cloc . --quiet
```

### Adding Custom Gates

Edit the "Verification Gates" section in `execute.md`:

```markdown
#### 3.5 Custom Gates

- [ ] **Performance gate**: Benchmark shows no regression >5%
- [ ] **Security gate**: No new vulnerabilities introduced
- [ ] **API gate**: Public API unchanged (or documented)
```

---

## Troubleshooting

### "Claude doesn't see my skill"

1. Verify the path: `/mnt/skills/user/refactor/SKILL.md`
2. Check file permissions
3. Restart the Claude session

### "Plan is too vague"

Provide more context:
- Paste actual repo structure (`tree -L 2`)
- Share example file contents
- Describe what the codebase does

### "Iteration keeps failing"

1. Check if the iteration scope is too large
2. Verify untouchable boundaries aren't being violated
3. Run tests manually to see actual errors
4. Split the iteration into smaller steps

---

## Example Session

```
User: Using the refactor skill, create a plan to reduce my Python 
      trading system from 48k to 12k LOC. Tests: pytest. 
      Backtest: python run_backtest.py. Don't touch strategies/.

Claude: [Reads SKILL.md, then plan.md]
        
        ## Baseline Assessment
        | Metric | Value |
        | Starting LOC | 48,000 |
        | Target LOC | 12,000 |
        | Reduction | 36,000 LOC (75%) |
        ...
        
        ## Iteration 1: Remove Legacy Data Loaders
        **Intent**: Delete unused CSV and XML data loaders
        **Files**: data/loaders/csv_legacy.py, data/loaders/xml_loader.py
        ...

User: Execute iteration 1.

Claude: [Reads execute.md]
        
        ## Pre-Flight Checks
        ✅ Git status clean
        ✅ Tests passing (142 passed)
        ✅ Backtest passing
        ✅ Created branch: refactor/iteration-1
        
        ## Executing Transformation
        Deleting data/loaders/csv_legacy.py...
        Deleting data/loaders/xml_loader.py...
        Removing imports from data/loaders/__init__.py...
        
        ## Verification
        ✅ Tests: 140 passed (2 deleted with code)
        ✅ Backtest: PASSED
        ✅ LOC: 48,000 → 47,200 (-800)
        
        ## Iteration 1 Complete ✅
        Proceeding to iteration 2...
```

---

## Files Reference

After setup, your skill directory should look like:

```
/mnt/skills/user/refactor/
├── SKILL.md          (1.8 KB) — Entry point, workflow overview
├── plan.md           (4.2 KB) — Plan generation template
├── execute.md        (5.1 KB) — Iteration execution protocol
└── templates/
    └── progress.md   (1.4 KB) — Progress tracking template
```

Total: ~12.5 KB
