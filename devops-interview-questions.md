# DevOps Interview Questions

> "If you can't answer these, you're not ready for DevOps."
> — Principal DevOps Engineer ([@devopsbymo](https://www.tiktok.com/@devopsbymo))

Five foundational questions every DevOps engineer should be able to answer confidently. Each section gives a quick-recall summary, a deeper explanation, and the follow-up details interviewers tend to probe for.

---

## 1. What's the Difference Between Docker and Kubernetes?

**What this question is testing:** Your understanding of containerisation and container orchestration.

### Quick answer
- Docker is a platform used to create, package, and run containers.
- Containers provide a lightweight and consistent environment for applications.
- Kubernetes is a container orchestration platform that manages large numbers of containers.
- Kubernetes handles scaling, load balancing, self-healing, and deployment automation.
- In simple terms: **Docker runs containers, Kubernetes manages containers at scale.**

### Deeper detail

**What containers actually are.** A container packages an application together with its dependencies, libraries, and runtime into a single artifact (an *image*). Unlike a virtual machine, a container shares the host operating system's kernel and isolates processes using Linux primitives — **namespaces** (isolation of PIDs, network, mounts, users) and **cgroups** (resource limits for CPU, memory, I/O). This makes containers far lighter than VMs: they start in milliseconds and have minimal overhead because there's no guest OS to boot.

**Where Docker fits.** Docker popularised containers by giving developers an ergonomic toolchain:
- A `Dockerfile` describes how to build an image layer by layer.
- The **image** is an immutable, versioned, layered filesystem; layers are cached and shared to save space and speed up builds.
- A **registry** (Docker Hub, ECR, GCR, Harbor) stores and distributes images.
- The Docker Engine (`dockerd` + `containerd` + a low-level runtime like `runc`) runs containers from images.
- Docker addresses the classic *"works on my machine"* problem because the same image runs identically everywhere.

Note: Docker is one implementation. The **OCI (Open Container Initiative)** standardises the image and runtime formats, so Kubernetes can run any OCI-compliant container — it no longer needs Docker specifically (the Dockershim was removed in Kubernetes 1.24 in favour of CRI runtimes like `containerd` and CRI-O).

**The problem Kubernetes solves.** Running one container on a laptop is easy. Running hundreds across a fleet of machines — keeping them healthy, reachable, and correctly sized — is hard. Kubernetes is a declarative orchestrator: you describe the *desired state* (e.g. "run 5 replicas of this image"), and the control loop continuously reconciles reality toward that state.

Key Kubernetes building blocks:
- **Pod** — the smallest deployable unit; one or more tightly-coupled containers sharing network and storage.
- **Deployment** — manages replica sets and rolling updates/rollbacks of stateless pods.
- **Service** — a stable virtual IP/DNS name that load-balances across a set of pods.
- **Ingress** — HTTP(S) routing from outside the cluster to services.
- **ConfigMap / Secret** — externalised configuration and sensitive data.
- **StatefulSet / DaemonSet / Job / CronJob** — workload types for stateful apps, per-node agents, and batch jobs.
- **Control plane** — `kube-apiserver`, `etcd` (cluster state store), `scheduler`, and `controller-manager`; the **kubelet** runs on each node.

**Capabilities Kubernetes adds on top of "just running containers":**
- *Self-healing* — restarts failed containers, reschedules pods off dead nodes.
- *Horizontal scaling* — manual or automatic (HPA based on CPU/memory/custom metrics).
- *Service discovery & load balancing* — built-in DNS and virtual IPs.
- *Rolling updates and rollbacks* — zero-downtime deployments with health gates.
- *Declarative configuration* — version-controlled YAML, enabling GitOps.

**How to frame the relationship.** They aren't competitors — they operate at different layers. Docker (or any OCI builder/runtime) produces and runs individual containers; Kubernetes orchestrates many containers across a cluster. A typical pipeline builds a Docker image, pushes it to a registry, and Kubernetes pulls and schedules it. For local single-host workloads, **Docker Compose** is the lightweight orchestration alternative.

### Likely follow-up questions
- What's the difference between a container and a virtual machine?
- How do you reduce a Docker image's size? (multi-stage builds, slim/distroless base images, fewer layers, `.dockerignore`)
- What's the difference between `CMD` and `ENTRYPOINT` in a Dockerfile? Between `COPY` and `ADD`?
- How does Docker layer caching work, and how do you order a Dockerfile to maximise cache hits?
- What is a Pod, and why might it contain more than one container? (sidecars)
- How does a Service find its Pods? Explain labels, selectors, and `kube-proxy`.
- How do liveness, readiness, and startup probes differ?
- How does a rolling update work, and how would you roll one back?
- How do you persist data in Kubernetes? (PV, PVC, StorageClass, StatefulSets)
- How do requests and limits affect scheduling and OOM behaviour?
- Why was Dockershim removed, and what replaced it?

---

## 2. What Happens When You Type a Website URL Into Your Browser?

**What this question is testing:** Your knowledge of networking, DNS, and web communication.

### Quick answer
- The browser first checks DNS to find the website's IP address.
- A connection is established with the web server using TCP/IP and often HTTPS.
- The browser sends an HTTP request to the server.
- The server responds with HTML, CSS, JavaScript, and other resources.
- The browser processes these files and renders the webpage for the user.

### Deeper detail

This is the classic "explain the whole stack" question. A strong answer walks the request top to bottom and shows you understand each layer.

**1. URL parsing.** The browser splits the URL into scheme (`https`), host (`example.com`), optional port, path, query string, and fragment. It may first hit the HSTS list to force HTTPS.

**2. DNS resolution.** The browser needs an IP for the hostname and checks a cascade of caches before going to the network:
- Browser cache → OS cache → router cache → configured resolver.
- If unresolved, the **recursive resolver** walks the hierarchy: root servers → TLD servers (`.com`) → the domain's authoritative nameservers, which return the A (IPv4) / AAAA (IPv6) record.
- Results are cached according to their **TTL**. DNS traditionally uses UDP on port 53 (falling back to TCP), with DoH/DoT now common for privacy.

**3. TCP connection (transport layer).** The browser opens a TCP connection to the server's IP on the port (443 for HTTPS) via the **three-way handshake** (SYN → SYN-ACK → ACK). TCP provides reliable, ordered, error-checked delivery. (HTTP/3 replaces TCP with **QUIC over UDP** to cut handshake latency and avoid head-of-line blocking.)

**4. TLS handshake (for HTTPS).** Before any HTTP data flows, TLS negotiates encryption: agree on protocol/cipher, the server presents its **certificate**, the client validates it against trusted CAs, and both derive session keys (TLS 1.3 does this in a single round trip). The result is confidentiality, integrity, and authentication of the server.

**5. HTTP request.** The browser sends a request line + headers (e.g. `GET / HTTP/2`, `Host`, `User-Agent`, `Accept`, cookies). Modern connections use HTTP/2 or HTTP/3 with multiplexing and header compression.

**6. Server-side processing.** The request may pass through a CDN edge, load balancer, and reverse proxy before reaching an application server. The app may query databases/caches, render a response, and return an HTTP status code (200, 301, 404, 500…) plus headers and a body.

**7. Response and rendering.** The browser receives HTML and begins parsing:
- Builds the **DOM** from HTML and the **CSSOM** from CSS.
- Fetches subresources (CSS, JS, images, fonts) — often in parallel.
- JavaScript can manipulate the DOM; `async`/`defer` affect when scripts block parsing.
- Combines DOM + CSSOM into the **render tree**, performs **layout** (geometry) and **paint** (pixels), then composites layers to the screen.

**8. Ongoing activity.** After first paint, the page may make XHR/fetch calls, open WebSockets, run service workers, and cache assets for repeat visits.

**Cross-cutting concerns worth mentioning:** caching at every level (browser, CDN, server), connection reuse/keep-alive, redirects, and the role of the **OSI/TCP-IP model** layers (link → IP → TCP → TLS → HTTP) — naming these signals depth.

### Likely follow-up questions
- What's the difference between TCP and UDP, and when would you choose each?
- Walk me through the TCP three-way handshake. What is TCP slow start?
- What's the difference between an A, AAAA, CNAME, MX, and TXT record?
- What does DNS TTL control, and how does it affect failover?
- What does a TLS handshake actually negotiate, and what changed in TLS 1.3?
- How does HTTPS give you confidentiality, integrity, *and* authentication?
- What problems do HTTP/2 and HTTP/3 (QUIC) solve over HTTP/1.1? (multiplexing, head-of-line blocking)
- What's the difference between status codes 301/302, 401/403, and 502/504?
- What are CORS and the same-origin policy, and why do they exist?
- How does browser/CDN caching work? (`Cache-Control`, `ETag`, CDN edge caching)
- What's the difference between `async` and `defer` on a script tag, and how do they affect rendering?

---

## 3. What Is Infrastructure as Code (IaC)?

**What this question is testing:** Your understanding of modern infrastructure management and automation.

### Quick answer
- Infrastructure is defined using code rather than manual configuration.
- Changes can be version controlled, reviewed, and audited.
- IaC enables consistent and repeatable deployments.
- Common tools include Terraform, CloudFormation, and Ansible.
- It reduces human error and improves scalability and reliability.

### Deeper detail

**Core idea.** Instead of clicking through a cloud console ("ClickOps") or running ad-hoc commands, you describe your infrastructure — networks, VMs, load balancers, databases, IAM policies — in machine-readable files committed to version control. The tooling then creates, updates, or destroys real resources to match that definition.

**Declarative vs imperative.**
- *Declarative* (Terraform, CloudFormation, Pulumi, Kubernetes manifests): you state the **desired end state**; the tool computes the diff and the steps to reach it. This is the dominant model.
- *Imperative* (raw scripts, parts of Ansible): you specify the **sequence of steps** to perform.

**The role of state and idempotency.** Declarative tools track what they manage. Terraform keeps a **state file** mapping config to real-world resource IDs, enabling `plan` (preview the diff) before `apply`. Operations are **idempotent** — running them repeatedly converges to the same result rather than duplicating resources.

**Provisioning vs configuration management** — two related but distinct categories:
- *Provisioning* stands up the infrastructure itself (compute, network, storage). Tools: **Terraform**, **CloudFormation** (AWS), **Pulumi**, **Bicep** (Azure), **OpenTofu**.
- *Configuration management* installs and configures software on existing machines. Tools: **Ansible**, **Chef**, **Puppet**, **SaltStack**.

**Immutable vs mutable infrastructure.** A modern pattern is *immutable* infrastructure: rather than patching live servers, you bake a new image (e.g. with **Packer**) and replace instances wholesale. This eliminates **configuration drift** — the slow divergence between what's defined and what's actually running.

**Key benefits, expanded:**
- **Version control** — infra changes are diffed, peer-reviewed in pull requests, and tied to commit history; you can roll back to a known-good state.
- **Repeatability & consistency** — spin up identical dev/staging/prod environments, or replicate a whole region for disaster recovery, from the same code.
- **Auditability & compliance** — git history is your change log; policy-as-code tools (**OPA/Conftest**, **Sentinel**, **checkov**, **tfsec**) enforce security/compliance rules before deployment.
- **Speed & scale** — provision dozens of resources in minutes; parameterise with modules/variables to avoid copy-paste.
- **Reduced human error** — no forgotten manual steps; the code *is* the documentation.
- **Disaster recovery** — rebuild from code instead of restoring a snapshot of hand-built infrastructure.

**GitOps** extends IaC: git is the single source of truth, and an automated agent (Argo CD, Flux) continuously reconciles the live environment to match the repository.

**Trade-offs to acknowledge** (shows maturity): state files can contain secrets and must be secured/locked; large blast radius if a bad change is applied; a learning curve; and the need for testing (`terraform plan`, `terratest`, dry runs) to avoid destructive surprises.

### Likely follow-up questions
- What's the difference between declarative and imperative IaC? Where does Ansible sit?
- What is Terraform state, why is it needed, and how do you manage it for a team? (remote backend, state locking)
- How do you keep secrets out of state files and out of version control?
- What's the difference between provisioning and configuration management?
- What is configuration drift, and how does immutable infrastructure prevent it?
- What's the difference between `terraform plan` and `terraform apply`? What does `terraform import` do?
- How do you structure reusable IaC? (modules, variables, workspaces/environments)
- How is IaC different from GitOps? How do Argo CD / Flux fit in?
- How would you test infrastructure code or enforce policy before apply? (checkov, tfsec, OPA/Sentinel, terratest)
- How do you handle a change with a large blast radius safely? (targeted plans, staged rollout, review gates)

---

## 4. What's the Difference Between a Load Balancer and a Reverse Proxy?

**What this question is testing:** Understanding of traffic management and application architecture.

### Quick answer
- A Load Balancer distributes traffic across multiple servers.
- It improves availability, scalability, and fault tolerance.
- A Reverse Proxy sits between clients and backend services.
- It can provide caching, SSL termination, security, and routing.
- Some tools (e.g., NGINX) can perform both roles.

### Deeper detail

These overlap heavily, which is exactly why interviewers ask. The honest framing: **every load balancer is a kind of reverse proxy, but a reverse proxy does more than balance load.** The distinction is one of *primary purpose*, not strict category.

**Reverse proxy — the general concept.** A reverse proxy is a server that sits in front of one or more backend servers and forwards client requests to them. Clients talk only to the proxy; they never see the backends directly. Beyond forwarding, it commonly provides:
- **SSL/TLS termination** — decrypt HTTPS at the edge so backends handle plain HTTP, centralising certificate management.
- **Caching** — serve cached responses for static or repeatable content, reducing backend load.
- **Compression** (gzip/brotli) and request/response rewriting.
- **Security** — hide backend topology, enforce rate limits, integrate a **WAF**, mitigate some DDoS traffic.
- **Path/host-based routing** — send `/api` to one service and `/static` to another (this is essentially what a Kubernetes Ingress does).
- **Authentication, header injection, and observability** at a single choke point.

**Load balancer — the general concept.** A load balancer's primary job is to **distribute incoming requests across multiple backend instances** to maximise throughput and availability. Its defining features:
- **Distribution algorithms** — round-robin, least-connections, weighted, IP-hash.
- **Health checks** — stop routing to unhealthy instances; route around failures.
- **Session persistence ("sticky sessions")** — keep a user pinned to one backend when needed.
- **Horizontal scaling & fault tolerance** — add/remove instances transparently; no single backend is a single point of failure.

**Layer 4 vs Layer 7** — a crucial distinction for this question:
- **L4 (transport)** load balancers route on IP and TCP/UDP port without inspecting payload. Fast and protocol-agnostic (e.g. AWS NLB, HAProxy in TCP mode).
- **L7 (application)** load balancers/reverse proxies understand HTTP — they can route by URL, host, headers, or cookies, terminate TLS, and cache (e.g. AWS ALB, NGINX, Envoy, Traefik). L7 features are what blur the line with reverse proxies.

**Forward proxy vs reverse proxy** (a common follow-up): a **forward** proxy sits in front of *clients* and represents them to the internet (egress filtering, anonymity, corporate gateways). A **reverse** proxy sits in front of *servers* and represents them to clients. Direction is the key difference.

**How to summarise the distinction:**
- Use **load balancer** language when the goal is *spreading traffic across many identical backends for scale and resilience*.
- Use **reverse proxy** language when the goal is *an intelligent front door doing TLS, caching, routing, and security* — even with a single backend.
- Tools like **NGINX, HAProxy, Envoy, and Traefik** do both, which is why in practice the same component often plays both roles. Cloud providers split them into dedicated services (ALB/NLB, Cloud Load Balancing, Azure Front Door / Application Gateway).

### Likely follow-up questions
- What's the difference between a forward proxy and a reverse proxy?
- What's the difference between Layer 4 and Layer 7 load balancing? When would you pick each?
- Name some load-balancing algorithms and when each is appropriate. (round-robin, least-connections, weighted, IP-hash)
- What are sticky sessions, and what problem do they create for scaling?
- What is SSL/TLS termination, and why do it at the load balancer/proxy?
- How do health checks work, and what happens when a backend fails one?
- How would a reverse proxy help mitigate a DDoS attack or add a WAF?
- What does a Kubernetes Ingress controller have in common with a reverse proxy?
- What's the difference between AWS ALB and NLB?
- How does an API gateway differ from a plain reverse proxy?

---

## 5. What Is CI/CD and Why Is It Important?

**What this question is testing:** Knowledge of modern software delivery practices.

### Quick answer
- CI (Continuous Integration) automates code building and testing.
- CD (Continuous Delivery/Deployment) automates application releases.
- It helps detect issues early in the development process.
- Reduces manual effort and deployment risks.
- Enables faster, more reliable software delivery.

### Deeper detail

**Continuous Integration (CI).** Developers merge small changes into a shared mainline frequently (ideally several times a day). Each push triggers an automated pipeline that:
- Builds the application/artifact.
- Runs automated tests — unit, then integration.
- Runs static analysis, linting, and security/dependency scanning (SAST, SCA).

The goal is to **catch integration problems within minutes of writing the code**, when they're cheapest to fix, and to keep the mainline always in a buildable, tested state. CI directly combats "integration hell" — the painful big-bang merges that happen when branches drift apart for weeks.

**Continuous Delivery (CD).** Every change that passes CI is automatically prepared for release and pushed through environments (dev → staging → prod-like). The software is *always in a deployable state*, but the final push to production is a **manual, one-click decision** — common where a business gate or change window is required.

**Continuous Deployment (also CD).** Goes one step further: every change that passes all automated gates is released to production **with no manual approval**. This demands high confidence in the test suite and strong observability/rollback. Don't conflate the two C-D's — explaining the difference signals depth.

**A typical pipeline:**
`commit → build → unit tests → integration tests → security/quality scans → package artifact/image → deploy to staging → automated acceptance/smoke tests → (approval) → deploy to production → post-deploy verification & monitoring`

**Deployment strategies that make CD safe:**
- **Blue/Green** — run two identical environments and switch traffic over instantly (easy rollback by switching back).
- **Canary** — release to a small percentage of users first, watch metrics, then ramp up.
- **Rolling** — replace instances gradually.
- **Feature flags** — decouple *deploy* from *release*; ship code dark and toggle it on for subsets of users.

**Why it matters — the business case:**
- **Faster time to market** — smaller, more frequent releases instead of risky quarterly big-bangs.
- **Early defect detection** — failures surface in minutes, not in production; cheaper to fix.
- **Lower deployment risk** — small batch sizes mean a smaller blast radius and easy rollback.
- **Consistency & repeatability** — the pipeline does the same steps every time, removing manual error and "release day" stress.
- **Developer productivity & morale** — automation frees engineers from toil and tight feedback loops keep them in flow.
- **Auditability** — every change is traceable through the pipeline.

**Supporting practices & tooling.** CI/CD relies on solid version control and branching strategy (trunk-based development, short-lived branches), automated testing at multiple levels (the test pyramid), and IaC for consistent environments. Common tools: **GitHub Actions, GitLab CI, Jenkins, CircleCI, Argo CD, Azure DevOps, Spinnaker.** Maturity is often measured with the **DORA metrics**: deployment frequency, lead time for changes, change failure rate, and mean time to recovery (MTTR).

### Likely follow-up questions
- What's the difference between Continuous Delivery and Continuous Deployment?
- Walk me through the stages of a CI/CD pipeline you've built.
- What's the difference between blue/green, canary, and rolling deployments?
- How do feature flags let you decouple *deploy* from *release*?
- How do you roll back a bad deployment quickly and safely?
- What belongs in CI vs CD? Where do security scans (SAST/DAST/SCA) fit?
- How do you keep pipelines fast as the test suite grows? (parallelism, caching, test pyramid, selective runs)
- How do you manage secrets and environment-specific config in a pipeline?
- What is "integration hell" and how does CI prevent it?
- What are the four DORA metrics, and what do they tell you about delivery health?
- How would you gate a production deploy on quality/health signals?
