# Trainer Teleprompter Script

## Introduction to Containerization and Docker

**Format:** Instructor-led session · 10 slides · about 35–40 minutes including questions
**Audience:** Developers, testers and operations staff new to containers

---

### How to use this script

- Text in plain paragraphs is **spoken aloud**, word for word or paraphrased in your own voice.
- Lines starting with `>` are **trainer cues**. Do not read them out.
- **[PAUSE]** means stop for a breath or to let the slide land.
- **[ASK]** means pose the question and wait for answers before moving on.
- **[NEXT]** means advance the slide.
- Timings are guides. Trim the Q&A if you are running over.

### Timing at a glance

| # | Slide | Time |
|---|-------|------|
| 1 | Cover | 1 min |
| 2 | What we will cover | 2 min |
| 3 | The problem containers solve | 4 min |
| 4 | What is containerization | 4 min |
| 5 | Containers compared with virtual machines | 5 min |
| 6 | Docker components | 5 min |
| 7 | Docker images | 5 min |
| 8 | From Dockerfile to running container | 6 min |
| 9 | Docker registry | 4 min |
| 10 | Key takeaways and questions | 4 min |
| | **Total** | **about 40 min** |

---

## Slide 1 · Cover (1 min)

> **On screen:** "Introduction to Containerization — Docker components, images and registry"

Good morning, everyone, and welcome.

Today's session is an introduction to containerization and the core building blocks of Docker. By the end, you will know what a container is, what the main Docker components do, how an image is built and layered, and how registries store and share images.

You do not need any prior Docker experience. If you can read a command line, you are ready.

[PAUSE]

Let me show you what we are going to cover.

> **[NEXT]**

---

## Slide 2 · What we will cover (2 min)

> **On screen:** four numbered cards: 01 Containerization basics, 02 Docker components, 03 Docker images, 04 Docker registry

We have four topics today.

**One**, containerization basics. We will look at the problem containers solve and how they differ from virtual machines.

**Two**, the Docker components. That is the client, the daemon, images, containers and the registry, and how they fit together.

**Three**, Docker images. We will cover layers, Dockerfiles and how to build an image.

**Four**, the Docker registry. That is how images are stored, shared and pulled.

> **Cue:** Mention breaks or question policy here, for example "Please ask questions as we go."

Let's start with the problem.

> **[NEXT]**

---

## Slide 3 · The problem containers solve (4 min)

> **On screen:** the quote "It works on my machine." plus four problem statements

Let's begin with a sentence almost every developer has said or heard.

*"It works on my machine."*

[PAUSE]

[ASK] "Has this happened to anyone here?" Wait for hands or nods.

Why does it happen? Because the application behaves differently from one environment to another. A laptop, a test server and production can each have a different operating system, different libraries and different versions of the same software.

That leads to four common pain points.

First, **different environments**. Operating systems, libraries and versions vary across laptops, test servers and production.

Second, **manual setup**. Setup steps are slow and hard to repeat, so people make mistakes.

Third, **conflicts on shared servers**. When two applications share one server, they can fight over the same dependencies.

Fourth, **slow onboarding**. A new team member can spend days just getting a working environment.

Containers remove much of this variation by shipping the environment together with the application.

> **Cue:** Keep this slide conversational. If the group has a real story of a "works on my machine" failure, let one person share it briefly.

> **[NEXT]**

---

## Slide 4 · What is containerization (4 min)

> **On screen:** definition sentence plus three cards: Isolated, Portable, Lightweight

So what is containerization?

Containerization packages an application together with its libraries, dependencies and configuration into a single, portable unit called a **container**.

[PAUSE]

Three properties matter most.

**Isolated.** Each container has its own filesystem, its own processes and its own view of the network. It is separate from other containers on the same machine.

**Portable.** The same container runs on a laptop, a test server or a cloud platform without changes. That is the direct answer to "it works on my machine."

**Lightweight.** Containers share the host operating system's kernel. Because they do not carry a whole operating system, they start quickly and use fewer resources.

> **Cue:** Emphasise the word "kernel." It sets up the comparison on the next slide.

You may be thinking, "That sounds like a virtual machine." Let's compare the two.

> **[NEXT]**

---

## Slide 5 · Containers compared with virtual machines (5 min)

> **On screen:** six-row comparison table, Virtual machine versus Container

A virtual machine virtualizes the hardware and runs a full guest operating system on top of it. A container virtualizes at the operating system level and shares the host kernel.

Let's walk down the table.

**Isolation.** A virtual machine gives you a full guest operating system. A container gives you process-level isolation with a shared kernel.

**Startup time.** Virtual machines take minutes. Containers take seconds.

**Size.** Virtual machines are measured in gigabytes. Containers are measured in megabytes.

**Overhead.** It is higher for virtual machines and low for containers.

**Density.** You fit fewer virtual machines on a host and more containers on the same host.

**Best for.** Virtual machines suit cases where you need different operating systems. Containers suit packaging and running applications.

> **Cue:** Say this aloud: "These sizes and startup times are typical. They vary by workload." Do not present them as guarantees.

[PAUSE]

[ASK] "Does this mean containers replace virtual machines?" Take one or two answers.

The short answer is no. They solve different problems, and many teams run containers on top of virtual machines.

> **[NEXT]**

---

## Slide 6 · Docker components (5 min)

> **On screen:** five cards: Client, Daemon, Images, Containers, Registry, plus a "typical flow" bar

Now let's look at how Docker itself is put together. Docker uses a client-server architecture, and there are five pieces to know.

**One, the client.** This is the `docker` command-line tool. When you type a command, the client sends your build, pull or run request to the daemon.

**Two, the daemon.** This is a background service called `dockerd`. It does the real work: it builds images, and it creates and manages containers.

**Three, images.** These are read-only templates that hold an application and its dependencies.

**Four, containers.** These are running instances of images. They are isolated from each other and from the host.

**Five, the registry.** This is a service that stores and distributes images. Docker Hub is the best-known example.

[PAUSE]

Read the bar at the bottom as one sentence. The client sends a command, the daemon builds or runs it, and images are pulled from or pushed to a registry.

> **Cue:** If a whiteboard is available, draw three boxes (Client → Daemon → Registry) and trace one `docker run` through them.

> **[NEXT]**

---

## Slide 7 · Docker images (5 min)

> **On screen:** four bullet statements on the left, a stacked layer diagram on the right

Let's look more closely at images.

An image is a **read-only template** used to create containers.

It is **built from layers**, one for each instruction in the Dockerfile.

Layers are **cached and shared**. That speeds up builds and saves disk space.

And every image is **identified by a name and a tag**, for example `python:3.12-slim`.

[PAUSE]

Now look at the diagram on the right. Read it from the bottom up.

At the bottom is the base operating system layer. On top of that sits the language runtime. Then the dependencies. Then the application code. These are all **read-only**.

When you start a container, Docker adds one thin **writable container layer** on top. Anything the running container changes is written there.

This design is why rebuilds are fast. Unchanged layers are reused from the cache. And many containers can share the same image layers, each with its own thin writable layer.

> **Cue:** Point at the dashed box at the top of the diagram when you say "writable layer." It is the only layer that changes at runtime.

> **[NEXT]**

---

## Slide 8 · From Dockerfile to running container (6 min)

> **On screen:** a sample Dockerfile on the left, `docker build` and `docker run` commands on the right

This is the practical slide. A **Dockerfile** describes how to build an image.

Here is a simple example for a Node application. Let's read it line by line.

`FROM node:20-slim` chooses the base image to start from.

`WORKDIR /app` sets the working folder inside the image.

`COPY package*.json ./` copies the dependency list in first.

`RUN npm install` installs those dependencies.

`COPY . .` copies the rest of the application code.

`CMD ["node", "server.js"]` sets the command that runs when the container starts.

Each instruction adds a layer to the image.

> **Cue:** Say this aloud: "This is a Node example, but it is only an illustration. Any language works the same way."

[PAUSE]

Now two commands.

`docker build -t myapp .` creates an image from the Dockerfile in the current folder, and tags it `myapp`.

`docker run -d -p 8080:3000 myapp` starts a container in the background. The `-d` flag means detached. The `-p 8080:3000` flag maps port 8080 on your machine to port 3000 inside the container.

[ASK] "If I open a browser at port 8080, which port inside the container am I reaching?" Expected answer: 3000.

> **Cue:** If you plan a live demo, run both commands now. Have the Dockerfile and a sample app ready in advance.

> **[NEXT]**

---

## Slide 9 · Docker registry (4 min)

> **On screen:** definition, "Public and private" card, "Typical workflow" card with four commands

Once you have built an image, you need a place to share it. That place is a **registry**.

A registry stores and distributes Docker images. Images live in **repositories**, and they are identified by **tags**.

There are three kinds of registry to know.

**Docker Hub** is the default public registry.

**Cloud registries** are offered by the big providers: Amazon ECR, Google Artifact Registry and Azure Container Registry.

**Self-hosted** registries run inside your own network, for teams that need to keep images private.

[PAUSE]

The typical workflow has four steps.

`docker login` signs you in to the registry.

`docker tag myapp user/myapp:1.0` gives your image a name that includes your account and a version.

`docker push user/myapp:1.0` uploads it.

`docker pull user/myapp:1.0` downloads it on any other machine.

Remember the name format: **registry, slash, repository, colon, tag**.

> **Cue:** Say this aloud: "When you do not name a registry, Docker uses Docker Hub by default."

> **[NEXT]**

---

## Slide 10 · Key takeaways (4 min)

> **On screen:** four numbered takeaways

Let's close with four ideas to take away.

**One.** Containers package an application with its dependencies, so it runs consistently anywhere.

**Two.** The Docker client, daemon, images, containers and registry work together.

**Three.** Images are layered, read-only templates, and containers are their running instances.

**Four.** Registries store and share images, so teams can build once and run anywhere.

[PAUSE]

Thank you. I'd like to open the floor now.

[ASK] "What questions do you have?"

> **Cue:** If the room is quiet, prompt with: "What is one place in your current work where a container would remove a setup problem?"

---

## Appendix A · Likely questions and short answers

**Are containers less secure than virtual machines?**
They share the host kernel, so the isolation boundary is different from a virtual machine's full guest operating system. Containers offer process-level isolation. Many teams combine both approaches.

**What is the difference between an image and a container?**
An image is a read-only template. A container is a running instance of an image, with a thin writable layer on top.

**Do I need Docker Hub?**
No. Docker Hub is the default, but you can use a cloud registry or host your own.

**Why copy `package*.json` before the rest of the code?**
It keeps the dependency install in its own layer. If only your code changes, Docker reuses the cached layer and skips the install, which makes rebuilds faster.

**Does Docker only work with Node?**
No. The Node example is an illustration. Any language works the same way.

## Appendix B · Trainer checklist

- [ ] Slides open and tested on the presentation screen
- [ ] Docker installed and working if you plan a live demo
- [ ] Sample Dockerfile and app ready for slide 8
- [ ] Network access confirmed if you will pull from Docker Hub
- [ ] Water, timer and a backup copy of this script
