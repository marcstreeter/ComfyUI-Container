# Tiltfile for running MMRenderer/ComfyUI locally

# 1. Install the official NVIDIA GPU Operator
# We use local_resource with 'helm upgrade --install --wait' because the operator creates CRDs 
# and requires specific installation ordering that raw YAML cannot handle.
# The '--wait' flag perfectly blocks until the operator has fully injected the node and the GPU is ready!
local_resource(
    'gpu-operator',
    cmd='helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update nvidia && helm upgrade --install gpu-operator nvidia/gpu-operator --namespace gpu-operator --create-namespace --wait --set driver.enabled=false --set toolkit.enabled=true --set devicePlugin.enabled=true',
    labels=['infrastructure']
)

# 2. Ensure the target namespace exists in the cluster
k8s_yaml(local('kubectl create namespace mmrender --dry-run=client -o yaml'))


# 3. Load Helm chart with values configured for local development
k8s_yaml(helm(
    'helm/comfyui-nvidia',
    name='comfyui-nvidia',
    namespace='mmrender',
    values=['helm/comfyui-nvidia/values.yaml']
))

# 3. Configure the Kubernetes resource in Tilt
# This ensures Tilt knows how to map local ports to the Pod and provides a better UI experience
k8s_resource(
    'comfyui-nvidia',
    resource_deps=['gpu-operator'],
    port_forwards=['8188:8188'],
    labels=['comfyui'],
    links=[
        link('http://localhost:8188', 'UI: ComfyUI Web Interface'),
        link('http://localhost:8188/api/prompt', 'API: Prompt Endpoint'),
        link('http://localhost:8188/system_stats', 'API: System Stats'),
        # Adding the internal cluster DNS as a link so it shows up in the Tilt UI
        link('http://comfyui-nvidia.mmrender.svc.cluster.local:8188', 'Internal DNS: comfyui-nvidia.mmrender.svc.cluster.local:8188')
    ]
)
