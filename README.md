# ConfineClaw-Releases

**Run OpenClaw safely — inside a Firecracker microVM.**

ConfineClaw is a secure, desktop-friendly runtime for OpenClaw, distributed as a prebuilt installer. It removes the need for Docker, npm, or manual setup, and runs everything inside a strongly isolated environment.

---

## 🚀 Why ConfineClaw

OpenClaw is powerful — but it can execute code, modify files, and interact with your system.

ConfineClaw adds a practical safety layer:

* 🔒 **Safer for running untrusted agents** — uses Firecracker microVMs instead of standard container isolation
* 🧱 **Stronger isolation model** — avoids shared-kernel risks common in typical container setups
* ⚡ **Zero setup** — no Docker, no npm, no environment preparation
* 🖱️ **Desktop-first workflow** — launch from a shortcut instead of a complex CLI
* 📦 **Reproducible releases** — prebuilt, versioned, and consistently generated artifacts

---

## ⚡ Quick Start

1. Download the `.run` installer from Releases
2. Make it executable and run:

```bash
chmod +x ConfineClaw-vxxxx.yy.zz.run
./ConfineClaw-vxxxx.yy.zz.run
```

3. Launch ConfineClaw from the desktop shortcut
4. Enter your `sudo` password when prompted
5. (First run only) configure disk sizes (`input.ext4` / `output.ext4`) or use defaults
6. Wait for initialization
7. Login:

```
username: root
password: root
```

You are now inside a Firecracker VM running OpenClaw in an isolated environment.

---

## 🔧 Access & Configuration

### 🖥️ Accessing the OpenClaw Environment

After initialization, ConfineClaw runs OpenClaw inside a Firecracker VM.

#### Option 1 (Recommended)

Use the default desktop workflow to start and access ConfineClaw.

#### Option 2 (Advanced Users)

If you prefer to configure the environment manually, you can first access the Firecracker VM, then enter the OpenClaw container:

```bash
ssh root@172.16.0.2
ctr -n default task exec -t --exec-id configure gateway /bin/bash
```

This method allows advanced users to directly access and configure the OpenClaw environment inside the container.

---

### 🌐 Accessing the Web UI

The OpenClaw UI runs inside the VM and is not exposed by default.

You can forward the port to your host:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@172.16.0.2
```

Then open:

```
http://127.0.0.1:18789
```

---

### ⚠️ Note

These steps are intended for advanced users.
Future versions aim to simplify access and remove the need for manual port forwarding.

---

## 🧩 What You Get

* A fully packaged OpenClaw runtime
* A secure execution environment based on Firecracker
* A clean separation between host system and agent activity
* A ready-to-use setup for running AI agents locally

---

## 📦 Releases

This repository contains only release artifacts:

* Prebuilt binaries
* Packaged runtimes
* `.run` installers

All artifacts are automatically generated from the main repository to ensure consistency and reproducibility.

---

## 📌 Notes

ConfineClaw focuses on **safe execution and ease of use**.

For advanced configuration (models, providers, plugins, etc.),
refer to the upstream OpenClaw project.

---

## 🔥 One-Line Summary

> **ConfineClaw = OpenClaw, but sandboxed, packaged, and ready to run.**
