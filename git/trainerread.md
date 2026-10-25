# Teleprompter Script: Git for Data Workflows

**Session:** Development Essentials | **Duration:** 2 hours | **Slides:** 31
**Tools covered:** Git, Git LFS, nbdime

### How to read this script

| Marker | Meaning |
|--------|---------|
| 🗣️ **SAY** | Spoken text. Read it naturally, in your own voice. |
| 🖥️ **DO** | Something to show or type on screen. |
| ⏸️ **PAUSE** | Stop and let the room react, or ask a check-in question. |
| 💡 **TIP** | Trainer-only note. Do not read aloud. |
| ➡️ **TRANSITION** | The line that carries you to the next slide. |

### Timing map

| Part | Slides | Topic | Time | Clock (cumulative) |
|------|--------|-------|------|--------------------|
| I | 1–4 | Introduction | 5 min | 0:05 |
| II | 5–9 | Git essentials | 20 min | 0:25 |
| III | 10–15 | Git LFS (hands-on) | 25 min | 0:50 |
| IV | 16–19 | Structuring projects | 25 min | 1:15 |
| V | 20–24 | nbdime (hands-on demo) | 25 min | 1:40 |
| VI | 25–29 | Branching strategies | 20 min | 2:00 |
| VII | 30–31 | Closing and Q&A | 10 min | 2:10 (see pacing note) |

> 💡 **Pacing note:** The agenda lists 5+20+25+25+25+20+10 = **130 minutes**, slightly over "2 hours". If you start late, shorten the Part IV discussion and leave the Part VI exercise as take-home. Part III and Part V are the two live demos, so protect those.

### Before you go live (checklist)

- [ ] Terminal open, font size large, in an empty folder
- [ ] `git`, `git lfs`, `nbdime` installed and tested (`git lfs version`, `nbdiff --version`)
- [ ] A sample CSV of a few MB ready (`data/sales.csv`) and a remote repo created (GitHub, GitLab, or `git init --bare ../remote.git` if offline)
- [ ] Two versions of a notebook ready for the nbdime demo: `old.ipynb`, `new.ipynb`, plus a notebook with a prepared merge conflict
- [ ] Slides open at slide 1; chat window visible for questions

---

# PART I: INTRODUCTION (5 min)

---

## Slide 1: Title: Git for Data Workflows
⏱️ ~1 min

🗣️ **SAY:**
"Good morning, everyone, and welcome. This is a live session on **Git for Data Workflows**, part of our Development Essentials track. We have about two hours together.

Today is about version control for the kind of work you actually do: notebooks, scripts, configuration files, and large data files. We'll use three tools: **Git**, **Git LFS**, and **nbdime**.

Look at the three boxes on the slide, because they are the whole session in a nutshell. One: **Track**. We keep notebooks, scripts and configs under version control. Two: **Store**. We handle large data files with Git LFS. Three: **Collaborate**. We use branching and notebook-aware diffs so several people can work on the same project without stepping on each other."

➡️ **TRANSITION:** "Let me show you how the next two hours are laid out."

---

## Slide 2: Session Agenda
⏱️ ~1 min

🗣️ **SAY:**
"Here's the agenda. Seven parts, two hours.

We start with a **5-minute introduction**. Then **Git essentials**, 20 minutes, where we build the foundation. Then **Git LFS**, 25 minutes, and this is a hands-on part, where we track a large file. After that, **structuring projects**, 25 minutes, on how to lay out notebooks, scripts and configs. Then **notebook versioning with nbdime**, another 25 minutes and our second hands-on demo. **Branching strategies**, 20 minutes. And we finish with **10 minutes for doubts and Q&A**.

So: two hands-on parts, LFS and nbdime, and ten minutes protected for your questions. If something is unclear, please don't wait for the Q&A. Raise your hand or type in the chat at any point."

➡️ **TRANSITION:** "Before we begin, a quick word on what I'm assuming and what you'll leave with."

---

## Slide 3: Prerequisites and Learning Outcomes
⏱️ ~1.5 min

🗣️ **SAY:**
"On the left, the prerequisites. I'm assuming three things: you're **comfortable with the command line at a basic level**, you have some **awareness of software development workflows**, and you understand the **files in a data project**: notebooks, scripts and configs.

You should also have Git installed and have gone through the introductory version-control material. If Git isn't installed yet, please do that now, because we'll type commands together.

On the right are the five **learning outcomes**. By the end of today you should be able to:
1. Use **Git essentials** for data and ETL workflows.
2. **Version-control large files with Git LFS**.
3. **Structure a data-oriented codebase**.
4. **Version-control notebooks with nbdime**.
5. Use **branching strategies** for feature-based work.

I'll come back to this list at the end so we can check each one off."

⏸️ **PAUSE:** "Quick show of hands or a thumbs-up in chat: is everyone able to run `git --version` in a terminal?"

➡️ **TRANSITION:** "Let's start with why we're here at all. Why isn't basic Git enough for data teams?"

---

## Slide 4: Why Data Teams Need More Than Basic Git
⏱️ ~1.5 min

🗣️ **SAY:**
"This table is the motivation for the whole session. Left column: **common problems**. Right column: **where we fix them today**.

First problem: **notebook changes show up as unreadable JSON diffs**. If you've ever opened a pull request that changed a notebook, you know the pain. We fix that in **Part V with nbdime**.

Second: **large datasets bloat the repository and block pushes**. GitHub will refuse a push with a file over 100 MB. We fix that in **Part III with Git LFS**.

Third: **folders and files have no clear structure**. Everyone has opened a project called `final_v2_REAL_final`. We fix that in **Part IV**, structuring data projects.

Fourth: **parallel work overwrites other people's changes**. Two people edit the same file and one of them loses work. That's **Part VI, feature-based branching**.

And notice the line at the bottom: **Part II first builds the Git essentials that all of this relies on.** Everything else sits on top of the basics, so let's get those solid."

➡️ **TRANSITION:** "So, Git essentials. Let's start with the mental model."

---

# PART II: GIT ESSENTIALS (20 min)

---

## Slide 5: How Git Tracks Your Work
⏱️ ~4 min

🗣️ **SAY:**
"Git has four places your work can live. Read the diagram left to right.

**Working directory.** These are the files you actually edit on disk.
When you run **`git add`**, the changes move to the...
**Staging area.** Think of this as a shopping basket: the changes you've *chosen* to include in the next commit.
When you run **`git commit`**, they're saved into the...
**Local repository.** Commits stored on *your machine*.
When you run **`git push`**, they're shared to the...
**Remote repository.** The shared copy on GitHub or GitLab.

And the line at the bottom: **`git pull`** brings remote commits back into your working directory.

Three ideas to take from the boxes underneath.

**Stage selectively.** The staging area exists so you can group related changes into one logical commit. If you changed a cleaning function and also fixed a typo in the README, those are two commits, not one.

**Commit is local.** This one surprises beginners. Nothing is shared with your team until you **push**. A commit on your laptop is private.

**Every commit is a snapshot.** It has an id, an author, a time, and a message. That's what lets you answer 'which version produced this result?' months later."

⏸️ **PAUSE:** "Quick check: if I commit but don't push, can my teammate see it? ... Right, no. It's local until we push."

➡️ **TRANSITION:** "Now let's look at the commands that move work through those four places."

---

## Slide 6: Core Commands You Will Use Daily
⏱️ ~5 min

🖥️ **DO (optional live demo):** In an empty folder, run the commands below as you talk through the table.
```bash
git init
echo "print('hello')" > clean.py
git status
git add clean.py
git commit -m "Add placeholder cleaning script"
git log --oneline
```

🗣️ **SAY:**
"Here are the commands you'll use every single day. Let me go row by row.

**`git init`** creates a new repository. You run it once, at the start of a project folder.

**`git status`** shows changed and staged files. My advice: **run it before every commit.** It's the cheapest habit that prevents the most mistakes.

**`git add <file>`** stages changes. For example, `git add src/clean.py`.

**`git commit -m`** saves a snapshot with a message. One logical change per commit.

**`git log --oneline`** lists history. Great for answering 'when did this change?'

**`git diff`** shows *unstaged* line changes. Use it to review your own work before you stage it.

**`git restore <file>`** discards changes in a file. This is your undo for 'I broke something and want the last committed version back.' Be careful, because discarded uncommitted changes can't be recovered.

And **`git push`** and **`git pull`** share and fetch commits to sync with the remote."

💡 **TIP:** If time is tight, skip the live demo here and save the typing for the LFS section. The table is enough.

➡️ **TRANSITION:** "Knowing the commands is half of it. The other half is writing commits that your future self will thank you for."

---

## Slide 7: Commits That Work for Data Projects
⏱️ ~3.5 min

🗣️ **SAY:**
"Here are the good commit habits.

**One logical change per commit.** **A specific message in the imperative mood**, as if giving an instruction: 'Add', 'Fix', 'Remove'. **Commit often and push regularly.** **Never commit secrets or credentials.** And **check `git status` before every commit**.

Now look at the examples on the right, because this is where it gets real.

A **weak** message: 'update'. Update *what*? A **strong** message: 'Add outlier removal to the sales cleaning step'. Now I know exactly what changed and why.

Another weak one: 'fixed stuff'. A strong one: 'Fix date parsing for UTC timestamps'.

Why does this matter for data work in particular? Because when a number in a report looks wrong three weeks from now, you'll scan the log to find *which change* caused it. 'update' tells you nothing. 'Add outlier removal to the sales cleaning step' tells you exactly where to look."

⏸️ **PAUSE:** "Quick exercise: someone give me a good commit message for 'I changed the null handling in the amount column'." *(Example answer: "Fix null handling in transform step for amount column".)*

➡️ **TRANSITION:** "Now, notebooks. These behave very differently from normal code files, and that's where our problems start."

---

## Slide 8: Tracking Notebooks: The Problem
⏱️ ~3.5 min

🗣️ **SAY:**
"An `.ipynb` file looks like one file, but it's actually a bundle. Inside the one JSON file you have: **source code cells and markdown**, **outputs** such as tables, plots and embedded images, **execution counts** for every cell, and **kernel and metadata**.

Now consider what that does in Git.

**Re-running a notebook changes the counts and outputs even when you didn't change any code.** So Git says 'this file changed' and you have no idea why.

**Diffs show JSON noise, not cell edits.** A reviewer sees a wall of braces and a long base64 string.

**Merge conflicts inside JSON are hard to fix.** One misplaced comma and the notebook won't open.

And **embedded images bloat the repository history**, because every re-run with a plot stores a fresh copy of the image.

The takeaway is that Git is not broken here. It simply doesn't understand notebooks. And the line at the bottom says it: **Part V solves this with nbdime.** Hold that thought."

➡️ **TRANSITION:** "One more piece of foundation before we move on: keeping junk out of the repository."

---

## Slide 9: Keep the Repository Clean with .gitignore
⏱️ ~4 min

🖥️ **DO (optional):** Create the file live.
```bash
cat > .gitignore << 'EOF'
# Python and Jupyter
__pycache__/
.ipynb_checkpoints/
.venv/
# Secrets and local config
.env
# Generated data and logs
data/processed/
*.log
EOF
git add .gitignore
git commit -m "Add .gitignore for data project"
```

🗣️ **SAY:**
"This is a `.gitignore` file. It tells Git what to *not* track. Let's read it by section.

**Python and Jupyter:** `__pycache__`, `.ipynb_checkpoints`, and your virtual environment `.venv`. These are generated by tools and are different on every machine.

**Secrets and local config:** `.env`. This is the most important line in the file. Passwords, API keys and database credentials must never enter Git history. Once a secret is committed, removing it later is painful, and you must assume it's compromised.

**Generated data and logs:** `data/processed/` and `*.log`. If your code can regenerate it, you don't need to version it.

The rules of thumb on the right: **Ignore anything you can regenerate. Always ignore secrets and local settings. Commit the `.gitignore` file itself**, so the whole team shares the same rules. And the last one is a pointer forward: **shared source data goes to Git LFS, not here.** Which is exactly where we're going next."

💡 **TIP:** Create `.gitignore` *before* adding data or notebooks to a project. Retrofitting it later is the most common beginner mistake.

➡️ **TRANSITION:** "That's the foundation. Let's tackle the first big problem: large files."

---

# PART III: GIT LFS (25 min, hands-on)

---

## Slide 10: The Large File Problem
⏱️ ~3 min

🗣️ **SAY:**
"Let's talk about why large files are a problem. Three facts on the left.

**100 MB.** GitHub will block pushes of any single file above this size. You'll get a rejection error.

**Every version.** Git keeps every saved version of a file in history. So if a 100 MB CSV changes ten times, you've stored roughly a gigabyte, not 100 MB.

**Slow clones.** A heavy history makes every clone and fetch slow for *everyone* on the team, including the new joiner on their first day.

The typical culprits are on the right: CSV and Parquet extracts, model files and images, exported results and archives.

And the result, at the bottom: **a blocked push, a repository that's slow to clone, and history you can't trim without rewriting it.** That last one is the nasty part. Once a huge file is in history, removing it means rewriting history, which affects the whole team. So the best fix is to prevent it."

➡️ **TRANSITION:** "Git LFS is the prevention. Here's how it works."

---

## Slide 11: How Git LFS Works
⏱️ ~4 min

🗣️ **SAY:**
"LFS stands for **Large File Storage**. The idea is simple: Git stores a tiny *pointer*, and the real file lives somewhere else. Four steps.

**One: you track a pattern**, for example `*.csv`. That tells LFS which files it should manage.

**Two: Git stores a pointer.** A small text file goes into history, not the data.

**Three: the content goes to LFS storage.** It's uploaded when you push.

**Four: checkout fetches the file.** When you check out or clone, the real content replaces the pointer in your working directory.

Look at the pointer file on the right. Three lines: a **version**, an **`oid`**, which is the SHA-256 hash of the content, and the **size**. That's about 130 bytes standing in for a 100 MB file.

Why this helps: **history stays small because only pointers are versioned in Git.** Cloning stays fast, and you're not carrying every old version of the dataset."

⏸️ **PAUSE:** "So if I look at this file in Git history, do I see the data or the pointer? ... The pointer."

➡️ **TRANSITION:** "Let's do it for real."

---

## Slide 12: Hands-On: Track a Data File with LFS
⏱️ ~6 min

🖥️ **DO:** Type each command live in the terminal. Let learners follow along.
```bash
git lfs install
git lfs track "*.csv"
git add .gitattributes
git add data/sales.csv
git commit -m "Track sales data with LFS"
git push origin main
```

🗣️ **SAY (step by step as you type):**
"Command one: **`git lfs install`**. This sets up the LFS hooks. You do it **once per machine**.

Command two: **`git lfs track "*.csv"`**. This records the pattern in a file called `.gitattributes`. Let me show you.
" *(run `cat .gitattributes` and show the line)*

"Command three: **`git add .gitattributes`**. This is important: **commit the rules first.** If you add the data file before `.gitattributes`, Git will store the whole file normally and LFS won't catch it.

Command four: **`git add data/sales.csv`** stages the data file.

Command five: **`git commit`**. Now the *commit* stores a pointer, not the data.

Command six: **`git push origin main`**. The push uploads the actual content to LFS storage. Watch the terminal: you should see an 'Uploading LFS objects' line."

🖥️ **DO (proof):** Show the pointer.
```bash
git show HEAD:data/sales.csv
```
"This prints the pointer file: version, oid, size. That's what's in Git. The real file is on your disk and in LFS storage."

💡 **TIP:** If anyone's push is rejected for size, they committed the CSV *before* tracking it. The fix is `git lfs migrate import --include="*.csv"` (covered on the next slide). Reassure them that it's a common mistake.

➡️ **TRANSITION:** "Once LFS is set up, here are the commands you'll use to manage it."

---

## Slide 13: Managing LFS Files
⏱️ ~3.5 min

🗣️ **SAY:**
"Five commands to know.

**`git lfs ls-files`** lists the files stored with LFS. Use it to **confirm a file is tracked**. If your file isn't in this list, LFS isn't managing it.

**`git lfs status`** shows pending LFS changes. Run it **before committing**.

**`git lfs pull`** downloads LFS content. Use it **after cloning without content**: if you open a CSV and see a tiny text file, you have the pointer and need to pull.

**`git lfs untrack "*.csv"`** stops tracking a pattern. You'd use that for a change of policy.

**`git lfs migrate import`** moves *existing* history into LFS. This is for when **a large file was committed by mistake**. A warning: it rewrites history, so tell your team before you run it.

And a note at the bottom: **hosting providers set LFS storage and bandwidth quotas.** Check your plan. LFS isn't unlimited, and many teams are surprised by the bill or the cap."

🖥️ **DO (optional):** Run `git lfs ls-files` and `git lfs status` to show real output.

➡️ **TRANSITION:** "Now it's your turn."

---

## Slide 14: Exercise: Try It Yourself
⏱️ ~6 min (learners work; you circulate)

🗣️ **SAY:**
"Five steps. Take about five minutes and follow along.

**One: prepare data.** Place a CSV of a few MB in your data folder. It doesn't need to be huge.

**Two: enable LFS.** Run `git lfs install`, then track the CSV pattern.

**Three: commit and push.** Commit `.gitattributes` *and* the file, then push.

**Four: verify.** Run `git lfs ls-files` to confirm tracking.

**Five: clone and check.** Clone the repo into another folder and run `git lfs pull` if needed.

The **expected result** at the bottom: `git lfs ls-files` lists your file, and the push shows an LFS upload. If you see both, you've succeeded. Drop a ✅ in the chat when you're done, and shout if you're stuck."

💡 **TIP:** While they work, watch for the three common errors: (1) `git lfs: command not found`, which means LFS isn't installed; (2) the file was committed before tracking; (3) forgot to commit `.gitattributes`.

⏸️ **PAUSE:** Wait for ✅ responses, then debrief: "Who saw the LFS upload line? Who had to run `git lfs pull` after cloning?"

➡️ **TRANSITION:** "Before we leave LFS, one honest question: is it always the right answer?"

---

## Slide 15: When LFS Fits and When It Does Not
⏱️ ~2.5 min

🗣️ **SAY:**
"LFS is great, but it isn't universal.

**Good fit:** datasets and assets that **change rarely**, files from **tens of MB up to a few GB**, and teams **already on GitHub or GitLab**.

**Consider alternatives** when you have **very large or fast-changing data**: use object storage like S3, GCS or Azure Blob. When you need **data lineage**, meaning how the data was produced, use a data versioning tool such as **DVC**. And when data **must stay in a secure store**, don't put it in Git at all.

The sentence at the bottom is the key one: **LFS versions the files. It does not record how the data was produced.** So LFS tells you *which* file, not *how it was built*. If lineage matters to you, that's where DVC comes in.

Rule of thumb: LFS for modest, stable files; object storage or DVC when the data gets big, fast-moving or needs lineage."

➡️ **TRANSITION:** "We've solved the large-file problem. Next: how should the whole project be organised?"

---

# PART IV: STRUCTURING DATA PROJECTS (25 min)

---

## Slide 16: Anatomy of a Data Project
⏱️ ~6 min

🗣️ **SAY:**
"This is a layout I recommend as a starting point. Let me walk the tree.

**`data/`** holds `raw/` and `processed/`. **`notebooks/`** is for exploration. **`src/`** holds reusable code. **`configs/`** holds parameters and settings. **`tests/`** holds your checks. At the top level: a **README**, a **`requirements.txt`**, a **`.gitignore`**, and a **`.gitattributes`** for LFS.

Three principles on the right.

**Separate raw from processed.** **Raw data is never edited. Processed data is rebuilt by code.** If you manually fix a raw file, you lose reproducibility. Nobody can tell what you changed.

**Keep exploration apart from reusable code.** **Notebooks explore. The `src` folder holds stable logic.** When a notebook cell works and you'll need it again, promote it into a function in `src`.

**Document the entry points.** **The README explains setup, data sources, and how to run.** A new teammate should be able to read it and be running in minutes.

None of this is Git-specific, but it makes every Git feature we've covered work better: cleaner diffs, cleaner `.gitignore`, clearer reviews."

⏸️ **PAUSE (discussion):** "Look at one of your current projects. Where does it mix raw data, code and outputs in one folder? Think about it for a moment. I'll take one or two examples."

➡️ **TRANSITION:** "Let's make this concrete: for each folder, what goes in it and how does Git treat it?"

---

## Slide 17: What Goes Where, and How Git Treats It
⏱️ ~6 min

🗣️ **SAY:**
"This table connects structure back to everything we've learned. Row by row.

**`data/raw`**: original source data, never edited. Git treatment: **track with LFS, or keep it outside Git** entirely.

**`data/processed`**: cleaned and derived outputs. **Ignore it and regenerate from code.** That matches the `.gitignore` we wrote earlier.

**`notebooks`**: exploration and reporting. **Track, with outputs handled**, which is the nbdime and output-stripping story coming in Part V.

**`src`**: reusable ETL and utility functions. **Track and review in pull requests.** This is your production-quality code, so it deserves proper review.

**`configs`**: paths, parameters, settings. **Track, but never include secrets.**

**`tests`**: checks for code and data. **Track.**

If you remember one row, remember `data/processed`: *ignore and regenerate*. And one more: `src` goes through pull requests, because that's where quality matters most."

➡️ **TRANSITION:** "Two of those folders, `notebooks` and `src`, raise a common question: when do I use which?"

---

## Slide 18: Notebooks and Scripts: Choose the Right Tool
⏱️ ~5 min

🗣️ **SAY:**
"Neither is better. They have different jobs.

**Notebooks** are for **exploration and quick experiments**, **visualisation and storytelling**, and **sharing findings with stakeholders**.

**Scripts and modules** are for **reusable ETL steps**, **scheduled and automated jobs**, and **code that is tested and reviewed**.

Why does this matter for Git? Plain `.py` files diff and review beautifully. Notebooks don't. So the more logic you move into `src`, the easier your reviews become.

The workflow at the bottom is the one to adopt: **prototype in a notebook, then move stable logic into `src` and import it back.** You get fast exploration, and your final logic lives in testable, reviewable code. The notebook then becomes a thin layer that calls functions from `src`."

⏸️ **PAUSE (discussion):** "When should logic move out of a notebook? Hint: when you copy-paste it into a second notebook."

➡️ **TRANSITION:** "The last structural piece: configs, secrets and environments."

---

## Slide 19: Configs, Secrets and Environments
⏱️ ~5 min

🗣️ **SAY:**
"Three boxes.

**Configs.** **Track parameters and paths. One file per environment**: dev, test, prod. **No hard-coded paths in code.** If I see `C:\Users\priya\data\sales.csv` inside a script, it won't run on anyone else's machine.

**Secrets.** **Keep them in `.env`. Ignore `.env` in Git. Commit a `.env.example` template.** The template shows the variable *names* with placeholder values, like `DB_PASSWORD=changeme`, so teammates know what to fill in, without exposing real credentials.

**Environments.** **Pin versions in `requirements.txt`, or use `environment.yml`.** The goal is **rebuild the same setup anywhere**. 'It works on my machine' is the enemy.

The line at the bottom is the test of a well-structured project: **clone, install, edit one config file, run.** Four steps. If your project needs more than that, something is hidden."

💡 **TIP:** If anyone asks "what if I already committed a secret?": remove it with `git rm --cached .env`, add it to `.gitignore`, and **rotate the secret immediately**, because it still exists in history.

➡️ **TRANSITION:** "Now back to the notebook problem we flagged in Part II. Time for nbdime."

---

# PART V: NOTEBOOK VERSIONING WITH NBDIME (25 min, hands-on demo)

---

## Slide 20: Why Plain Git Diffs Fail on Notebooks
⏱️ ~4 min

🖥️ **DO (optional, powerful):** Show a real noisy diff first.
```bash
git diff --no-ext-diff notebooks/01_eda.ipynb
```
"Look at that. Can anyone tell me what changed? ... Neither can I."

🗣️ **SAY:**
"This table compares plain `git diff` with nbdime across four aspects.

**Unit of change.** Git sees **lines of JSON**. nbdime sees **notebook cells**.

**Outputs.** Git shows **raw text, including base64 images**. nbdime **renders them side by side**.

**Execution counts.** In Git they **appear as real changes**. In nbdime they **can be ignored**.

**Review effort.** Git: **high and error-prone**. nbdime: **low and focused on content**.

The summary line: **nbdime understands notebook structure, so reviews focus on what really changed.** It does the reading so your reviewers don't have to decode JSON."

➡️ **TRANSITION:** "nbdime is actually a small toolkit. Here's what's in it."

---

## Slide 21: The nbdime Toolkit
⏱️ ~3 min

🗣️ **SAY:**
"Five pieces.

**`nbdiff`**: compare two notebooks, in the **terminal**.
**`nbdiff-web`**: a rich visual diff with rendered outputs, in the **browser**.
**`nbmerge`**: a three-way merge of notebooks, in the **terminal**.
**`nbmerge-web`**: resolve conflicts visually, in the **browser**.
And **`nbdime config-git`**: plugs nbdime into `git diff` and `git merge`. It's a **one-time setup**.

A quick way to remember: *diff* is for reading changes, *merge* is for combining changes, and the *web* versions are the visual ones. For daily work, the config step matters most, because once it's done, your normal Git commands automatically become notebook-aware."

➡️ **TRANSITION:** "Let me show you. I'll install it and run a real diff."

---

## Slide 22: Demo: Install and Use nbdime
⏱️ ~7 min

🖥️ **DO:** Type each command live.
```bash
pip install nbdime
nbdime config-git --enable --global
git diff notebooks/01_eda.ipynb
nbdiff-web old.ipynb new.ipynb
```

🗣️ **SAY (step by step):**
"Step one: **`pip install nbdime`**. This installs the toolkit.

Step two: **`nbdime config-git --enable --global`**. This tells Git to use nbdime for notebook files. Notice `--global`, so it applies to every repo on this machine. You only do this once.

Step three: **`git diff notebooks/01_eda.ipynb`**. Same command as before, but now look at the output. Instead of JSON braces, I'm seeing a cell-aware diff: *this* cell's source changed, *this* output changed. Compare that to what we saw a moment ago."

🖥️ **DO:** Run `nbdiff-web old.ipynb new.ipynb`. A browser tab opens.

🗣️ **SAY:**
"Step four: **`nbdiff-web old.ipynb new.ipynb`**. This opens a rich diff in the browser. Look at how it's organised: source changes, output changes and metadata changes are shown separately. Outputs such as tables and plots are *rendered*, so I can see an old plot next to a new plot. I can also toggle off the things I don't care about, like outputs or execution counts, to focus on the code."

🖥️ **DO (optional):** `nbdiff -s notebooks/01_eda.ipynb` to show sources-only, and `--ignore-outputs` in the web view.

⏸️ **PAUSE:** "Questions on the diff view before we move to merging?"

💡 **TIP:** If the browser doesn't open (remote or locked-down machine), fall back to terminal `nbdiff`, which still shows the cell-aware output.

➡️ **TRANSITION:** "Diffs are for reading. What about when two people changed the same notebook?"

---

## Slide 23: Merging Notebooks and Resolving Conflicts
⏱️ ~6 min

🖥️ **DO:** Use a pre-prepared repo where two branches edit the same cell.
```bash
git merge feature/stats-cell
# CONFLICT reported
git mergetool          # or: nbmerge-web ...
# choose per cell, save
git add notebooks/01_eda.ipynb
git commit
```

🗣️ **SAY:**
"Five steps.

**One: two branches edit one notebook.** Both changed cells in the same file.

**Two: the merge reports a conflict.** Run `git merge` and read the message.

**Three: open the merge tool.** Use `nbmerge-web`, or `git mergetool`.

**Four: choose per cell.** For each conflicting cell, you can keep either version or combine them.

**Five: save and commit.** Run `git add`, then commit the merge."

*(Walk through it live: show the conflict, open the merge tool, pick a side for the conflicting cell, save, commit.)*

"The line at the bottom is the whole point: **conflicts are resolved per cell, never inside raw JSON.** Without nbdime, you'd be hand-editing JSON with conflict markers inside it, and one bad edit corrupts the notebook. With nbdime, you make a decision per cell."

💡 **TIP:** If a merge produces invalid JSON, the escape hatch is `git merge --abort`, then retry using `git mergetool`. Always mention this, because it makes learners less afraid of merges.

➡️ **TRANSITION:** "Tools help, but habits matter more. Four habits for healthy notebooks."

---

## Slide 24: Notebook Hygiene Habits
⏱️ ~5 min

🗣️ **SAY:**
"Four habits.

**One: restart and run all.** Before you commit, **confirm the notebook runs top to bottom.** Notebooks let you run cells out of order, so a notebook can look fine but fail from scratch. Restart the kernel and run everything. If it passes, it's reproducible.

**Two: handle bulky outputs.** **Clear heavy outputs, or strip them with a pre-commit tool.** A tool called `nbstripout` can do this automatically on every commit. The exception: if the notebook *is* the published report, you might want the outputs committed. That's a deliberate choice.

**Three: number and name notebooks.** Names like **`01_eda.ipynb`** make order and purpose clear. Compare that with `Untitled7.ipynb`.

**Four: one owner at a time.** **Avoid two people editing the same notebook in parallel.** Even with nbdime, notebook merges are inherently hard. The easiest conflict to resolve is the one that never happens.

Together with nbdime, these habits make notebooks first-class citizens in your Git workflow."

⏸️ **PAUSE:** "Which of these four habits do you think your team struggles with most? Put a number in chat."

➡️ **TRANSITION:** "Last technical topic: how a whole team works together safely, with branches."

---

# PART VI: BRANCHING STRATEGIES (20 min)

---

## Slide 25: Why Branch: Isolate Work, Keep Main Stable
⏱️ ~3.5 min

🗣️ **SAY:**
"Look at the diagram. The top line is **`main`**. Below it, a branch called **`feature/clean-sales-data`**. The branch is created *from* main, work happens on it, and then it comes back to main through a **pull request, review, and merge**.

Why do this? Three reasons.

**Isolation.** **Experiments cannot break the shared code.** You can try something risky on a branch with zero consequences for anyone else.

**Stable main.** **Main always holds reviewed, working work.** Anyone can clone main at any time and trust that it runs.

**Parallel progress.** **Several people change the project at once.** Each person has their own branch, so nobody blocks anybody.

Think of main as the published edition of a book, and branches as individual drafts. You only publish what's been reviewed."

➡️ **TRANSITION:** "Here are the commands."

---

## Slide 26: Branching Commands
⏱️ ~4 min

🖥️ **DO (live, quick):**
```bash
git branch
git switch -c feature/add-region-filter
# make a small change, commit it
git switch main
git merge feature/add-region-filter
git branch -d feature/add-region-filter
```

🗣️ **SAY:**
"Six commands.

**`git branch`**: list branches. See where you are.
**`git switch -c feature/name`**: create *and* switch to a new branch. This is how you start new work.
**`git switch main`**: go back to main. Do this to update before merging.
**`git merge feature/name`**: merge a branch into the *current* one. Combine finished work.
**`git branch -d feature/name`**: delete a merged branch. Tidy up.
**`git push -u origin feature/name`**: publish and track a branch. This is how you open a pull request.

One detail: `merge` merges *into the branch you're currently on*. So to merge a feature into main, first switch to main."

➡️ **TRANSITION:** "Now let's put the commands into a repeatable workflow."

---

## Slide 27: The Feature-Branch Workflow
⏱️ ~4 min

🗣️ **SAY:**
"Six steps, and this is the workflow most data teams use.

**One: branch from main.** Start from the *latest* main, so pull first.
**Two: commit small changes.** One logical change per commit.
**Three: push the branch.** Share it with the remote.
**Four: open a pull request.** **Describe what and why.** If you changed a schema or a column, say so in the description, because downstream consumers need to know.
**Five: review and fix.** Teammates comment, you update.
**Six: merge and delete.** Merge to main, remove the branch.

And the rule at the bottom: **keep branches short-lived: days, not weeks.** The longer a branch lives, the further it drifts from main, and the more painful the merge becomes."

⏸️ **PAUSE:** "Why is a three-week-old branch risky? ... Because main has moved on and the conflicts pile up."

➡️ **TRANSITION:** "Feature branching isn't the only strategy. Let's compare the main options."

---

## Slide 28: Choosing a Branching Strategy
⏱️ ~4 min

🗣️ **SAY:**
"Three strategies.

**Feature branching**: one short branch per feature, merged by pull request. **Best fit: most data teams.** This is our recommended default.

**Trunk-based**: very small changes merged to main daily. **Best fit: mature teams with strong automation**, meaning good tests and CI. It moves fast, but it needs a safety net.

**Git Flow**: long-lived `develop`, release and hotfix branches. **Best fit: fixed release schedules**, where you maintain multiple versions in production.

For most data teams, start with **feature branching**. Add complexity only when you have a real reason.

Finally, the naming convention at the bottom: **`feature/clean-sales-data`**, **`fix/date-parsing`**, **`exp/lstm-forecast`**. A prefix, a slash, a short description. The `exp/` prefix is especially useful for data science: it signals that this is an experiment, and you only merge what's validated."

➡️ **TRANSITION:** "Last slide in this part: practices specific to data work."

---

## Slide 29: Branching Practices for Data Work
⏱️ ~4.5 min

🗣️ **SAY:**
"Four practices.

**One: one branch per feature or experiment.** This **keeps results traceable to a change.** If a model's accuracy changed, you can point to the exact branch.

**Two: sync with main often.** **Pull main into your branch to avoid large conflicts.** A little and often beats a big painful merge.

**Three: avoid parallel notebook edits.** **Agree who edits which notebook.** Even with nbdime, prevention beats cure.

**Four: protect main.** **Require pull requests and reviews before merging.** On GitHub or GitLab, you can enforce this in the repository settings, so nobody pushes straight to main by accident.

The line at the bottom: **small branches and frequent merges prevent painful conflicts.**"

💡 **TIP (if asked about rebase):** "Merge preserves history and is safe on shared branches. Rebase gives a linear history but rewrites commits, so **only rebase your own, unpushed branches**."

💡 **TIP (if time is short):** Skip the live demo on slide 26 and set the full feature-cycle (branch, change, simulate a teammate change on main, merge, view `git log --oneline --graph --all`) as take-home practice.

➡️ **TRANSITION:** "That's all the content. Let's pull it together."

---

# PART VII: CLOSING AND Q&A (10 min)

---

## Slide 30: Key Takeaways
⏱️ ~3 min

🗣️ **SAY:**
"Let's close the loop on the five learning outcomes from the start. One line each.

**Git essentials:** small, specific commits and a clean `.gitignore`.
**Git LFS:** pointers in Git, large content in LFS storage.
**Project structure:** separate raw data, notebooks, source and configs.
**nbdime:** cell-aware diffs and merges for notebooks.
**Branching:** short feature branches, reviewed before merging.

If you take away just one thing from today: pick *one* habit and apply it on Monday. Maybe it's adding a `.gitignore`, or tracking your CSVs with LFS, or installing nbdime.

And a heads-up on what's next: our upcoming module is **Application Deployment**, which builds on everything we did today."

➡️ **TRANSITION:** "Now the part I want you to drive: your questions."

---

## Slide 31: Questions and Discussion
⏱️ ~7 min

🗣️ **SAY:**
"We have about ten minutes for doubts. Version control is a habit: small commits, clear structure, short branches keep data projects healthy.

I've grouped the topics on the slide to help you ask:
1. **Git and LFS**: commands, quotas, large files.
2. **Notebooks and nbdime**: diffs, merges, hygiene.
3. **Branching**: workflow and conflicts.

Who'd like to start?"

### Prepared questions (if the room is quiet)

| Question | Short answer |
|----------|--------------|
| What goes into Git history when I commit an LFS-tracked file? | A small pointer file (version, SHA-256 oid, size). The real content is in LFS storage. |
| Why commit `.gitattributes` with, or before, the data file? | So LFS handles the file from its very first commit, and teammates share the same rules. |
| Name two reasons raw notebook diffs are hard to read. | Outputs and base64 images, plus metadata such as execution counts. |
| Which nbdime command opens a browser diff? | `nbdiff-web`. |
| GitHub Flow vs Git Flow? | GitHub Flow: `main` plus short-lived feature branches. Git Flow adds long-lived `develop`, `release` and `hotfix` branches for scheduled releases. |
| When is rebasing unsafe? | On shared or already-pushed branches, because it rewrites history. |

### Quick troubleshooting (for live questions)

| Symptom | Fix |
|---------|-----|
| CSV shows as a tiny text pointer after clone | `git lfs install`, then `git lfs pull` |
| Push rejected, file over 100 MB | `git lfs migrate import --include="*.csv"` (rewrites history; tell the team) |
| `git lfs: command not found` | Install Git LFS, then `git lfs install` |
| `git diff` on a notebook still shows raw JSON | `nbdime config-git --enable --global`, or use `nbdiff` directly |
| Notebook merge produced invalid JSON | `git merge --abort`, retry with `git mergetool` or `nbmerge-web` |
| `.env` committed by mistake | `git rm --cached .env`, add to `.gitignore`, and **rotate the secrets** |
| Committed to `main` by accident (not pushed) | `git branch feature/x`, then `git reset --hard origin/main` |

🗣️ **SAY (closing line):**
"Thank you, everyone. Good commits, clear structure, short branches. See you in the next module on Application Deployment."

---

## Appendix: One-Page Command Cheat Sheet (to share after the session)

```bash
# Basics
git init | git status | git add . | git commit -m "msg"
git log --oneline --graph --all
git diff | git diff --staged
git restore <file> | git restore --staged <file>

# LFS
git lfs install
git lfs track "*.csv"
git lfs ls-files
git lfs status
git lfs pull
git lfs migrate import --include="*.csv"

# Notebooks
pip install nbdime
nbdime config-git --enable --global
nbdiff <nb> | nbdiff-web <old> <new>
git mergetool

# Branching
git switch -c feature/<name>
git merge <branch>
git branch -d <branch>
git push -u origin <branch>
```
