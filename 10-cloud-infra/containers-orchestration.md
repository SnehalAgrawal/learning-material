# Cloud Architecture: Containers & Orchestration

### 1. Overview
Containerization and Orchestration have fundamentally changed how applications are shipped and managed. For senior engineers, this is not just about writing a `Dockerfile`, but about understanding the networking, security boundaries, and lifecycle management of applications running in a cluster (Kubernetes).

### 2. Key Concepts
*   **Docker (Containerization)**: Packaging an application and its dependencies into a single image that runs consistently across any environment.
*   **Kubernetes (Orchestration)**: A platform to automate the deployment, scaling, and management of containerized applications.
    *   *Pod*: The smallest deployable unit (one or more containers).
    *   *Service*: An abstract way to expose an application running on a set of Pods as a network service.
    *   *Deployment*: Defines the "Desired State" for pods (e.g., "Run 3 instances of Image V2").
*   **Control Plane vs. Worker Nodes**: The Brain (K8s) vs. the Brawn (Servers).

### 3. Real-World Usage
*   **Microservices Deployment**: Packaging 20 different services as Docker images and using Kubernetes to ensure each has the right amount of CPU/RAM and can talk to each other.
*   **Development Parity**: Ensuring that a developer's Mac, the CI server, and the Production AWS cluster all run the *exact* same binary environment.
*   **Self-Healing**: Kubernetes automatically restarting a container if it crashes or moving it to a healthy server if the original hardware fails.
*   **Secret Management**: Using Kubernetes `Secrets` to inject database passwords into containers at runtime without hardcoding them in the image.

### 4. Tradeoffs
*   **Containers vs. VMs**: Containers are faster and use less memory but have weaker security isolation because they share the same Host OS Kernel.
*   **K8s Complexity vs. Agility**: Kubernetes is a "Design Pattern for Data Centers." It introduces massive learning and operational overhead, but provides unmatched flexibility for scaling and updates.
*   **Image Size**: Using "Alpine" or "Distroless" images reduces attack surface and deployment time but can make debugging harder (no `curl` or `ls` available).

### 5. When NOT to Use
*   **Small, Static Websites**: If you are hosting a static blog or a simple Portfolio, Kubernetes is 1000x too complex. Use Vercel, Netlify, or a simple S3 bucket.
*   **Legacy Desktop Apps**: Apps that require a GUI or specific hardware drivers often don't fit well in a standard container environment.

### 6. Interview Focus
*   **Lifecycle**: "Explain the K8s 'Reconciliation Loop.' What happens when I run `kubectl apply`?"
    * **The "Desired State" Philosophy**: Kubernetes operates on a "Declarative" model, not an imperative one. You don't tell Kubernetes *how* to do something; you tell it *what* you want.
        * **The Control Plane**: The K8s "Brain" (composed of `kube-apiserver`, `etcd`, `kube-controller-manager`, `kube-scheduler`) constantly watches the cluster state.
    * **The Loop (Reconciliation)**:
        1.  **Observe**: The API Server sees that you just ran `kubectl apply -f deployment.yaml`, which requests 3 replicas of your app.
        2.  **Compare**: The `Deployment Controller` compares the *Desired State* (3 pods) with the *Current State* (maybe 0 pods running).
        3.  **Act**: Since there is a mismatch, the Controller creates "Pod" resources to fill the gap.
        4.  **Schedule**: The `Scheduler` assigns these new Pods to worker nodes with available resources.
        5.  **Sync**: The Kubelet on the worker node pulls the Docker image and starts the container.
        6.  **Repeat**: This loop runs continuously. If a pod dies, the loop detects the change and creates a new one to maintain the desired count.
*   **Networking**: "How do two pods in a Kubernetes cluster talk to each other if their IP addresses change every time they restart?" (Hint: Services/InClusterDNS).
    * **The Problem**: Pods are ephemeral. They can be created or destroyed at any moment, and they get a new IP address each time. If Pod A tries to call Pod B directly by IP, the connection will break as soon as Pod B restarts.
    * **The Solution: Services & ClusterDNS**:
        1.  **Services (Stable Abstraction)**: You don't talk to Pods; you talk to *Services*. A Service is a stable virtual IP (ClusterIP) that acts as a load balancer for a set of Pods.
        2.  **Selectors**: You define a `Service` with a `selector` (e.g., `app: backend`). Kubernetes automatically adds any Pod with that label to the Service's endpoint list.
        3.  **In-Cluster DNS**: All Pods in the cluster have access to a DNS service (like CoreDNS).
        4.  **Resolution**: When Pod A wants to talk to Pod B, it doesn't use an IP. It does a DNS lookup for `backend-service.default.svc.cluster.local`.
        5.  **Load Balancing**: The DNS returns the *Service's* stable IP. The Service then routes the traffic to one of the *healthy, real Pods* behind it.
    * **Result**: Pods can come and go, but as long as they have the correct `label`, the Service will automatically route traffic to them, and the Pods can always find each other via the stable Service Name.
*   **Security**: "What are the security risks of running a Docker container as 'root'? How do you prevent it?"
    * **Risk 1: Container Escape (The "Break Out")**:
        * **The Attack**: If an application running as `root` inside a container has a vulnerability (e.g., a Kernel exploit), it can potentially gain "root" privileges on the *Host Machine* itself, not just inside the container.
        * **The Impact**: The attacker can access all other containers on the same host, steal data, or damage the node.
    * **Risk 2: Privilege Escalation**:
        * **The Attack**: Even if the attacker can't escape the host, running as root within the container gives them full control over the container's filesystem and processes. They can modify system libraries, install malware, or tamper with other applications running in different containers on the same node (if sharing resources).
    * **Risk 3: Image Vulnerabilities**:
        * **The Attack**: Many base images (especially older ones) run their default user as root. If you pull a compromised image, it might contain root-level backdoors.
    * **How to Prevent It (The Fixes)**:
        1.  **Non-Root User**: In your `Dockerfile`, explicitly define a non-root user and switch to it before the `CMD`. 
            ```dockerfile
            RUN adduser -u 5678 --disabled-password appuser
            USER appuser
            ```
        2.  **User Namespace Mapping**: Configure Docker/Kubernetes to map the container's root user (UID 0) to a high-numbered, unprivileged user on the host.
        3.  **Drop Capabilities**: Use `securityContext` in Kubernetes to drop dangerous Linux capabilities (like `CAP_SYS_ADMIN`) that the container doesn't need.
        4.  **Read-Only Filesystem**: Run the container's root filesystem as read-only (`readOnlyRootFilesystem: true`) to prevent attackers from writing malware to the container itself.
        5.  **Image Scanning**: Use tools like Trivy or Snyk in your CI/CD pipeline to scan Docker images for known CVEs before deploying them.

### 7. Common Mistakes
*   **The "Latest" Tag**: Using `:latest` for your production images, which causes unpredictable versions to be deployed during auto-scaling events. Always use specific versions or git hashes.
*   **Missing Health Checks**: Not defining `liveness` and `readiness` probes, causing K8s to send traffic to pods that are still "Booting" or "Zombies."
*   **Storing State in Containers**: Thinking a container's filesystem is persistent. If a container restarts, all data not in a "Volume" is deleted.
