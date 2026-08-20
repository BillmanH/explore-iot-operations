# IoT Operations Applications

This directory contains containerized applications designed to be deployed to
Azure IoT Operations Kubernetes clusters running on edge devices.

These are mostly for demo purposes but can also be used to test and validate an
IoT Operations build.

## Available Applications

### Edge MQTT simulator

An MQTT publisher that generates realistic factory telemetry for Azure IoT
Operations.

**Features:**

- MQTT v5 with ServiceAccountToken (K8S-SAT) authentication
- Configurable industrial equipment and business-event messages
- Topic-based routing for factory telemetry
- Automatic reconnection and message buffering

**Quick Deploy:**

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "your-username"
```

[Read the docs](./edgemqttsim/README.md)

### Demo historian

An MQTT subscriber that stores factory telemetry in PostgreSQL and exposes a
query API.

**Features:**

- MQTT v5 with ServiceAccountToken (K8S-SAT) authentication
- Wildcard MQTT subscriptions
- PostgreSQL message history and retention
- HTTP health and query endpoints

**Quick Deploy:**

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "demohistorian" -RegistryName "your-username"
```

[Read the docs](./demohistorian/README.md)

## Deployment Scripts

This folder contains three modular PowerShell deployment scripts that work with
applications in the modules folder.

### Deploy-ToIoTEdge.ps1

Deploy applications to remote IoT Operations clusters through Azure Arc.

**Usage:**

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "your-username"
.\Deploy-ToIoTEdge.ps1 -AppFolder "demohistorian" -RegistryName "myacr" -RegistryType "acr" -ImageTag "v1.0"
```

**Parameters:**

- `-AppFolder` (required): Name of the application folder to deploy
- `-RegistryName` (required): Docker Hub username or ACR name
- `-RegistryType`: `dockerhub` or `acr` (default: `dockerhub`)
- `-ImageTag`: Image tag (default: `latest`)
- `-SkipBuild`: Skip Docker build and push and use an existing image
- `-EdgeDeviceIP`: Direct SSH connection fallback
- `-ConfigPath`: Override the default configuration location

### Deploy-Local.ps1

Run applications locally for development and testing.

**Usage:**

```powershell
.\Deploy-Local.ps1 -AppFolder "edgemqttsim"
.\Deploy-Local.ps1 -AppFolder "demohistorian" -Mode docker -Port 8080
.\Deploy-Local.ps1 -AppFolder "edgemqttsim" -Mode python -Clean
```

**Parameters:**

- `-AppFolder` (required): Name of the application folder to run
- `-Mode`: `python`, `docker`, `uv`, or `auto` (default: `auto`)
- `-Port`: Local port (default: `5000`)
- `-Build`: Force a Docker rebuild
- `-Clean`: Clean the Python virtual environment before setup

### Deploy-Check.ps1

Check deployment status and health for deployed applications.

**Usage:**

```powershell
.\Deploy-Check.ps1 -AppFolder "edgemqttsim"
.\Deploy-Check.ps1 -AppFolder "demohistorian" -EdgeDeviceIP "192.168.1.100"
```

**Parameters:**

- `-AppFolder` (required): Name of the application folder to check
- `-EdgeDeviceIP`: Direct connection to the edge device
- `-ConfigPath`: Override the default configuration location

## Deployment Workflows

### Two-Machine Workflow

Use this workflow when Docker and Kubernetes access are on separate machines.

**On the machine with Docker:**

1. Build and push the container image to Docker Hub or ACR.
2. Note the full image name, such as `username/edgemqttsim:latest`.

**On the machine with Kubernetes or Arc access:**

Run the deployment script with `-SkipBuild`:

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "your-username" -SkipBuild
```

The script detects whether Docker is missing and provides instructions for
manual image building.

### Remote Deployment

Deploy applications from a Windows development machine to a remote IoT
Operations cluster:

1. Configure the cluster in `../config/aio_config.json`.
2. Run the deployment script from the modules folder:

   ```powershell
   .\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "your-registry"
   ```

The script handles:

- Building Docker images
- Pushing images to the container registry
- Connecting to Arc-enabled clusters
- Deploying to Kubernetes
- Verifying deployment status

### Local Development

Run an application locally before deploying:

```powershell
.\Deploy-Local.ps1 -AppFolder "edgemqttsim"
```

### Check Deployment Status

```powershell
.\Deploy-Check.ps1 -AppFolder "edgemqttsim"
```

## Prerequisites

### Remote Deployment

- Docker Desktop on Windows or macOS, unless using the two-machine workflow
- Azure CLI (`az`)
- `kubectl`
- Access to a container registry
- Azure IoT Operations deployed and connected to Azure Arc

### Local Development

- Python 3.8 or later, Docker, or `uv`
- Application dependencies from the module's `requirements.txt`

## Project Structure

```text
modules/
├── README.md
├── Deploy-ToIoTEdge.ps1
├── Deploy-Local.ps1
├── Deploy-Check.ps1
├── edgemqttsim/
│   ├── app.py
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── deployment.yaml
│   ├── message_structure.yaml
│   └── README.md
└── demohistorian/
    ├── app.py
    ├── Dockerfile
    ├── requirements.txt
    ├── deployment.yaml
    ├── config.yaml
    └── README.md
```

## Configuration

### Cluster Configuration

The IoT Operations cluster configuration is stored in:

```text
../config/aio_config.json
```

This file contains:

- Azure subscription details
- Resource group name
- Cluster name and location
- Deployment preferences

The deployment scripts read this configuration automatically.

### Application Configuration

Each application can provide its own configuration for settings such as:

- Registry type and name
- Image tags
- Development port and runtime preferences

## Common Commands

### Build and Push to Docker Hub

```bash
cd modules/edgemqttsim
docker build -t edgemqttsim:latest .
docker tag edgemqttsim:latest YOUR-DOCKERHUB-USERNAME/edgemqttsim:latest
docker login
docker push YOUR-DOCKERHUB-USERNAME/edgemqttsim:latest
```

### Build and Push to Azure Container Registry

```bash
cd modules/edgemqttsim
docker build -t edgemqttsim:latest .
docker tag edgemqttsim:latest YOUR-ACR-NAME.azurecr.io/edgemqttsim:latest
az acr login --name YOUR-ACR-NAME
docker push YOUR-ACR-NAME.azurecr.io/edgemqttsim:latest
```

After pushing the image, deploy it from a machine without Docker:

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "YOUR-USERNAME" -SkipBuild
```

### Deploy an Application

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "myusername"
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "myusername" -ImageTag "v1.0"
```

### Check Application Status

```powershell
.\Deploy-Check.ps1 -AppFolder "edgemqttsim"
```

### Run Locally

```powershell
.\Deploy-Local.ps1 -AppFolder "edgemqttsim"
.\Deploy-Local.ps1 -AppFolder "edgemqttsim" -Mode docker
```

### View Application Logs

```bash
kubectl logs -n default -l app=edgemqttsim -f
```

### Update an Application

Modify the source and redeploy with a new tag:

```powershell
.\Deploy-ToIoTEdge.ps1 -AppFolder "edgemqttsim" -RegistryName "your-username" -ImageTag "v1.1"
```

## Adding New Applications

To add an application:

1. Create a folder under `modules`.
2. Add the application code, `Dockerfile`, `deployment.yaml`, and dependency
   file.
3. Use `<YOUR_REGISTRY>` in the deployment manifest image name.
4. Use the `app: {app-name}` label for pod selection.
5. Add an application README and an entry to this document.
6. Deploy it with `Deploy-ToIoTEdge.ps1`.

## Technology Stack

- **Container runtime:** Docker
- **Orchestration:** Kubernetes (K3s)
- **Python package manager:** `uv`
- **Edge platform:** Azure IoT Operations
- **Cloud integration:** Azure Arc

## Related Documentation

- [Project README](../../README.md)
- [Edge MQTT simulator](./edgemqttsim/README.md)
- [Demo historian](./demohistorian/README.md)

## Support

For issues or questions:

1. Check the application-specific README.
2. Review the quickstart documentation.
3. Check the Azure IoT Operations documentation.
4. Review Kubernetes logs and events.