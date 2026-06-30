# Containers, Docker, containerd, and Kubernetes — explained step by step

A plain-language breakdown of four terms that get used interchangeably but actually describe four different layers of the same stack. Written so it makes sense whether you've never touched a terminal or you've been doing ops for years.

## Step 1: What problem are we even solving?

Before any of this existed, deploying software looked like: install an app on a server, hope its dependencies (specific versions of libraries, runtimes, configs) didn't clash with everything else on that server, and pray it behaved the same way it did on your laptop. It often didn't. "It works on my machine" became a running joke because every machine was slightly different.

The fix was to package an application together with everything it needs to run, in a way that behaves identically no matter where you put it. That packaging is a **container**.

## Step 2: What is a container, technically?

A container is an isolated environment for running a single application, created using features built into the Linux kernel — mainly two things:

- **Namespaces**, which give a process its own private view of the system: its own filesystem, its own network interface, its own list of running processes. The application inside thinks it's the only thing on the machine.
- **Cgroups** (control groups), which limit how much CPU, memory, and other resources that process is allowed to use, so one container can't starve everything else on the server.

The key thing to understand: a container is **not** a tiny virtual machine. It doesn't boot its own operating system kernel. It shares the host machine's kernel and just gets a fenced-off slice of it. That's why containers start in milliseconds and use a fraction of the resources a VM would.

A **container image** is the packaged, frozen snapshot of an application plus its dependencies — think of it as a blueprint. A **container** is what you get when that blueprint is actually run as a live process.

## Step 3: What is Docker?

Docker is the tool that made containers usable by ordinary developers. Before Docker, the underlying Linux kernel features existed, but using them directly was painful and manual. Docker wrapped all of it into one product with:

- A simple CLI (`docker run`, `docker build`, `docker push`)
- A way to define images declaratively with a `Dockerfile`
- A daemon (background service) that manages everything on your machine
- A public registry (Docker Hub) to share images

When Docker launched in 2013, it bundled its own full container engine internally. Over time, Docker split that engine out into its own standalone, lower-level component — which is now its own independent open-source project: **containerd**.

So today, when you type `docker run`, Docker's daemon doesn't run the container itself anymore — it delegates that job down to containerd underneath it.

## Step 4: What is containerd?

containerd is a **container runtime** — software whose only job is to manage the actual lifecycle of containers on a single machine:

- Pulling container images from a registry
- Unpacking and storing them on disk
- Creating the isolated namespace/cgroup environment
- Starting, stopping, and monitoring the container process
- Managing local storage and networking for containers

It deliberately does *less* than Docker — no CLI for humans, no image-building tools, no registry. It's a focused background engine meant to be controlled by other software, not by a person typing commands. It's now part of the Cloud Native Computing Foundation (CNCF), the same neutral foundation that governs Kubernetes.

You can think of containerd as the engine, and Docker as the car built around that engine — dashboard, steering wheel, and all the parts a driver actually interacts with.

## Step 5: What is Kubernetes, and why isn't it just "bigger Docker"?

Containers solve running *one* application reliably. But real systems usually involve dozens or hundreds of containers spread across many physical or virtual machines, and that introduces new problems:

- Which machine should each container run on?
- What happens when a machine dies, or a container crashes?
- How do containers find and talk to each other across machines?
- How do you roll out an update without downtime?
- How do you scale a service up or down automatically?

**Kubernetes** is the system that answers all of this. It's a **container orchestrator** — it doesn't run containers itself, it manages a cluster of machines and decides, continuously, what should be running where, restarting things that fail and adjusting as conditions change. You describe the *desired state* ("I want 3 copies of this app running, each with this much memory") and Kubernetes works continuously to keep reality matching that description.

## Step 6: How does Kubernetes actually start a container, then?

This is where everything connects. Kubernetes needs an actual container runtime to do the low-level work on each machine — it doesn't reinvent that wheel. Here's the chain of command, step by step, on every machine (called a *node*) in a cluster:

1. The Kubernetes control plane decides a container needs to run on a particular node.
2. **kubelet**, an agent process running on that node, receives that instruction.
3. kubelet talks to a container runtime through a standardized interface called the **CRI** (Container Runtime Interface) — a contract that lets Kubernetes work with *any* compatible runtime without caring about its internals.
4. **containerd** (the most common CRI-compatible runtime today) receives the request and actually pulls the image and starts the container.

Notice that **Docker is not in this chain at all**. Kubernetes used to support Docker directly through a compatibility shim, but that was removed years ago — Kubernetes now talks to runtimes only through CRI, and containerd implements CRI natively. Docker itself was never CRI-compliant on its own, since CRI didn't exist when Docker was built.

This is a common point of confusion: people assume Kubernetes "runs on Docker" because Docker was the on-ramp most people first learned containers with. In reality, modern Kubernetes clusters typically run on containerd directly, with Docker nowhere in the runtime path.

## Step 7: Putting it all together with an analogy

Imagine a shipping company:

- A **container** (the concept) is a shipping crate — standardized, sealed, and portable, so it doesn't matter what's inside or which ship carries it.
- **containerd** is the dockworker who actually loads, unloads, and moves individual crates on one ship.
- **Docker** is the office software a person uses to request "load this crate" — friendly, with a nice interface, but ultimately just instructing the dockworker (containerd) to do the work.
- **Kubernetes** is the entire shipping company's logistics command center — deciding which ships go where, rerouting around problems, replacing a ship that sinks, and keeping the whole fleet running toward its goals, talking to every ship's dockworker through one standard radio protocol (CRI).

## Quick reference table

| Term | What it is | What it does |
|---|---|---|
| Container | A concept/unit | An isolated, packaged process sharing the host kernel |
| Docker | A developer tool | CLI, image building, local container management — user-facing |
| containerd | A container runtime | Actually starts/stops/manages containers on one machine |
| Kubernetes | An orchestrator | Decides what runs where across many machines, and keeps it that way |

## Why this matters for the CKA exam

CKA clusters are built with `kubeadm` and run on containerd, not Docker. When you troubleshoot at the container level on the exam, you'll use `crictl` (a CLI built to talk directly to containerd/CRI runtimes) rather than `docker` commands — for example `crictl ps`, `crictl images`, and `crictl logs`. Knowing this layering up front means none of that will feel mysterious when you hit it in a lab.
