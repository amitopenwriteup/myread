# Trainer Teleprompter Script

## Oracle Cloud Infrastructure Registry (OCIR) — Lab Deck

**Format:** Instructor-led session with live demos · 9 slides · about 35–40 minutes including questions
**Audience:** Developers and DevOps engineers new to OCI Container Registry

---

### How to use this script

- Plain paragraphs are **spoken aloud**, word for word or in your own voice.
- Lines starting with `>` are **trainer cues**. Do not read them out.
- **[PAUSE]** means stop for a breath and let the slide land.
- **[ASK]** means pose the question and wait for answers.
- **[DEMO]** means switch to the Console or terminal.
- **[NEXT]** means advance the slide.
- Timings are guides. Shorten the Q&A if you are running over.

### Timing at a glance

| # | Slide | Time |
|---|-------|------|
| 1 | Title | 1 min |
| 2 | What Is OCIR? | 4 min |
| 3 | OCIR Images: Key Concepts | 5 min |
| 4 | The OCIR Image Lifecycle | 3 min |
| 5 | Demo Part 1: Create Repository | 6 min |
| 6 | Demo Part 2: Push Images | 6 min |
| 7 | Demo Part 3: Pull Images | 5 min |
| 8 | Demo Part 4: Image Management Tasks | 5 min |
| 9 | Summary | 3 min |
| | **Total** | **about 38 min** |

### Before you start (demo prep)

- [ ] OCI Console open and signed in
- [ ] Docker running on your machine, with the `nginx:latest` image already pulled locally
- [ ] Tenancy namespace and your region key written down (the deck uses `mytenancy` and `iad` as examples)
- [ ] Policy in place that lets your user manage repositories in the target compartment
- [ ] Terminal font enlarged so the back row can read it

---

## Slide 1 · Title (1 min)

> **On screen:** "Oracle Cloud Infrastructure Registry (OCIR)" — "Concepts, repository management, and image lifecycle: create, push, pull, and manage"

Welcome, everyone.

Today's session is about the Oracle Cloud Infrastructure Registry, usually called OCIR. It is now officially named OCI Container Registry, but you will still see "OCIR" everywhere in the documentation and in image paths.

We will cover the key concepts first, then work through four live demos: create a repository, push an image, pull an image, and manage your images.

[PAUSE]

Let's start with the basics.

> **[NEXT]**

---

## Slide 2 · What Is OCIR? (4 min)

> **On screen:** definition plus four numbered cards: Fully Managed, Docker Compatible, IAM Integrated, Regional Endpoints

So what is OCIR?

OCIR is Oracle's **fully managed, Docker-compatible registry service**. You use it to store and share container images across your OCI tenancy.

There are four things to remember.

**One, fully managed.** There is no infrastructure to patch or scale. Oracle operates the registry service for you.

**Two, Docker compatible.** It works with the standard `docker` command-line tool. The same commands you already know apply: login, tag, push and pull.

**Three, IAM integrated.** Access is controlled through OCI IAM policies and compartments. So the same permissions model you use for the rest of OCI applies here.

**Four, regional endpoints.** Each region has its own registry URL. For example, `iad.ocir.io`.

[PAUSE]

[ASK] "Which region does your team use most?" Take one or two answers, and note the region key for the demos.

> **Cue:** Stress that the registry is regional. An image pushed to one region's endpoint is not automatically in another region.

> **[NEXT]**

---

## Slide 3 · OCIR Images: Key Concepts (5 min)

> **On screen:** six concept tiles plus the image path format at the bottom

This slide is the vocabulary for the rest of the session. Six terms.

**Registry.** This is the regional service endpoint. For example `iad.ocir.io` or `phx.ocir.io`.

**Repository.** A named collection of related images. For example, `myapp`.

**Image.** A versioned container image. It is a repository plus a tag.

**Tag.** A label on an image version. For example `1.0` or `latest`.

**Auth token.** A token you use instead of a password when you run `docker login`.

**Visibility.** Public or private. It controls whether someone can pull without authenticating.

[PAUSE]

Now the most important line on the slide, the **image path format**. Read it from left to right:

region key, dot, `ocir.io`, slash, tenancy namespace, slash, repository name, colon, tag.

So a full path looks like `iad.ocir.io/` followed by your namespace, then the repository, then the tag.

> **Cue:** Point to each segment of the path as you say it. Learners will need this exact format in the push and pull demos.

[ASK] "Which part of the path tells Docker which region to talk to?" Expected answer: the region key at the start.

> **[NEXT]**

---

## Slide 4 · The OCIR Image Lifecycle (3 min)

> **On screen:** four steps: 01 Create Repository, 02 Push Images, 03 Pull Images, 04 Manage Images

There are four core operations you will perform when managing images in OCIR. They are also the structure of our demos.

**One, create a repository.** You provision a named repository in a compartment.

**Two, push images.** You tag a local image and run `docker push` to send it to OCIR.

**Three, pull images.** You run `docker pull`, either by tag or by an immutable digest.

**Four, manage images.** You add tags, list images, set visibility and retention, and delete what you no longer need.

[PAUSE]

Let's do each of these live.

> **[NEXT]**

---

## Slide 5 · Demo Part 1: Create Repository (6 min)

> **On screen:** three steps on the left (sign in, create the repository, generate an auth token) and a terminal-style panel on the right

**[DEMO]** We begin in the Console. In this first demo we provision a private repository before we push anything.

**Step one, sign in to the Console.** Then open the menu and go to **Developer Services, then Container Registry**.

**Step two, create the repository.** Click **Create Repository**. Choose the compartment. Enter the name. In the lab we use `devops-lab/myapp`. And set the visibility to **Private**.

> **Cue:** Point out that the repository name uses a slash. The name `devops-lab/myapp` becomes part of the image path in the next demo.

**Step three, generate an auth token.** Go to **User Settings, then Auth Tokens, then Generate Token**.

[PAUSE]

This next point is important. **Copy the token now. You can only see it once.** If you lose it, you must generate a new one.

> **Cue:** Tell learners to store the token in a password manager or a secure note. Do not paste it into chat or into shared screens.

[ASK] "Why do we use a token and not our console password?" Expected answer: the token is used with the Docker CLI in place of a password, and it can be revoked without changing the account password.

> **[NEXT]**

---

## Slide 6 · Demo Part 2: Push Images (6 min)

> **On screen:** three steps on the left (log in, tag, push) and the terminal commands on the right

**[DEMO]** Now we push an image. Three steps.

**Step one, log in to OCIR.** Run `docker login iad.ocir.io`.

When Docker asks for a username, use your **tenancy namespace, slash, your username**. In the example that is `mytenancy/john.doe`. For the password, paste the **auth token**.

**Step two, tag the local image.** We start from a local `nginx:latest` image. Run `docker tag`, then the source image, then the full OCIR path: `iad.ocir.io/mytenancy/devops-lab/myapp:1.0`.

The tag command points the new name at the full repository path. Nothing is uploaded yet.

**Step three, push to OCIR.** Run `docker push` with the same full path. Docker uploads the image layers to the repository.

[PAUSE]

When the push finishes, go back to the Console and refresh the repository. The image and its tag `1.0` should now appear.

> **Cue:** If the login fails, check the three usual causes: wrong region endpoint, wrong username format, or a token that was copied with extra spaces.

> **[NEXT]**

---

## Slide 7 · Demo Part 3: Pull Images (5 min)

> **On screen:** three steps on the left (pull by tag, pull by digest, run the image) and commands on the right

**[DEMO]** Now we retrieve the image. There are two ways to pull.

**Pull by tag.** Run `docker pull` with the full path ending in `:1.0`. This fetches whatever image the tag currently points to.

**Pull by digest.** Here the path ends in `@sha256:` followed by the digest. A digest is **immutable**. It guarantees you get the exact same image bytes every time.

[PAUSE]

Why does that matter? A tag can be moved to a different image later. A digest cannot. So for production deployments, a digest gives you repeatability.

**Run the pulled image.** Run `docker run -d -p 8080:80`, then the full image path. The `-d` flag runs it in the background. And `-p 8080:80` maps port 8080 on your machine to port 80 in the container.

Open a browser at port 8080 to verify that the container started correctly.

[ASK] "Which would you use in a production deployment, tag or digest, and why?" Expected answer: digest, because it cannot change.

> **[NEXT]**

---

## Slide 8 · Demo Part 4: Image Management Tasks (5 min)

> **On screen:** three tasks on the left (add and list tags, change visibility, delete and retain) and commands on the right

**[DEMO]** The last demo covers the day-to-day management of images.

**Add and list tags.** You can apply additional tags to an image, for example `latest`. To review the versions, use the Console or the CLI with `oci artifacts container image list`.

> **Cue:** The slide shows a short local form of the tag command. For an image in OCIR, the tag you apply must use the full OCIR path, as in the push demo.

**Change visibility.** In the Console, choose **Edit Repository** and toggle the visibility between **Private** and **Public**.

[PAUSE]

Be careful here. A public repository allows **unauthenticated pulls**. Only make a repository public on purpose.

**Delete and retain.** You can remove old tags with `oci artifacts container image delete`. And for automatic cleanup, create a **retention policy**: **Retention Policies, then Create Policy**.

> **Cue:** Explain that a retention policy saves manual cleanup work as repositories grow.

> **[NEXT]**

---

## Slide 9 · Summary (3 min)

> **On screen:** four takeaways and "Thank you"

Let's recap four things.

**One.** OCIR is Oracle's managed, Docker-compatible container registry.

**Two.** The core concepts are registry, repository, image, tag and auth token.

**Three.** The workflow is: create a repository, push, pull, then manage your images.

**Four.** Management tasks include tagging, visibility, retention and deletion.

[PAUSE]

Thank you. I'd like to open the floor for questions.

[ASK] "What questions do you have?"

> **Cue:** If the room is quiet, ask: "What would you need to do before your team could use OCIR for a real application?"

---

## Appendix A · Quick command reference

All values below are the examples used in the deck.

```bash
# Log in (username format: <tenancy-namespace>/<username>; password: auth token)
docker login iad.ocir.io

# Tag a local image with the full OCIR path
docker tag nginx:latest iad.ocir.io/mytenancy/devops-lab/myapp:1.0

# Push
docker push iad.ocir.io/mytenancy/devops-lab/myapp:1.0

# Pull by tag
docker pull iad.ocir.io/mytenancy/devops-lab/myapp:1.0

# Pull by digest
docker pull iad.ocir.io/mytenancy/devops-lab/myapp@sha256:<digest>

# Run the pulled image
docker run -d -p 8080:80 iad.ocir.io/mytenancy/devops-lab/myapp:1.0

# List and delete images (OCI CLI)
oci artifacts container image list
oci artifacts container image delete
```

**Image path format:** `<region-key>.ocir.io/<tenancy-namespace>/<repository-name>:<tag>`

## Appendix B · Likely questions and short answers

These answers are trainer aids and are not on the slides. Check them against your tenancy and the current OCI documentation.

**Why does login fail with "unauthorized"?**
Check the endpoint matches your region, the username includes the tenancy namespace, and the token was pasted without extra spaces. Accounts that sign in through federated identity can have a different username format, so check your own tenancy's format before the session.

**Can I recover a lost auth token?**
No. Tokens are shown once. Generate a new one and update any scripts that used the old one.

**What is the difference between a tag and a digest?**
A tag is a movable label. A digest identifies the exact image content and does not change.

**Is an image in one region available in another?**
Each region has its own registry endpoint, so plan for pushing to each region you need.

**Who can pull a private image?**
Only authenticated users whose IAM policies allow it.
