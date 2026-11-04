# Trainer Guide: Operator SDK and OLM

**Deck:** Operator_SDK_and_OLM.pptx (23 slides)
**Audience:** Developers, DevOps/platform engineers and SREs who know basic Kubernetes (Pods, Deployments, Services, kubectl)
**Suggested duration:** 3 to 3.5 hours with demos and hands-on (about 90 minutes of talk if you skip labs)
**Format:** Presentation, live demo, short hands-on lab

---

## 0. Before You Start

### Learning objectives
By the end of the session, participants can:

1. Explain what an Operator is and how the reconcile loop works.
2. Describe the four building blocks of an Operator (CRD, CR, controller, RBAC and Deployment).
3. Choose between Go, Ansible and Helm for building an Operator.
4. Install `operator-sdk` and walk through the typical Go workflow.
5. Explain what OLM is, its architecture, and its key resources.
6. Trace an Operator installation from Subscription to running Pod.
7. Explain how OLM v1 differs from classic OLM (v0).
8. Troubleshoot common install and subscription problems.

### Prerequisites to confirm with the audience
- Comfortable with `kubectl`, YAML, Pods, Deployments, Services, RBAC basics.
- Know what a CRD is (if not, spend 5 minutes on it at slide 4).
- For the hands-on: access to a Kubernetes or OpenShift cluster with **cluster-admin**, Linux or macOS terminal, internet access to GitHub.

### Trainer preparation checklist
- [ ] Cluster ready (kind, minikube, OpenShift Local or a shared training cluster).
- [ ] `kubectl` works and you have cluster-admin.
- [ ] Test the install commands from slides 11 and 12 on your own machine beforehand.
- [ ] Decide: is OLM already installed on the cluster? (On OpenShift it is built in. `operator-sdk olm install` is for vanilla Kubernetes.)
- [ ] Have a registry account ready if you plan to demo `make bundle` with a real image.
- [ ] Keep a terminal font size large enough for the back row.

### Timing plan

| Segment | Slides | Time |
| --- | --- | --- |
| Welcome and agenda | 1 to 2 | 5 min |
| Operator concepts | 3 to 6 | 25 min |
| Operator SDK overview and build types | 7 to 9 | 20 min |
| Go workflow walkthrough | 10 | 15 min |
| Install operator-sdk (demo) | 11 to 12 | 15 min |
| Break | | 10 min |
| OLM concepts and architecture | 13 to 16 | 30 min |
| Bundles, catalogs, install flow | 17 to 18 | 20 min |
| OLM v1 | 19 | 10 min |
| SDK and OLM together | 20 | 10 min |
| Hands-on lab | 21 | 25 min |
| Troubleshooting | 22 | 10 min |
| Takeaways and Q&A | 23 | 10 min |

---

## Slide 1: Title. Operator SDK and OLM

**Time:** 2 min

### What to say
"Today's topic has two halves. The first half is about **building** Operators with the Operator SDK. The second half is about **running and managing** them with the Operator Lifecycle Manager, or OLM."

Point at the title graphic: the SDK covers **build** and **package**; OLM covers **install** and **update**. The bundle is the handoff between the two.

### Key message
SDK = the factory. OLM = the app store plus the update manager.

### Engagement
Ask: "Who has already installed an Operator, for example from OperatorHub, Prometheus Operator or a database operator?" This tells you the group's experience level.

---

## Slide 2: Agenda

**Time:** 3 min

### Talking points
- **Part 1: Operator SDK.** What an Operator is, what the SDK does, build types and workflow, installing the CLI.
- **Part 2: OLM.** Why it exists, architecture and resources, bundles and catalogs, install flow, OLM v1, and how SDK and OLM fit together.
- Mention there is a hands-on near the end and a troubleshooting table to keep as reference.

### Trainer tip
Set expectations: "We will not write a full controller in Go today. The focus is understanding the concepts, the tooling and the lifecycle."

---

## Slide 3: Section divider. Part 1: Operator SDK

**Time:** 1 min

"From custom controller to a packaged Operator." This is the journey: start with a controller, end with something you can ship and install.

---

## Slide 4: What is an Operator

**Time:** 8 min

### Concept in plain words
Kubernetes already knows how to run stateless things (Deployments, ReplicaSets). It does not know how to run **a PostgreSQL cluster with replication, backups and failover**. A human operator (a DBA or SRE) knows those steps. An **Operator** is that human knowledge written as software.

> An Operator = **Custom Resource Definition (CRD)** + **custom controller**.

### Walk through the diagram (left to right)
1. **Custom Resource**: the user writes YAML saying what they want, such as "I want a 3-node Memcached." This is the *desired state*.
2. **Operator controller**: code that reads the CR and decides what to do. This is the *reconcile logic*.
3. **Cluster resources**: the controller creates and manages Deployments, Services, ConfigMaps and so on.

### Three benefits on the slide
| Benefit | Explain it as |
| --- | --- |
| Automate day-2 work | Day 1 is install. Day 2 is everything after: scaling, backups, upgrades, recovery. Operators automate day 2. |
| Declarative API | Users say **what**, not **how**. Same model as Deployments. |
| Self-healing | If someone deletes a Pod or edits a resource, the controller notices and puts it back. |

### Analogy
A thermostat. You set the target temperature (desired state). The thermostat constantly measures the room (actual state) and turns heating on or off to match. You never say "turn the heater on for 10 minutes."

### Check for understanding
"If a Pod created by an Operator is deleted manually, what happens and why?" (Answer: the controller sees actual state differs from desired and recreates it.)

### Common confusion
- An Operator is **not** a special Kubernetes object. It is a pattern: a controller that watches custom resources.
- An Operator usually runs as a normal Deployment inside the cluster.

---

## Slide 5: Anatomy of an Operator

**Time:** 6 min

### The four building blocks

1. **CRD (Custom Resource Definition)**
   Teaches the API server a new type. After applying a `Memcached` CRD, `kubectl get memcached` works. Think of it as the *schema* or *class*.

2. **Custom Resource (CR)**
   An *instance* of that type, such as `memcached-sample` with `size: 3`. It has `spec` (desired state, written by the user) and `status` (observed state, written by the controller).

3. **Controller**
   The code. It watches CRs and related objects and runs reconcile logic to make reality match.

4. **RBAC and Deployment**
   The controller is just a Pod. It needs a ServiceAccount plus Roles/ClusterRoles to create the resources it manages, and a Deployment to run it.

### Teaching tip
Use a class and object analogy for CRD and CR. Then ask: "Which of the four does the user write? Which does the developer write?" (User writes CR. Developer writes CRD types, controller and RBAC rules, and the SDK generates much of the YAML.)

### Common confusion
Learners often think the CRD contains logic. It does not. The CRD is only the data shape. All logic lives in the controller.

---

## Slide 6: The reconcile loop

**Time:** 8 min

This is the single most important concept in Part 1. Slow down here.

### The four steps
| Step | What happens |
| --- | --- |
| **Watch** | Controller subscribes to events on the CR and on objects it owns (for example the Deployment it created). |
| **Compare** | Reads desired state (CR spec) and actual state (live cluster objects). |
| **Act** | Creates, updates or deletes resources to close the gap. |
| **Report** | Writes observed information back into the CR `status`. |

Then it **repeats**, forever. Point at the "repeat" arrow.

### Key ideas to stress
- **Level-based, not edge-based.** The controller does not care *what event* happened. It only cares "does reality match the spec right now?" That makes it robust if events are missed or the controller restarts.
- **Idempotent.** Running reconcile ten times with no change must produce the same result and no side effects. Mention this as a design rule for people who will write controllers.
- **Owned objects.** When the controller sets an owner reference, deleting the CR garbage-collects the children automatically.

### Mini example to narrate
"CR says `size: 3`. Controller finds 2 Pods. Act: create 1. Report: status lists 3 Pod names. Someone later edits the CR to `size: 5`. Watch fires. Compare finds 3 vs 5. Act: create 2."

### Check for understanding
"What happens if the controller crashes for 5 minutes while someone changes the CR?" (When it restarts, it reconciles and catches up. No lost events problem because it compares states.)

---

## Slide 7: What is the Operator SDK

**Time:** 6 min

### Definition
A framework from the **Operator Framework** project that makes building, testing and packaging Operators faster and less error-prone. Without it you would write a lot of boilerplate by hand.

### Three parts
1. **CLI**: the `operator-sdk` command. Scaffolds projects, creates APIs, generates and validates bundles.
2. **Libraries**: for Go, built on **Kubebuilder** and **controller-runtime**. These give you the client, caches, watches, and manager.
3. **Tooling**: bundle generation, `bundle validate`, **scorecard** tests, and OLM helper commands.

### Trainer note
Mention that Operator SDK Go projects share their layout with Kubebuilder, so knowledge transfers. Do not go deep into controller-runtime internals unless asked.

### Context
Operator Framework has two pillars: **Operator SDK** (build) and **OLM** (run). That is the exact split of today's session.

---

## Slide 8: Three ways to build

**Time:** 8 min

| Type | Built on | Best when |
| --- | --- | --- |
| **Go** | controller-runtime, Kubebuilder | You need full control, complex logic, or performance |
| **Ansible** | Ansible roles and playbooks | You already have Ansible automation |
| **Helm** | An existing Helm chart | You want to wrap a chart quickly |

### How to explain each
- **Go:** Maximum power. You write reconcile code. Needed for complex lifecycle logic such as database failover or custom backup flows. Steeper learning curve.
- **Ansible:** Reuse existing playbooks. The operator runs Ansible when the CR changes. Good for ops teams with Ansible skills and little Go experience.
- **Helm:** Fastest path. You point the SDK at a chart and it generates an Operator that installs and upgrades releases based on a CR. Limited to what a chart can express.

### Capability levels (note under table)
Five levels: **1 Basic Install, 2 Seamless Upgrades, 3 Full Lifecycle, 4 Deep Insights, 5 Auto Pilot.**
Helm operators reach the early levels. Go and Ansible can cover the full range.

### Discussion question
"Your team already has a Helm chart and wants an Operator by next week. Which type? What might force you to move to Go later?" (Helm for speed. Move to Go if you need backups, failover, or custom day-2 logic that charts cannot express.)

### Decision shortcut for learners
Existing chart, simple needs: Helm. Existing playbooks: Ansible. New, complex, performance-sensitive: Go.

---

## Slide 9: What the SDK does

**Time:** 6 min

Six jobs in a pipeline. Walk numbers 1 to 6:

| # | Job | Detail |
| --- | --- | --- |
| 1 | **Scaffold** | `init` and `create api` produce the project layout, Makefile, Dockerfile and sample files. |
| 2 | **Generate** | From your Go types it generates the CRD YAML, RBAC manifests and deepcopy code (`make manifests generate`). |
| 3 | **Build and run** | Run the controller locally against a cluster for fast dev, or build an image and deploy. |
| 4 | **Package** | `make bundle` creates an OLM bundle from your manifests. |
| 5 | **Validate and test** | `operator-sdk bundle validate` checks bundle correctness. **Scorecard** runs functional tests. |
| 6 | **Deploy and clean up** | `operator-sdk run bundle` installs through OLM. `operator-sdk cleanup` removes it. |

### Key message
"Steps 1 to 3 are the inner dev loop. Steps 4 to 6 connect to OLM. That is the bridge to Part 2."

---

## Slide 10: Typical Go workflow

**Time:** 15 min (walk the commands, optionally demo `init` and `create api`)

### Flow: init → create api → make manifests → make bundle → run bundle → cleanup

Walk the commands one at a time:

```bash
operator-sdk init --domain example.com \
  --repo github.com/example/memcached-operator
```
Creates the project. `--domain` becomes the API group suffix. `--repo` sets the Go module path.

```bash
operator-sdk create api --group cache --version v1alpha1 \
  --kind Memcached --resource --controller
```
Creates the Go types for the CRD (`Memcached`) and a controller skeleton. Result: API group `cache.example.com/v1alpha1`, kind `Memcached`.

```bash
make manifests generate
```
Regenerates CRD and RBAC YAML plus deepcopy code from your Go types. Run it **every time** you change the types.

```bash
make bundle IMG=<registry>/memcached-operator:v0.0.1
```
Produces the `bundle/` directory (CSV, CRDs, metadata). `IMG` is the operator image reference that goes into the CSV.

```bash
operator-sdk bundle validate ./bundle
```
Checks the bundle structure against OLM rules before you waste time deploying.

```bash
operator-sdk run bundle <registry>/memcached-bundle:v0.0.1
```
Installs the operator on the cluster **through OLM**, so you test the real install path. Requires OLM on the cluster.

```bash
operator-sdk cleanup memcached-operator
```
Removes what `run bundle` created.

### Trainer notes
- The example uses a **Memcached** operator, a classic SDK tutorial. Replace registry placeholders with your own.
- Explain `v1alpha1`: an early API version. Mature APIs move to `v1beta1` and `v1`.
- If you demo live, run `init` and `create api` then show the folder tree: `api/`, `internal/controller/` (or `controllers/` in older versions), `config/`, `bundle/`, `Makefile`. Point out where the reconcile function lives.
- Folder names can differ slightly across SDK versions. Check against the version you demo.

### Common mistakes
- Forgetting `make manifests generate` after editing types, so the CRD in the cluster is stale.
- Using a bundle image that is not pushed to a registry the cluster can reach.
- Running `run bundle` on a cluster without OLM.

---

## Slide 11: Install operator-sdk. Variables

**Time:** 7 min

### Purpose
These shell variables make the download commands portable across machines. Skip this section if everyone already has the CLI installed.

### Walk through
| Variable | Explanation |
| --- | --- |
| `ARCH` | Maps `x86_64` to `amd64` and `aarch64` to `arm64`. Release binaries use Go-style arch names, while `uname -m` prints different ones. Other values pass through unchanged. |
| `OS` | Lowercase kernel name, such as `linux` or `darwin`. |
| `OPERATOR_SDK_VERSION` | Pins the release, here **v1.41.1**. Pinning gives reproducible installs. |
| `OPERATOR_SDK_DL_URL` | Builds the base GitHub release URL from the version. |

### Teaching points
- Explain `case ... esac` briefly for non-shell experts.
- Line breaks with `\` are for the slide only. In a terminal, they work, or you can paste on one line.
- Tell learners they can change the version later, but should keep labs consistent so everyone sees the same behavior.

---

## Slide 12: Install. Download and verify

**Time:** 8 min (demo live)

### Steps
1. **Download** the binary and `checksums.txt` with `curl -fLO`.
   - `-f` fail on HTTP errors, `-L` follow redirects, `-O` save with remote file name.
2. **Verify** with `grep ... | sha256sum -c -`. Confirms the file is intact and matches the published hash.
3. **Make executable and move to PATH** with `chmod +x` then `mv` to `/usr/local/bin/operator-sdk`.
4. **`hash -r`** clears the shell's cached command locations, so the new binary is found immediately.
5. **`operator-sdk version`** confirms it works.

### Why verify the checksum?
"You are about to run a downloaded binary with access to your cluster credentials. The checksum proves the file was not corrupted or tampered with in transit."

### Note on the slide
Writing to `/usr/local/bin` needs root. Use `sudo` if you are not root, or move the binary to any folder already on your PATH.

### Live demo tip
Run the commands for real. If the download is slow on shared Wi-Fi, pre-download the binary and show only the verify and move steps.

### Link to troubleshooting
Mention now that slide 22 lists the failures learners are likely to hit here: checksum mismatch, permission denied, command not found, exec format error.

---

## Slide 13: Section divider. Part 2: Operator Lifecycle Manager

**Time:** 3 min (include a short recap and a break if you are at the midpoint)

### Bridge from Part 1
"We can now build and package an Operator. But how does an admin on a cluster discover it, install it safely, keep it updated, and handle its dependencies? That is OLM."

### Quick recap questions
- What are the four parts of an Operator?
- What are the four steps of the reconcile loop?

---

## Slide 14: What is OLM

**Time:** 6 min

### Definition
> OLM is the **package manager for Operators**. It installs them, keeps them updated, and manages dependencies and permissions.

### Three verbs
1. **Discover:** browse operators published in catalogs (for example OperatorHub).
2. **Install:** declare what you want. OLM creates the CRDs, RBAC and Deployment.
3. **Update:** follow channels, then approve or automate upgrades.

### Analogy
`apt`, `yum` or an app store for the cluster. Installing an operator by hand means applying CRDs, RBAC and Deployments yourself, and you would have to track versions and upgrades manually. OLM does that from one declaration.

### Why OLM matters (benefits to state)
- Consistent, repeatable installs.
- Controlled upgrades with version paths.
- Dependency resolution between operators.
- Permission scoping (which namespaces an operator can act in).
- Prevents two operators from fighting over the same CRD.

### Check
"What problem would you face installing 20 operators by raw YAML?" (Version drift, no upgrade path, permissions scattered, hard to audit.)

---

## Slide 15: OLM architecture (v0)

**Time:** 8 min

### Two cooperating operators in the `olm` namespace

**Catalog Operator** (the *resolver*)
- Watches **CatalogSource** and **Subscription** objects.
- Resolves dependencies and chooses the right bundle version.
- Creates **InstallPlans**.

**OLM Operator** (the *installer*)
- Watches **ClusterServiceVersions (CSVs)**.
- Creates the Deployments, RBAC and CRDs the CSV describes.
- Reports operator status.

The **InstallPlan** is the handoff between them.

### Memorization hook
"Catalog Operator decides **what** to install. OLM Operator **does** the install."

### Teaching point
OLM is itself implemented as Operators, using the same pattern learners just studied. This is a good "aha" moment.

### Trainer note
On OpenShift these run in `openshift-operator-lifecycle-manager`, and catalogs in `openshift-marketplace`. On upstream Kubernetes, `olm` is the namespace shown in the deck. Mention this if your audience uses OpenShift.

---

## Slide 16: Key OLM resources

**Time:** 10 min

Five objects that learners will use constantly. Explain each, then link them in a story.

| Resource | Purpose | Plain-words explanation |
| --- | --- | --- |
| **ClusterServiceVersion (CSV)** | Describes one operator version: deployment, permissions, owned CRDs | The "package manifest" or "app descriptor" for one version. Also shows install status. |
| **CatalogSource** | Points to a catalog of operator bundles | The "repository URL." |
| **Subscription** | Declares the operator, channel and catalog you want | "I want package X, from channel Y, from catalog Z." |
| **InstallPlan** | List of resources to install or update; may need approval | The "shopping list" OLM will apply. |
| **OperatorGroup** | Sets the namespaces an operator watches and its RBAC scope | Defines the operator's **target namespaces**. |

### Story to link them
"An admin creates a **Subscription** pointing at a **CatalogSource**. OLM generates an **InstallPlan** listing everything to apply. Once approved, it creates the **CSV** that describes the operator version. The **OperatorGroup** in that namespace controls where the operator is allowed to watch."

### OperatorGroup deep dive (common stumbling block)
Operators declare which install modes they support:
- **OwnNamespace:** watches only the namespace it is installed in.
- **SingleNamespace:** watches one other namespace.
- **MultiNamespace:** watches a set of namespaces.
- **AllNamespaces:** watches the whole cluster.

The OperatorGroup must match a mode the operator supports. Only **one OperatorGroup per namespace** is allowed. Mismatches are a frequent cause of failed installs.

### Hands-on suggestion
Show real objects: `kubectl get csv,subscription,installplan,catalogsource,operatorgroup -A`. Seeing real output makes the table stick.

---

## Slide 17: Bundles and catalogs

**Time:** 10 min

### Four terms
| Term | Meaning |
| --- | --- |
| **Bundle** | One operator **version**: CSV, CRDs and metadata packaged in a container image. |
| **Catalog** | Bundles indexed as a **file-based catalog (FBC)** with channels. |
| **CatalogSource** | Cluster object pointing at the catalog image. |
| **Subscription** | Your request for a package and channel. |

### Flow of ideas
Bundle (one version) → many bundles indexed into a **Catalog** → catalog image referenced by a **CatalogSource** → user creates a **Subscription** against it.

### Channels
A channel is a stream of updates, for example `alpha`, `stable`, `fast`. A subscription follows one channel and receives updates published there.

### Upgrade paths (text under the diagram)
Each bundle can declare `replaces` or `skips`:
- **replaces:** "this version upgrades directly from version X."
- **skips:** "also allow upgrading from these versions, skipping the intermediates."

This gives OLM the **valid upgrade graph** within a channel, so it never jumps to an unsafe version.

### Analogy
Bundle = one app release. Catalog = the store shelf listing all releases. Channel = release track (stable or beta). `replaces` = "update from version N-1."

### Trainer note
FBC is the current catalog format (declarative YAML or JSON). If learners have older material that mentions SQLite-based index images, tell them that format is the legacy approach and FBC is the direction.

---

## Slide 18: How an install flows

**Time:** 10 min (best slide to draw on a whiteboard in parallel)

### The five steps
1. **Create OperatorGroup and Subscription.** The admin's only manual action.
2. **Catalog Operator resolves the bundle.** Looks in the catalog, chooses the version in the channel, and resolves dependencies.
3. **InstallPlan is created and approved.**
4. **OLM Operator installs the CSV.** Creates CRDs, RBAC and the operator Deployment.
5. **Operator Pod runs and CRDs are served.** Users can now create custom resources.

### Approval modes (box on slide)
- **Automatic:** installs immediately.
- **Manual:** waits until someone approves the InstallPlan.

Where used: Production clusters often use **Manual** so upgrades happen in a change window. Dev clusters use **Automatic**.

### Show the approval command (optional)
```bash
kubectl patch installplan <name> -n <namespace> \
  --type merge -p '{"spec":{"approved":true}}'
```

### Check for understanding
"A Subscription was created an hour ago but nothing is running. What are the first two things you check?" (InstallPlan awaiting approval, and CatalogSource not ready. This ties to the troubleshooting slide.)

### Whiteboard idea
Draw the five boxes and ask learners to name which component (admin, Catalog Operator, OLM Operator) owns each step.

---

## Slide 19: OLM v1, the next generation

**Time:** 10 min

### Comparison table
| Area | OLM v0 (classic) | OLM v1 |
| --- | --- | --- |
| Components | OLM Operator, Catalog Operator | **operator-controller**, **catalogd** |
| User APIs | Subscription, InstallPlan, CSV, OperatorGroup | **ClusterExtension**, **ClusterCatalog** |
| Scope | Namespaced or cluster-wide operators | Cluster-scoped objects |
| Model | Several objects to manage | One declarative object, GitOps friendly |

### How to explain
- **Why a redesign?** v0 asks users to manage several interacting objects and carries complexity in OperatorGroups and install modes. v1 simplifies to a single declarative object (`ClusterExtension`) describing what you want installed, which suits GitOps workflows.
- **catalogd** serves catalog content. **operator-controller** reconciles ClusterExtensions into installed content.
- Both worlds currently exist. Many production clusters still run v0.

### Important status note from the deck
OLM v1 is GA in **OpenShift 4.22**. Verify the status for your specific platform and version before the session, because availability differs across upstream Kubernetes and OpenShift releases.

### Critical clarification (say it explicitly)
The `operator-sdk olm ...` commands in this deck set up **classic OLM (v0)**. They do not install OLM v1.

### Discussion question
"Why might a GitOps team prefer one ClusterExtension over a Subscription plus OperatorGroup plus InstallPlan?" (Fewer moving parts, simpler to put in Git, less implicit state.)

---

## Slide 20: Operator SDK and OLM together

**Time:** 10 min

### The pipeline (top row)
**Develop** (Operator SDK) → **Bundle** (`make bundle`) → **Validate** (`bundle validate`, scorecard) → **Run** (`run bundle` via OLM) → **Publish** (add to a catalog)

### Command reference on the slide
| Command | Purpose |
| --- | --- |
| `operator-sdk olm install` | Install classic OLM into the cluster |
| `operator-sdk olm status` | Check OLM components |
| `operator-sdk run bundle <bundle-image>` | Install your operator via OLM for testing |
| `operator-sdk cleanup <operator-name>` | Remove what `run bundle` installed |
| `operator-sdk olm uninstall` | Remove OLM from the cluster |

### Summary sentence
> "SDK builds and packages. OLM runs and updates."

### Add context
- `run bundle` is a **dev and test** tool. It builds temporary catalog objects so you can try a bundle quickly. For real distribution you add your bundle to a catalog (FBC) that clusters subscribe to.
- On OpenShift, OLM is already present, so `olm install` is typically not needed.

### Scorecard (mention briefly)
`operator-sdk scorecard` runs predefined tests against your bundle on a cluster, checking that the CR example applies, spec and status behave as expected, and so on.

---

## Slide 21: Hands-on. Verify and install OLM

**Time:** 25 min

### Lab commands
```bash
operator-sdk version                 # should print v1.41.1
operator-sdk olm install             # deploys OLM into the cluster
operator-sdk olm status              # checks the components
kubectl get pods -n olm              # OLM Pods should be Running
kubectl get csv -A                   # CSVs appear once an operator is installed
```

### Expected results
- `version` prints the pinned version.
- `olm install` creates CRDs and deployments, then reports success.
- `olm status` lists components as ready.
- `kubectl get pods -n olm` shows `olm-operator`, `catalog-operator`, `packageserver`, and a catalog pod such as `operatorhubio-catalog` in `Running` state (names can vary by version).
- `kubectl get csv -A` may initially show only the `packageserver` CSV. More appear as you install operators.

### Suggested lab flow
1. Everyone verifies their SDK version (5 min).
2. Install OLM and check status (5 min).
3. Inspect the OLM namespace and CRDs: `kubectl get crd | grep operators.coreos.com` (5 min).
4. **Stretch goal:** create an OperatorGroup and Subscription for a small operator from OperatorHub.io, approve the InstallPlan, and watch the CSV reach `Succeeded` (10 min).
5. **Cleanup:** `operator-sdk olm uninstall` when finished.

### Facilitation tips
- Walk the room. Most failures are permissions (not cluster-admin) or OLM already installed.
- Have a "buddy" rule: someone who finishes helps a neighbor.
- Ask learners to read the status output aloud instead of just saying "it worked."

### Safety note
Do this on a lab cluster. Installing OLM creates cluster-wide CRDs and should not be done on a shared production cluster without approval.

---

## Slide 22: Troubleshooting

**Time:** 10 min

Use as a reference, not a lecture. Walk each row and ask the group for the cause before revealing it.

| Symptom | Likely cause | First check | Trainer explanation |
| --- | --- | --- | --- |
| Checksum check fails | Incomplete or wrong file | Re-run `curl`; check OS and ARCH | Partial download, wrong platform file, or mismatched filename in `checksums.txt`. Do not run the binary until it passes. |
| Permission denied on `mv` | No write access to `/usr/local/bin` | Use `sudo`, or a folder on PATH | Alternative: `~/bin` added to PATH. |
| `command not found` | Shell cache or PATH | `hash -r`; `which operator-sdk` | Binary is not on PATH or the shell remembers the old lookup. |
| Exec format error | Wrong ARCH for this machine | `echo $ARCH`; `uname -m` | Downloaded an arm64 binary on amd64 or vice versa. Common on Apple Silicon with Rosetta terminals. |
| `olm install` fails | Not cluster-admin, or OLM exists | `operator-sdk olm status` | OLM creates cluster-scoped objects, so it needs cluster-admin. If OLM is already present (as on OpenShift) the install conflicts. |
| Subscription never installs | InstallPlan awaiting approval, or catalog not ready | `kubectl get installplan,catalogsource -A` | Manual approval pending, catalog pod not running, or no OperatorGroup in the namespace. |

### Extra debugging commands to teach
```bash
kubectl get csv -n <ns>                         # phase: Pending, Installing, Succeeded, Failed
kubectl describe csv <name> -n <ns>             # events and failure reasons
kubectl get pods -n olm                         # OLM health
kubectl logs -n olm deploy/catalog-operator     # resolution problems
kubectl logs -n olm deploy/olm-operator         # install problems
kubectl get operatorgroup -n <ns>               # exactly one per namespace
```

### Debugging mindset to teach
Follow the install flow from slide 18 in order: Subscription → InstallPlan → CSV → Pod. Find the first step that did not complete.

---

## Slide 23: Key takeaways

**Time:** 10 min (including Q&A)

### Three takeaways
1. **An Operator is a CRD plus a controller** that reconciles desired and actual state.
2. **Operator SDK** scaffolds, builds, validates and packages Operators as **OLM bundles**.
3. **OLM** installs and updates Operators from catalogs. **OLM v1** simplifies this with `ClusterExtension`.

### Closing recap questions (quick fire)
1. Name the four building blocks of an Operator.
2. What are the steps of the reconcile loop?
3. Which SDK build type would you pick if you already have a Helm chart?
4. Which OLM component resolves dependencies? Which one installs?
5. What does a Subscription declare?
6. What does `replaces` do in a bundle?
7. What is the v1 equivalent of Subscription?

### Suggested answers
1. CRD, CR, controller, RBAC plus Deployment.
2. Watch, Compare, Act, Report.
3. Helm.
4. Catalog Operator resolves. OLM Operator installs.
5. Package, channel, catalog source (and approval mode).
6. Defines the direct upgrade path from a previous version.
7. ClusterExtension.

### Next steps for learners
- Follow the official Operator SDK Go tutorial (Memcached) end to end.
- Browse OperatorHub.io and install one operator via OLM.
- Experiment with the scorecard on their own bundle.
- Read the OLM v1 documentation for their platform.
- Try writing a CR and watching reconcile logs.

---

## Appendix A: Glossary

| Term | Definition |
| --- | --- |
| **Operator** | Application-specific controller that automates operations using CRs. |
| **CRD** | Schema that extends the Kubernetes API with a new resource type. |
| **CR** | An instance of a CRD, holding desired state in `spec` and observed state in `status`. |
| **Controller** | Loop that reconciles actual state toward desired state. |
| **Reconcile** | One pass of comparing desired vs actual and acting on the difference. |
| **Kubebuilder** | SDK for building Kubernetes APIs in Go; basis of Operator SDK's Go support. |
| **controller-runtime** | Go library providing manager, client, caches and watches for controllers. |
| **Bundle** | Image containing a CSV, CRDs and metadata for one operator version. |
| **Catalog (FBC)** | Indexed collection of bundles, packages and channels. |
| **Channel** | Update stream within a package (alpha, stable, and so on). |
| **CSV** | ClusterServiceVersion; describes one operator version and its requirements. |
| **Subscription** | Request to install and track a package from a channel. |
| **InstallPlan** | Calculated set of resources to install or update. |
| **OperatorGroup** | Defines the target namespaces and RBAC scope of operators in a namespace. |
| **Scorecard** | SDK tool that runs functional tests against a bundle. |
| **ClusterExtension** | OLM v1 object describing an installed extension (operator). |
| **ClusterCatalog** | OLM v1 object pointing at a catalog. |

---

## Appendix B: Common Learner Questions and Answers

**Q: Do I need OLM to run an Operator?**
No. You can deploy an operator with plain manifests or Helm. OLM adds lifecycle management: discovery, versioned upgrades, dependency handling and permission scoping.

**Q: Operator vs Helm, which one?**
They solve different problems. Helm templates and installs resources. An Operator runs continuously and manages day-2 operations. Operators can even wrap Helm charts (the Helm operator type).

**Q: Can one Operator manage multiple CRDs?**
Yes. A single operator can own several CRDs, each with its own controller.

**Q: Where does the reconcile code live in a Go project?**
In the controller file under the controllers directory (`internal/controller/` in recent scaffolds). The key function is `Reconcile`.

**Q: What if two operators own the same CRD?**
OLM prevents conflicting ownership of a CRD, which is one reason it exists.

**Q: Does `run bundle` publish my operator?**
No. It is a quick test installation. Publishing means adding the bundle to a catalog.

**Q: Is OLM v0 deprecated?**
Treat it as the established, widely deployed system with v1 as the direction of travel. Check your platform's current documentation for support timelines.

---

## Appendix C: Demo Script (Optional, 15 minutes)

**Goal:** Show the end-to-end loop with minimal risk.

1. Show `operator-sdk version`.
2. `mkdir memcached-operator && cd memcached-operator`
3. Run `operator-sdk init` and `operator-sdk create api` from slide 10.
4. Open the generated `api/v1alpha1/memcached_types.go` and show where the `Spec` and `Status` structs live.
5. Run `make manifests` and show the generated CRD under `config/crd/bases/`.
6. Show `Reconcile` in the controller file. Narrate the loop (Watch, Compare, Act, Report).
7. Run `make bundle` (a placeholder image is fine) and show the `bundle/manifests/` CSV.
8. Run `operator-sdk bundle validate ./bundle`.
9. Optional if the registry and cluster are ready: `operator-sdk run bundle`, then `kubectl get csv`.

**Fallback plan:** Have a pre-built project folder and screenshots ready in case network or registry access fails.

---

## Appendix D: One-Page Cheat Sheet (hand out to learners)

```text
OPERATOR CONCEPT
  CRD (type) + CR (instance) + Controller (logic) + RBAC/Deployment (runtime)
  Reconcile loop: Watch → Compare → Act → Report → repeat

SDK WORKFLOW (Go)
  operator-sdk init --domain example.com --repo <module>
  operator-sdk create api --group G --version V --kind K --resource --controller
  make manifests generate
  make bundle IMG=<image>
  operator-sdk bundle validate ./bundle
  operator-sdk run bundle <bundle-image>
  operator-sdk cleanup <operator-name>

OLM (v0) COMMANDS
  operator-sdk olm install | status | uninstall

OLM OBJECTS
  CatalogSource → Subscription → InstallPlan → CSV → Operator Pod
  OperatorGroup scopes the namespaces the operator watches

OLM v1 OBJECTS
  ClusterCatalog + ClusterExtension (operator-controller, catalogd)

DEBUG ORDER
  Subscription → InstallPlan → CSV → Pod
  kubectl get installplan,catalogsource,csv -A
  kubectl logs -n olm deploy/catalog-operator
```

---

## Appendix E: Trainer Delivery Tips

- **Anchor on the thermostat analogy** early and return to it for reconcile, drift and self-healing.
- **Keep Part 1 and Part 2 connected.** At every bundle or catalog mention, remind learners where in the pipeline they are.
- **Use real cluster output** whenever possible (`kubectl get csv,subscription,installplan -A`). Abstract object names are easier to learn when learners see them.
- **Pause for questions** after slides 6, 10, 16 and 18. These carry the heaviest concepts.
- **Avoid deep controller-runtime internals** unless the audience is Go developers. This session is about the whole lifecycle.
- **Version awareness:** commands, folder names and OLM v1 status change over time. Re-check against current official docs before each delivery.
- **Pace:** if time is short, compress slides 11 to 12 (install steps) into a pre-read and keep the hands-on on slide 21.
