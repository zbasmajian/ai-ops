# Day 01 – Tools

| Tool | What we used it for | What I liked |
|---|---|---|
| **k9s** | Browsing the `ai-ops-training` cluster in a terminal UI: switching context/namespace, viewing pods, logs and describe output. | Everything about the cluster is a keystroke away (`:pods`, `l`, `d`), with no long kubectl commands to type. |
| **lazydocker** | Viewing Docker containers, images and logs, including the kind node container `ai-ops-training-control-plane`. | One screen shows every container's status, logs and stats; stop/restart is a single key. |
| **Docker Desktop** | Running the Docker engine (on WSL) that hosts the kind node container; recovering it after the "WSL is unresponsive" error. | It's the one thing that has to be running; once it was healthy, `./tools/kind start` resumed the cluster with nothing lost. |
| **Terminal** (Git Bash / PowerShell) | Running `./tools/kind start/stop/list/destroy`, `kubectl`, `docker ps`, and saving outputs with `tee`. | `NO_COLOR=1 … \| tee file.txt` gives a clean record of every step for later review. |
| **Claude Code** | Running the commands, diagnosing failures (missing RBAC binding, stuck WSL service, API server not up yet) and explaining the output. | It explained *why* things failed, not just that they failed, and checked before doing anything destructive. |

## What a kind node container is

- **A Docker container acting as a Kubernetes node.** kind runs each node as a container instead of a VM. Ours is `ai-ops-training-control-plane`, built from the `kindest/node:v1.37.0` image.
- **A small Linux system inside.** It boots systemd and runs containerd and the kubelet. Those are the startup messages in `docker logs`, and the reason the API server needed a few seconds after `docker start`.
- **It's the control plane.** etcd, the API server, the scheduler and the controller manager run as pods inside it, in the `kube-system` namespace.
- **It's also the worker.** The cluster has only this one node, so app pods like `busybox` run inside it too. Their containers are nested, run by the node's own containerd, which is why `docker ps` shows only the node and not `busybox`.
- **Its lifecycle is the cluster's lifecycle.** `./tools/kind stop` stops the container and keeps everything. `start` resumes it, while `destroy` deletes it and the whole cluster with it. Its published ports are how you reach it: the API port, 8081/8443 for ingress, and the 30xxx NodePorts.
