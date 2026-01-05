# Edge Image Builder Examples

Build customized SUSE Linux Micro images for edge deployments using the [SUSE Edge Image Builder](https://github.com/suse-edge/edge-image-builder).

---

## Available Configurations

| Configuration | Description |
|---------------|-------------|
| **blog-rancher-3node** | A 3-node k3s cluster running Rancher, suitable for a management cluster |
| **elemental** | A simple downstream node that registers to Elemental in a management cluster |

---

## Prerequisites

### Base Image

Download **SL-Micro.aarch64-6.2-Base-SelfInstall-GM.install.iso** from the [SUSE Linux Micro 6.2 download page](https://www.suse.com/download/sle-micro/).

### Required Tools

- [UTM](https://mac.getutm.app/) — Virtual machine host for macOS
- [Podman](https://podman.io/) — Container runtime
- [Ollama](https://ollama.com/) — Local LLM runtime (required for Liz on MacBook)

---

## System Setup

### Install UTM

```bash
brew install utm
```

### Install and Configure Podman

```bash
brew install podman
podman machine init --cpus 6 --memory 4096
podman machine start
podman pull registry.suse.com/edge/3.4/edge-image-builder:1.3.0
```

### Set Up Ollama (MacBook + Liz only)

```bash
brew install ollama
ollama pull qwen3-embedding:4b
ollama pull gpt-oss:20b
```

---

## Building an Image

### 1. Clone the Repository

```bash
git clone https://github.com/achuza/eib-manuf-lab.git
cd eib-manuf-lab/rancher-3node/
```

### 2. Prepare the Base Image

```bash
mkdir base-images
```

Copy your downloaded ISO into the `base-images/` directory:

```
base-images/SL-Micro-aarch64-6.x-Base-SelfInstall-GM-install.iso
```

> **Note:** A SUSE registration code is required if installing additional packages. Add your registration credentials before building.

### 3. Run the Build

```bash
podman run --privileged --rm -it \
  -v $(pwd):/eib \
  registry.suse.com/edge/3.4/edge-image-builder:1.3.0 \
  build --definition-file eib-config.yaml
```

---

## Resources

- [Edge Image Builder Documentation](https://github.com/suse-edge/edge-image-builder)
- [SUSE Linux Micro Downloads](https://www.suse.com/download/sle-micro/)
