# IoT Operations Demo Applications

This directory contains containerized demo applications for Azure IoT Operations
Kubernetes clusters running on edge devices.

## Applications

### Edge MQTT simulator

`edgemqttsim` publishes configurable factory telemetry to the Azure IoT Operations
MQTT broker with MQTT v5 and K8S-SAT authentication.

[Edge MQTT simulator documentation](./edgemqttsim/README.md)

### Demo historian

`demohistorian` subscribes to factory MQTT topics, stores messages in PostgreSQL,
and exposes health and query endpoints.

[Demo historian documentation](./demohistorian/README.md)

## Prerequisites

- Azure IoT Operations installed on an Arc-connected Kubernetes cluster
- Azure CLI and `kubectl`
- An `aio_config.json` file for the target environment
- A container registry configured in `aio_config.json`

Docker is not required on the management machine when the configured registry is
Azure Container Registry. The deployment script uses ACR cloud builds.

## Quick Deploy

Run the tracked deployment script from the repository root and select one module:

```powershell
.\quickstart\external_configuration\Deploy-EdgeModules.ps1 -ConfigPath "<path-to-aio_config.json>" -ModuleName edgemqttsim -Force
.\quickstart\external_configuration\Deploy-EdgeModules.ps1 -ConfigPath "<path-to-aio_config.json>" -ModuleName demohistorian -Force
```

The script:

1. Loads cluster and registry settings from `aio_config.json`.
2. Builds and pushes the selected image.
3. Connects to the cluster through Azure Arc.
4. Ensures the `mqtt-client` service account and registry pull secret.
5. Refreshes the AIO-managed MQTT broker CA trust bundle in `default`.
6. Applies the module's `deployment.yaml`.
7. Reports deployment status and log commands.

Use `-SkipBuild` only when the updated image already exists in the configured
registry.

## MQTT Security

Both modules use the projected Kubernetes ServiceAccount token at
`/var/run/secrets/tokens/broker-sat`. Before reading or sending that token, each
client validates the MQTT broker certificate with
`/var/run/certs/ca.crt`.

For the default listener on port `18883`, Azure IoT Operations manages the broker
certificate and public trust bundle with cert-manager and trust-manager. The
deployment script projects that managed trust bundle into the `default` namespace
for these demo workloads. Custom Key Vault-backed certificates should be delivered
to Kubernetes through AIO SecretSync rather than copied directly by an application.

If the CA is missing, unreadable, empty, or invalid, the application logs the
specific certificate problem and refuses the K8S-SAT MQTT connection.

## Observe the Demo

```powershell
kubectl get pods -n default -l app=edgemqttsim
kubectl get pods -n default -l app=demohistorian
kubectl logs -n default -l app=edgemqttsim -f
kubectl logs -n default -l app=demohistorian -c historian -f
```

The simulator logs published telemetry. The historian logs its MQTT connection,
subscription, and received message activity.

## Project Structure

```text
modules/
├── README.md
├── edgemqttsim/
│   ├── app.py
│   ├── deployment.yaml
│   ├── Dockerfile
│   ├── message_structure.yaml
│   ├── requirements.txt
│   └── README.md
└── demohistorian/
    ├── app.py
    ├── config.yaml
    ├── deployment.yaml
    ├── Dockerfile
    ├── requirements.txt
    └── README.md
```

## Troubleshooting

```powershell
kubectl describe pod -n default -l app=edgemqttsim
kubectl describe pod -n default -l app=demohistorian
kubectl get configmap azure-iot-operations-aio-ca-trust-bundle -n default
```

Check the application-specific README and pod events for additional guidance.