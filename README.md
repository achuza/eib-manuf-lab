# Edge Image Builder - Manufacturing Virtualization Lab

Build customized SUSE Linux Micro images for edge deployments using the [SUSE Edge Image Builder](https://github.com/suse-edge/edge-image-builder).

This repository contains configurations for building edge images with integrated virtualization and Liz AI capabilities. 

---

## Available Configurations

| Configuration | Description | EIB Version | Base OS |
|---------------|-------------|-------------|---------|
| **rancher-3node** | 3-node k3s cluster with Rancher, KubeVirt, Longhorn, CDI, and Rancher AI | 1.3 | SL Micro 6.2 |

### rancher-3node Features

- **Kubernetes**: k3s v1.33.6
- **Management**: Rancher 2.13.0
- **Virtualization**: KubeVirt 0.6.0 + CDI + Dashboard Extension
- **Storage**: Longhorn 1.9.2
- **AI**: Rancher AI Agent + UI Extension
- **IIoT**: Ignition by Inductive Automation

---

## Prerequisites

### Base Images

Download the appropriate base image for your configuration:

- **For rancher-3node**: [SL Micro 6.2 ISO](https://www.suse.com/download/sle-micro/)
  - `SL-Micro.aarch64-6.2-Base-SelfInstall-GM.install.iso`

### Required Tools

- **[Podman](https://podman.io/)** — Container runtime (required for building images)
- **[UTM](https://mac.getutm.app/)** — Virtual machine host for macOS (optional, for testing)
- **[Ollama](https://ollama.com/)** — Local LLM runtime (optional, required for Rancher AI features)

---

## System Setup (MacBook)

### 1. Install Podman (Required)

```bash
brew install podman
podman machine init --cpus 6 --memory 8192 --disk-size 100
podman machine start
```

### 2. Pull Edge Image Builder Container

```bash
# For rancher-3node configuration (EIB 1.3)
podman pull registry.suse.com/edge/3.4/edge-image-builder:1.3.0

# For elemental configuration (EIB 1.0) - if needed
podman pull registry.suse.com/edge/edge-image-builder:1.0.2
```

### 3. Install UTM

```bash
brew install utm
```

### 4. Set Up Ollama (Optional - for Rancher AI features)

```bash
brew install ollama
ollama serve  # Start the Ollama service

# In a new terminal, pull the required models
ollama pull gpt-oss:20b
ollama pull qwen3-embedding:4b
```

---

## Building Images

### Configuration: rancher-3node

This builds a complete virtualization-enabled management cluster with Rancher.

#### 1. Clone the Repository

```bash
git clone https://github.com/achuza/eib-manuf-lab.git
cd eib-manuf-lab
```

#### 2. Prepare the Configuration

Navigate to the rancher-3node directory:

```bash
cd rancher-3node
```

Create base images folder:

```bash
mkdir base-images
```


Place your downloaded base image in the same directory and rename it:

```bash
# Your ISO should be named according to eib-config.yaml
mv ~/Downloads/SL-Micro.aarch64-6.2-Base-SelfInstall-GM.install.iso ./sl62.iso
```

#### 3. Update SUSE Registration Code

Edit `eib-config.yaml` and add your SUSE registration code:

```yaml
operatingSystem:
  packages:
    sccRegistrationCode: YOUR-REGISTRATION-CODE-HERE
```

> **Note:** A valid SUSE registration code is required to install additional packages like `open-iscsi`.

#### 4. Run the Build

```bash
podman run --privileged --rm -it \
  -v $(pwd):/eib \
  registry.suse.com/edge/3.4/edge-image-builder:1.3.0 \
  build --definition-file eib-config.yaml
```

#### 5. Output

The built image will be created in the same directory:
- `rancher-3node-aarch64-6.2.iso`

---

## Deployment

### Deploying to UTM (macOS)

1. Open UTM and create a new virtual machine
2. Select "Virtualize" and choose "Linux"
3. Configure the VM:
   - **Boot ISO/Image**: Select your built image
   - **Memory**: 4096 MB minimum (8192 MB recommended)
   - **CPU Cores**: 4 minimum (8 recommended)
   - **Storage**: 100 GB minimum
   - **Network**: MAC Address of respective node
4. Start the VM to complete automated setup

### Network Configuration

The rancher-3node configuration uses:
- **API VIP**: `192.168.64.10`
- **API Host**: `192.168.64.10.sslip.io`

Ensure your network allows access to these addresses. For UTM, the default shared network typically uses the `192.168.64.x` range.

### Accessing Rancher

After deployment and cluster initialization:

1. Wait for all pods to be running (5-10 minutes)
2. Access Rancher UI at: `https://192.168.64.10.sslip.io`
3. Retrieve the bootstrap password:
   ```bash
   kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
   ```

---

## Troubleshooting

### Build Failures

**Issue**: "No space left on device"
- **Solution**: Increase Podman machine disk size:
  ```bash
  podman machine rm
  podman machine init --cpus 6 --memory 8192 --disk-size 100
  ```

**Issue**: "Permission denied" errors
- **Solution**: Ensure the Podman command includes `--privileged` flag

**Issue**: Package installation failures
- **Solution**: Verify your SUSE registration code is valid and properly set in `eib-config.yaml`

### Runtime Issues

**Issue**: Cluster not initializing
- **Solution**: Check node networking and ensure all three nodes can communicate on the `192.168.64.x` network

**Issue**: Rancher UI not accessible
- **Solution**: Verify all Rancher pods are running:
  ```bash
  kubectl get pods -n cattle-system
  ```

---

## Customization

### Adding/Removing Helm Charts

Modify the `kubernetes.helm.charts` section in `eib-config.yaml`. Each chart requires:
- `name`: Chart name
- `version`: Chart version
- `repositoryName`: Repository reference
- `targetNamespace`: Installation namespace
- `valuesFile`: (optional) Custom values file

### Changing Kubernetes Version

Update the version in `eib-config.yaml`:

```yaml
kubernetes:
  version: v1.33.6+k3s1
```

Check [k3s releases](https://github.com/k3s-io/k3s/releases) for available versions.

---

## Resources

- [Edge Image Builder Documentation](https://github.com/suse-edge/edge-image-builder)
- [SUSE Linux Micro Downloads](https://www.suse.com/download/sle-micro/)
- [Rancher Documentation](https://ranchermanager.docs.rancher.com/)
- [KubeVirt Documentation](https://kubevirt.io/user-guide/)
- [k3s Documentation](https://docs.k3s.io/)
- [Longhorn Documentation](https://longhorn.io/docs/)
