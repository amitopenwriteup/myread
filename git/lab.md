# Git for Data Workflows: Lab Objectives Explained

A guide to what each lab is for, what learners do, and what they should take away. It ends with a summary you can use to explain the whole session.

---

## At a Glance

| Part | Topic | Lab objective in one line |
|------|-------|---------------------------|
| I | Introduction | Understand why data work needs version control |
| II | Git essentials | Build the repo and learn the everyday Git loop |
| III | Git LFS | Version large data files without bloating the repo |
| IV | Project structure | Organise a data project so it is reproducible and teamwork-friendly |
| V | nbdime | Read and merge Jupyter notebook changes sensibly |
| VI | Branching | Develop features safely and collaboratively |
| VII | Q&A | Check understanding and troubleshoot |

All labs build one repository, **`sales-etl-workshop`**, so each part adds to the previous one.

---

## Objective 1: Apply Git Essentials to Data and ETL Workflows (Part II)

**What the lab does:** Learners create a repo, add a small ETL script (`etl.py`), write a `.gitignore`, and make their first commits.

**Why it matters:**
- Data work changes constantly. Git gives every change an author, timestamp and message, so you can answer "which version produced this result?"
- A `.gitignore` written *before* data and notebooks arrive keeps secrets (`.env`, `credentials.json`), generated outputs and caches out of history. Secrets that reach history are hard to remove.
- Good commit messages (imperative, one logical change each) make history useful for debugging and audits.

**Skills practised:**
- The flow: working directory → staging → local repo → remote
- `git status`, `add`, `commit`, `log`, `diff`, `restore`, `revert`
- Undoing mistakes safely (`git restore` for uncommitted changes, `git revert` for committed ones)

**What learners should be able to say afterwards:** "I know what each Git area does, I can commit cleanly, and I know what must never be committed."

---

## Objective 2: Version-Control Large Files Using Git LFS (Part III)

**What the lab does:** Learners generate a ~60 MB, 1.5-million-row CSV, tell LFS to track it, commit it, and inspect the resulting pointer file.

**Why it matters:**
- Git stores every version of every file. A 100 MB CSV edited 10 times can add roughly 1 GB to the repo.
- Hosts such as GitHub reject files over 100 MB in regular Git.
- LFS replaces the big file in history with a tiny pointer (version, SHA-256 `oid`, size) and keeps the real content on an LFS server.

**Key ideas to stress:**
1. `.gitattributes` holds the tracking rules, so commit it **before or with** the data file.
2. `git show HEAD:data/raw/sales.csv` printing a pointer, not data, proves LFS worked.
3. Tracking *after* a big file was already committed does not fix history. You need `git lfs migrate import`, which rewrites history, so coordinate with the team.
4. LFS is not for everything. Very large or fast-changing datasets may belong in object storage (S3, GCS, Azure Blob) or a tool like DVC.

**Skills practised:** `git lfs install`, `track`, `ls-files`, `status`, `pull`, `migrate import`.

---

## Objective 3: Structure a Data-Oriented Codebase (Part IV)

**What the lab does:** Learners restructure the repo into `config/`, `data/`, `notebooks/`, `src/`, `scripts/`, `tests/` and `docs/`, and add a README, `requirements.txt` and `.env.example`.

**Why it matters:** A predictable layout helps teammates find things and lets CI/CD run scripts. It also separates **exploration** (notebooks) from **production** (tested modules).

**Design principles to explain:**

| Principle | Meaning |
|-----------|---------|
| Separate code from data | Logic in `src/`, data in `data/` |
| Raw data is immutable | Never edit `data/raw/`; write results elsewhere |
| Notebooks explore, modules ship | Move stable logic from notebooks into `src/` |
| Config out of code | No hard-coded paths, credentials or environment names |
| Secrets never in Git | Commit `.env.example`, ignore the real `.env` |
| Reproducibility | Pin dependencies and document how to run |

**Practical tip:** Because `.gitignore` excludes `data/processed/`, `.gitkeep` files inside need `git add -f`. A cleaner option is to ignore the folder's contents (`data/processed/*`) and un-ignore the placeholder (`!data/processed/.gitkeep`).

**Discussion prompts:** Which of your projects mix raw data, code and outputs? What belongs in `data/sample/`? When should notebook logic move into `src/`?

---

## Objective 4: Diff and Merge Jupyter Notebooks with nbdime (Part V)

**What the lab does:** Learners build a small notebook, change and re-run it, and compare plain `git diff` with nbdime's output. They then create and resolve a merge conflict on the same notebook cell.

**Why it matters:** A `.ipynb` is JSON containing code, outputs and metadata. Re-running a cell changes execution counts and embedded base64 images, so diffs are mostly noise and reviewers cannot see what really changed.

**What nbdime adds:**
- `nbdiff` / `nbdiff-web`: content-aware diffs that separate source, outputs and metadata
- `nbmerge` / `nbmerge-web`: three-way merges that do not corrupt the JSON
- Git integration via `nbdime config-git --enable --global`, so `git diff` and `git mergetool` use it automatically

**Moments to demonstrate:**
1. `git diff --no-ext-diff` (noisy) versus `nbdiff` (readable)
2. `nbdiff -s` to view source changes only
3. A same-cell conflict resolved through `git mergetool`

**Team habits to reinforce:**
- Clear outputs before committing, or use `nbstripout`
- Keep notebooks small and focused
- Avoid two people editing the same notebook at once
- Restart the kernel and run all cells before committing
- Move reusable logic into `.py` files, which review well

---

## Objective 5: Apply Feature-Branch Strategies (Part VI)

**What the lab does:** Learners create `feature/add-region-filter`, commit a change, simulate a teammate changing `main`, integrate with a merge, and view the graph with `git log --oneline --graph --all`.

**Why it matters:** Branches isolate work so `main` stays stable, and pull requests give a place for review and tests.

**Strategies to compare:**

| Strategy | Core idea | Best for |
|----------|-----------|----------|
| GitHub Flow | `main` always deployable; short-lived feature branches; PR merges | Most data teams |
| Git Flow | `main` + `develop` + `feature/`, `release/`, `hotfix/` | Scheduled releases, multiple live versions |
| Trunk-based | Very short branches, merged many times a day, feature flags | Mature CI/CD teams |

**Key rules:**
- Use clear branch names: `feature/`, `fix/`, `exp/`, `docs/`.
- Merge is safe on shared branches. **Rebase only your own unpushed or personal branches**, because it rewrites history.
- Conflicts are resolved by editing out the `<<<<<<<`, `=======` and `>>>>>>>` markers, then `git add` and `git commit`.
- Data-specific cautions: use `exp/*` branches for experiments, avoid many branches each changing big LFS files (each version uses storage), protect `main`, and flag schema changes in PR descriptions.

---

## Part VII: Doubt Resolution and Self-Check

Use the six self-check questions to confirm understanding:

1. What does Git store for an LFS-tracked file? *A small pointer file.*
2. Why commit `.gitattributes` early? *So LFS handles the file from its first commit and everyone shares the rules.*
3. Why are raw notebook diffs hard to read? *Outputs/base64 blobs and metadata like execution counts.*
4. Which command opens a browser diff? *`nbdiff-web`.*
5. GitHub Flow vs Git Flow? *Short-lived branches off `main` versus long-lived `develop`, `release` and `hotfix` branches.*
6. When is rebasing unsafe? *On shared or already-pushed branches.*

**Common troubleshooting to mention:** LFS pointer files after clone (`git lfs install` then `git lfs pull`), push rejected for files over 100 MB (`git lfs migrate import`), a committed `.env` (`git rm --cached`, update `.gitignore`, **rotate secrets**), and a failed notebook merge (`git merge --abort`, then retry with `git mergetool`).

---

## Summary to Explain

**In one sentence:** Data projects need Git just like software projects, but with three adjustments: large files go through **LFS**, notebooks are read with **nbdime**, and the project is laid out so exploration and production code stay separate.

**Talking points:**

1. **Why Git for data work?** Every change is traceable and reversible, and teams can work in parallel without overwriting each other.
2. **Where plain Git falls short:** big binary files bloat the repo, notebooks produce noisy JSON diffs, and secrets can leak into history. Each gap has a fix.
3. **Git LFS:** history holds tiny pointers while real data lives on an LFS server. Commit `.gitattributes` first. Use object storage or DVC for very large datasets.
4. **Project structure:** keep code, data, config and notebooks apart. Raw data is immutable, config is external, secrets stay out of Git, and stable notebook logic moves to `src/`.
5. **nbdime:** understands notebook structure, so you can review source changes separately from outputs and merge without corrupting JSON. Pair it with `nbstripout` and small notebooks.
6. **Branching:** use short-lived feature branches and pull requests, protect `main`, merge on shared branches and rebase only on personal ones.

**What learners walk away with:** a working `sales-etl-workshop` repo that demonstrates all five skills, plus a command cheat sheet and troubleshooting table they can reuse.

**Next step:** the upcoming module on **Application Deployment** builds on this foundation.
