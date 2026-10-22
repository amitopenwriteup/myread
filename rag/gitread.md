# Git for Data Workflows
## Trainer Script (Teleprompter Format)

**How to use this script**

- Read the plain text out loud. It is written the way you speak.
- Text in **[BRACKETS]** is a stage cue. Do not read it aloud.
- Each line is one breath. Pause at every blank line.
- Times are guides. The live session is about 2 hours.

| Part | Slides | Time | Type |
|---|---|---|---|
| Opening and I. Introduction | 1 to 4 | 5 min | Theory |
| II. Git essentials | 5 to 9 | 20 min | Theory |
| III. Git LFS | 10 to 15 | 25 min | Simple hands-on |
| IV. Structuring projects | 16 to 19 | 25 min | Theory |
| V. Notebook versioning | 20 to 24 | 25 min | Hands-on demo |
| VI. Branching strategies | 25 to 29 | 20 min | Theory |
| VII. Doubts and Q and A | 30 to 31 | 10 min | Discussion |

---

# OPENING AND PART I: INTRODUCTION

---

## SLIDE 1 | Title
**[1 min]**

Good morning, everyone.
Welcome to this live session on Git for data workflows.

You already know basic Git.
Today we apply it to the things data teams actually work with.
Notebooks.
Large data files.
Project folders.
And teammates changing the same work at the same time.

Our tools are Git, Git LFS and nbdime.

**[POINT to the three cards]**
Track.
Store.
Collaborate.
Those are the three ideas for today.

**[CLICK]**

---

## SLIDE 2 | Session agenda
**[1 min]**

Here is the plan for about two hours.

Part one is a five minute introduction.
Part two is Git essentials, twenty minutes of theory.
Part three is Git LFS, twenty-five minutes, and it has a simple hands-on.
Part four is structuring data projects, twenty-five minutes of theory.
Part five is notebook versioning with nbdime, twenty-five minutes, and I will demo it.
Part six is branching strategies, twenty minutes.
Part seven is ten minutes for doubts and questions.

**[POINT to the right-hand cards]**
Two hands-on parts.
Please keep a terminal open.

---

## SLIDE 3 | Prerequisites and learning outcomes
**[1 min]**

Quick check on prerequisites.

You should be comfortable with a command line.
You should know a little about software development workflows.
And you should know what a data project contains.
Notebooks, scripts and config files.

**[ASK]** Is Git installed and working on your machine?
**[Wait for a thumbs up or chat replies.]**

**[POINT to the right card]**
By the end, you will be able to do five things.
Use Git for data and ETL work.
Version large files with Git LFS.
Structure a data codebase.
Version notebooks with nbdime.
And choose a branching strategy for feature work.

---

## SLIDE 4 | Why data teams need more than basic Git
**[2 min]**

Why are we here?
Because basic Git was designed for source code.
Data projects break it in four ways.

**[POINT to row one]**
Notebook changes show up as unreadable JSON.
Part five fixes that with nbdime.

**[POINT to row two]**
Large datasets bloat the repository and can block pushes.
Part three fixes that with Git LFS.

**[POINT to row three]**
Folders and files with no clear structure.
Part four covers project structure.

**[POINT to row four]**
Parallel work overwrites other people's changes.
Part six covers branching.

**[POINT to the bottom bar]**
But first, we need a solid base.
So Part two builds the Git essentials that everything else relies on.

---

# PART II: GIT ESSENTIALS (20 MIN)

---

## SLIDE 5 | How Git tracks your work
**[4 min]**

Let me draw the whole picture first.
Git has four places your work can live.

**[POINT left to right]**
The working directory. These are the files you edit.
The staging area. These are the changes you picked for the next commit.
The local repository. These are commits saved on your machine.
The remote repository. This is the shared copy on GitHub or GitLab.

**[POINT to the commands under the arrows]**
git add moves changes to staging.
git commit saves them locally.
git push shares them with the team.

**[POINT to the three cards]**
Three things to remember.
Staging lets you choose. Group related changes into one logical commit.
A commit is local. Nothing reaches your team until you push.
And every commit is a snapshot with an id, an author, a time and a message.

**[POINT to the bottom bar]**
And in the other direction, git pull brings remote commits back to you.

**[ASK]** If I commit but do not push, can my teammate see it?
**[Expected answer: no.]**

---

## SLIDE 6 | Core commands you will use daily
**[5 min]**

These are the eight commands you will use every day.
**[DEMO: Open a terminal in a small demo repository and run them as you speak.]**

git init creates a repository.
git status shows what changed and what is staged.
Make it a habit. Run status before every commit.

git add stages a file for the next commit.
git commit with the dash m flag saves the snapshot and its message.

git log with oneline lists the history.
Use it to find when something changed.

git diff shows line changes you have not staged yet.
Review before you stage.

git restore discards changes in a file.
That is your undo for a mistake.

And git push and git pull sync with the remote.

**[SAY]** If you remember only one habit, remember status, then add, then commit.

---

## SLIDE 7 | Commits that work for data projects
**[4 min]**

A commit is a message to your future self and your team.

**[POINT to the left card]**
Five habits.
One logical change per commit.
A specific message, in the imperative mood.
Commit often and push regularly.
Never commit secrets or credentials.
And check git status before every commit.

**[POINT to the right cards]**
Compare these two messages.
Weak. The word update.
Strong. Add outlier removal to the sales cleaning step.

Weak. Fixed stuff.
Strong. Fix date parsing for UTC timestamps.

**[PAUSE]**

With the strong message, a teammate knows what changed without opening the diff.
In data work this matters even more.
A change to a cleaning step can change every number downstream.

**[ASK]** Which message would you rather find in the log six months from now?

---

## SLIDE 8 | Tracking notebooks: the problem
**[4 min]**

Now the first data-specific problem.
Notebooks.

**[POINT to the left card]**
An .ipynb file is one JSON file.
It holds your code cells and markdown.
It holds the outputs, such as tables, plots and embedded images.
It holds an execution count for every cell.
And it holds kernel and metadata.

**[POINT to the right card]**
So what happens in Git?
Re-run a notebook and the counts and outputs change, even if no code changed.
The diff shows JSON noise, not cell edits.
Merge conflicts inside JSON are hard to fix.
And embedded images bloat the history.

**[DEMO, optional: Open an .ipynb file in a text editor to show the JSON.]**

**[POINT to the bottom bar]**
Do not worry. Part five solves this with nbdime.

---

## SLIDE 9 | Keep the repository clean with .gitignore
**[3 min]**

Before moving on, one file every data repository needs.
The .gitignore.

**[POINT to the code card]**
We ignore Python cache folders.
We ignore notebook checkpoints and the virtual environment.
We ignore the .env file, because it holds secrets.
And we ignore generated data and logs.

**[POINT to the right card]**
Four rules of thumb.
Ignore anything you can regenerate.
Always ignore secrets and local settings.
Commit the .gitignore file itself, so the whole team shares the same rules.
And shared source data does not belong here. It goes to Git LFS, which is our next topic.

**[SAY]** Note that ignoring a file already tracked needs git rm with the cached flag.
It will not stop tracking by itself.

**[Part II complete. Move to Part III.]**

---

# PART III: GIT LFS (25 MIN, SIMPLE HANDS-ON)

---

## SLIDE 10 | The large file problem
**[3 min]**

Let us talk about size.

**[POINT to the left card]**
GitHub blocks pushes of single files above one hundred megabytes.
That is a hard limit.

**[POINT to the middle card]**
Even below that limit, Git keeps every version of every file in history.
Change a fifty megabyte CSV ten times and you carry roughly five hundred megabytes of history.

**[POINT to the right card]**
That makes every clone and fetch slow, for everyone.

**[POINT to the bottom cards]**
The usual culprits are CSV and Parquet extracts, model files and images, and exported results.
The result is a blocked push, a slow repository, and history you cannot trim without rewriting it.

**[SAY]** Git LFS solves this. Let me show how.

---

## SLIDE 11 | How Git LFS works
**[4 min]**

The idea is simple.
Git does not store the big file.
Git stores a small pointer.

**[POINT left to right]**
Step one. You track a pattern, for example star dot csv.
Step two. Git stores a small pointer text file in history.
Step three. The real content goes to separate LFS storage when you push.
Step four. On checkout, LFS fetches the real file and replaces the pointer.

**[POINT to the pointer file]**
This is what a pointer looks like.
Three lines.
A version, a hash called the oid, and the size.
About a hundred bytes, instead of a hundred megabytes.

**[POINT to the right card]**
So history stays small.
Only pointers are versioned in Git.

**[ASK]** Where does the real file content live?
**[Expected answer: in LFS storage on the remote.]**

---

## SLIDE 12 | Hands-on: track a data file with LFS
**[6 min]**

Now we do it.
**[DEMO: Type each command live. Ask learners to follow in their own terminal.]**

**[Line 1]**
git lfs install.
This sets up LFS hooks once per machine.

**[Line 2]**
git lfs track, with star dot csv in quotes.
This writes a rule into a file called .gitattributes.

**[Line 3]**
git add .gitattributes.
Commit the rules first, so teammates track the same patterns.

**[Line 4]**
git add data slash sales.csv.
This stages the data file.

**[Line 5]**
git commit with a message.
Git commits a pointer, not the file.

**[Line 6]**
git push origin main.
The push uploads the content to LFS storage.

**[PAUSE]**

**[SAY]** If your push is rejected, tell me in the chat now.
The most common cause is forgetting to commit .gitattributes first.

---

## SLIDE 13 | Managing LFS files
**[4 min]**

Once files are tracked, here are the commands you will need.

**[Walk down the table]**
git lfs ls-files lists the files stored with LFS.
Use it to confirm that a file is tracked.

git lfs status shows pending LFS changes.
Run it before committing.

git lfs pull downloads LFS content.
Use it after cloning without content.

git lfs untrack removes a pattern.
Use it when the policy changes.

git lfs migrate import moves existing history into LFS.
Use it when a large file was committed by mistake.

**[WARN]** Migrate rewrites history.
Coordinate with your team before you use it on shared branches.

**[POINT to the bottom bar]**
And check your quotas.
Hosting providers limit LFS storage and bandwidth.

---

## SLIDE 14 | Exercise: try it yourself
**[6 min]**

Your turn.
You have about six minutes.

**[Read the five steps]**
One. Place a CSV of a few megabytes in the data folder.
Two. Run git lfs install, then track the CSV pattern.
Three. Commit .gitattributes and the file, then push.
Four. Run git lfs ls-files to confirm tracking.
Five. Clone the repository elsewhere, and run git lfs pull if needed.

**[POINT to the bottom bar]**
The expected result.
git lfs ls-files lists your file, and the push shows an LFS upload.

**[Walk through the chat and help.]**
**[Ask someone to share what they saw in the clone.]**

---

## SLIDE 15 | When LFS fits and when it does not
**[2 min]**

Be honest about what LFS is.

**[POINT to the left card]**
It fits datasets and assets that change rarely.
Files from tens of megabytes up to a few gigabytes.
And teams already on GitHub or GitLab.

**[POINT to the right card]**
Consider alternatives for very large or fast-changing data. Object storage is often better.
Consider a data versioning tool such as DVC when you need lineage.
And keep data in a secure store when it must not leave it.

**[POINT to the bottom bar]**
LFS versions the files.
It does not record how the data was produced.

**[Part III complete. Move to Part IV.]**

---

# PART IV: STRUCTURING DATA PROJECTS (25 MIN)

---

## SLIDE 16 | Anatomy of a data project
**[7 min]**

Good tooling needs good structure.
Here is a template.

**[POINT to the tree, top to bottom]**
A data folder, split into raw and processed.
A notebooks folder.
A src folder for reusable code.
A configs folder.
A tests folder.
A README.
A requirements file.
And the .gitignore and .gitattributes files we just discussed.

**[POINT to the right cards]**
Three principles.

Separate raw from processed.
Raw data is never edited.
Processed data is rebuilt by code.

Keep exploration apart from reusable code.
Notebooks explore. Src holds the stable logic.

Document the entry points.
The README explains setup, data sources and how to run the project.

**[SAY]** This is a starting template, not a rule.
Adapt the names to your team, but keep the separations.

**[ASK]** How is your current project laid out? Is raw data mixed with processed data?

---

## SLIDE 17 | What goes where, and how Git treats it
**[7 min]**

This table connects structure to Git behaviour.

**[Walk down the table]**
data slash raw holds original source data, never edited.
Track it with LFS, or keep it outside Git.

data slash processed holds cleaned and derived outputs.
Ignore it, and regenerate it from code.

notebooks hold exploration and reporting.
Track them, with outputs handled. Part five shows how.

src holds reusable ETL and utility functions.
Track it, and review changes in pull requests.

configs hold paths, parameters and settings.
Track them, but never include secrets.

tests hold checks for code and data.
Track them.

**[PAUSE]**

Notice the logic.
Anything you can rebuild, you ignore.
Anything that defines how you rebuild it, you track.

---

## SLIDE 18 | Notebooks and scripts: choose the right tool
**[5 min]**

A common failure.
A production job that exists only in a notebook.

**[POINT to the left card]**
Notebooks are best for exploration and quick experiments.
For visualisation and storytelling.
And for sharing findings with stakeholders.

**[POINT to the right card]**
Scripts and modules are best for reusable ETL steps.
For scheduled and automated jobs.
And for code that is tested and reviewed.

**[POINT to the bottom bar]**
The pattern.
Prototype in a notebook.
Then move stable logic into src, and import it back into the notebook.

**[ASK]** Does anyone run a scheduled job directly from a notebook today?

---

## SLIDE 19 | Configs, secrets and environments
**[6 min]**

Three things that decide whether a teammate can run your project.

**[POINT to Configs]**
Track parameters and paths in config files.
One file per environment.
And no hard-coded paths in code.

**[POINT to Secrets]**
Keep secrets in a .env file.
Ignore the .env file in Git.
And commit a .env.example template, so others know which values to set.

**[POINT to Environments]**
Pin versions in requirements.txt, or use environment.yml.
So anyone can rebuild the same setup anywhere.

**[POINT to the bottom bar]**
A good test.
A new teammate should clone, install, edit one config file, and run.

**[WARN]** If a secret is ever committed, rotate it.
Deleting the file later does not remove it from history.

**[Part IV complete. Move to Part V.]**

---

# PART V: NOTEBOOK VERSIONING WITH NBDIME (25 MIN, HANDS-ON DEMO)

---

## SLIDE 20 | Why plain Git diffs fail on notebooks
**[4 min]**

Now we fix the problem we saw in Part two.

**[DEMO: Show a real git diff of an .ipynb file that has a plot output.]**
This is what a plain diff looks like.
Lines of JSON.
Long strings of base64 for the image.
Nobody can review this.

**[POINT to the table]**
Look at the comparison.
Unit of change. Plain Git sees lines of JSON. nbdime sees notebook cells.
Outputs. Plain Git shows raw text, including base64 images. nbdime renders them side by side.
Execution counts. Plain Git shows them as real changes. nbdime can ignore them.
Review effort. High and error prone, versus low and focused.

**[POINT to the bottom bar]**
nbdime understands notebook structure.
So reviews focus on what really changed.

---

## SLIDE 21 | The nbdime toolkit
**[3 min]**

nbdime is a small toolkit.

**[Walk down the table]**
nbdiff compares two notebooks in the terminal.
nbdiff-web opens a rich visual diff in the browser, with rendered outputs.
nbmerge does a three-way merge in the terminal.
nbmerge-web resolves conflicts visually in the browser.
And nbdime config-git plugs nbdime into Git's diff and merge. You run it once.

**[SAY]** The browser tools are the most useful for review.
We will use them in a moment.

---

## SLIDE 22 | Demo: install and use nbdime
**[6 min]**

Let me show it live.
**[DEMO: Use a sample notebook where you changed one cell and re-ran another before the session.]**

**[Line 1]**
pip install nbdime.
That installs the toolkit.

**[Line 2]**
nbdime config-git, with enable and global.
This lets Git use nbdime for notebooks, in every repository on this machine.

**[Line 3]**
git diff on the notebook.
Now it is a cell-aware diff in the terminal. Compare it with what we saw a few minutes ago.

**[Line 4]**
nbdiff-web with two notebook files.
This opens a rich diff in the browser.
See the changed code cell.
See the changed output, side by side.

**[PAUSE]**

**[ASK]** What differences can you spot immediately that you could not before?

---

## SLIDE 23 | Merging notebooks and resolving conflicts
**[6 min]**

Diffs are half the story.
The other half is merging.

**[Read the five steps]**
One. Two branches edit the same notebook. Both change cells in the same file.
Two. You run git merge, and Git reports a conflict.
Three. You open the merge tool. Use nbmerge-web, or git mergetool.
Four. You choose per cell. Keep either version or combine them.
Five. Save, then git add, then commit the merge.

**[DEMO: Resolve a prepared conflict live, if time allows.]**

**[POINT to the bottom bar]**
The key point.
Conflicts are resolved per cell, never inside raw JSON.

---

## SLIDE 24 | Notebook hygiene habits
**[6 min]**

Tools help, habits help more.

**[POINT to each card]**
Restart and run all before you commit.
It confirms the notebook runs from top to bottom.

Handle bulky outputs.
Clear heavy outputs, or strip them with a pre-commit tool.

Number and name your notebooks.
Names like zero one underscore eda make order and purpose clear.

One owner at a time.
Avoid two people editing the same notebook in parallel.

**[SAY]** The last habit prevents most conflicts.
The tool is for when prevention fails.

**[Part V complete. Move to Part VI.]**

---

# PART VI: BRANCHING STRATEGIES (20 MIN)

---

## SLIDE 25 | Why branch: isolate work, keep main stable
**[4 min]**

Let us finish with collaboration.

**[POINT to the diagram, trace with your hand]**
This line is main.
Here we create a branch from main.
The branch is called feature slash clean-sales-data.
We make three commits there, away from main.
Then we merge back through a pull request, review and merge.

**[POINT to the three cards]**
Three benefits.
Isolation. Experiments cannot break the shared code.
A stable main. Main always holds reviewed, working work.
And parallel progress. Several people change the project at once.

**[ASK]** What happens today when two people edit the same pipeline at the same time?

---

## SLIDE 26 | Branching commands
**[4 min]**

The commands.
**[DEMO: Create a branch, commit, switch back to main and merge. Keep it short.]**

git branch lists the branches.
git switch with dash c creates a branch and switches to it. Use it to start new work.
git switch main returns to main. Update it before merging.
git merge brings a branch into the current one.
git branch with dash d deletes a merged branch. Tidy up.
And git push with dash u origin and the branch name publishes it, so you can open a pull request.

**[SAY]** Older tutorials use git checkout.
Git switch is the clearer modern command for the same job.

---

## SLIDE 27 | The feature-branch workflow
**[4 min]**

Six steps. Every feature, every time.

One. Branch from main. Start from the latest main.
Two. Commit small changes. One logical change per commit.
Three. Push the branch. Share it with the remote.
Four. Open a pull request. Describe what and why.
Five. Review and fix. Teammates comment, you update.
Six. Merge and delete. Merge to main, and remove the branch.

**[POINT to the bottom bar]**
Keep branches short lived.
Days, not weeks.
The longer a branch lives, the harder the merge.

---

## SLIDE 28 | Choosing a branching strategy
**[4 min]**

There are several strategies.
You should recognise the names.

**[Walk down the table]**
Feature branching. One short branch per feature, merged by pull request.
This suits most data teams, and it is what we use today.

Trunk-based development. Very small changes merged to main daily.
It suits mature teams with strong automation.

Git Flow. Long-lived develop, release and hotfix branches.
It suits fixed release schedules.

**[POINT to the naming card]**
And name your branches consistently.
Feature slash clean-sales-data.
Fix slash date-parsing.
Exp slash lstm-forecast, for an experiment.

The prefix tells everyone the purpose at a glance.

---

## SLIDE 29 | Branching practices for data work
**[4 min]**

Four practices that matter most for data teams.

One. One branch per feature or experiment.
It keeps results traceable to a change.

Two. Sync with main often.
Pull main into your branch to avoid large conflicts.

Three. Avoid parallel notebook edits.
Agree who edits which notebook.

Four. Protect main.
Require pull requests and reviews before anything merges.

**[POINT to the bottom bar]**
Small branches and frequent merges prevent painful conflicts.

**[Part VI complete. Move to the close.]**

---

# PART VII: DOUBT RESOLUTION AND Q AND A (10 MIN)

---

## SLIDE 30 | Key takeaways
**[3 min]**

Let me recap in five lines.

**[POINT to each card]**
Git essentials. Small, specific commits and a clean .gitignore.
Git LFS. Pointers in Git, large content in LFS storage.
Project structure. Separate raw data, notebooks, source and configs.
nbdime. Cell-aware diffs and merges for notebooks.
Branching. Short feature branches, reviewed before merging.

**[POINT to the bottom right box]**
What is coming next.
The upcoming module is Application Deployment.
Everything we did today, the structure, the branches and the clean history, makes deployment easier.

---

## SLIDE 31 | Questions and discussion
**[7 min or more]**

Thank you.
The floor is open.

Questions on commands, LFS quotas and large files.
Questions on notebooks and nbdime.
Questions on branching and conflicts.

If you want to go further, the additional readings are listed in the session agenda.

**[Closing line]**
Remember.
Version control is a habit.
Small commits, clear structure and short branches keep data projects healthy.

Thank you, everyone.

---

# TRAINER QUICK NOTES

**Timing checkpoints**

| Elapsed | You should be at |
|---|---|
| 0:05 | End of Part I (Slide 4) |
| 0:25 | End of Part II (Slide 9) |
| 0:50 | End of Part III (Slide 15) |
| 1:15 | End of Part IV (Slide 19) |
| 1:40 | End of Part V (Slide 24) |
| 2:00 | End of Part VI (Slide 29) |
| 2:10 | Close (Slide 31) |

The agenda adds up to 130 minutes.
If your slot is a strict 2 hours, trim about 10 minutes using the list below.

**If you are running late, shorten these first**

1. Slide 6. Demo only status, add and commit.
2. Slide 17. Read only the first three rows.
3. Slide 28. Mention Git Flow and trunk-based in one sentence each.

**If you are running early, extend these**

1. Slide 14. Give learners more time and review their results.
2. Slide 23. Resolve a second conflict live.
3. Slide 26. Practise creating and merging a branch together.

**Prepare before the session**

- A small demo repository with a few commits.
- A sample notebook with one changed cell and one changed output, committed on two branches.
- A second branch that edits the same notebook, ready to create a merge conflict.
- A GitHub or GitLab test repository with Git LFS enabled.
- nbdime installed and the browser tools tested once.
- A sample CSV of a few megabytes for the LFS exercise.
