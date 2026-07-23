# Tiltfile for running MMRenderer/ComfyUI locally
# Assumes MMInfra has already provisioned the cluster with the GPU Operator.

# 1. Preflight: verify GPU Operator was provisioned by MMInfra
gpu_ready = local(
    'kubectl get daemonset nvidia-device-plugin-daemonset -n gpu-operator -o jsonpath="{.status.numberReady}" 2>/dev/null || echo "0"',
    quiet=True,
)
if int(str(gpu_ready).strip()) == 0:
    fail("GPU Operator not found or not ready. Cluster not provisioned — please run `mise cluster:up` in MMInfra first.")

# 2. Ensure the target namespace exists in the cluster
k8s_yaml(local('kubectl create namespace mmrender --dry-run=client -o yaml'))

# 2. Load Helm chart with values configured for local development
# Use hostPath volumes pointing at the actual repo directories so existing model data is mounted
#
# On Fedora, /home is a symlink to var/home. The kind node (Debian) only has /home/... mounted,
# so paths must use /home/... not /var/home/... for hostPath volumes to resolve correctly.
def kind_path(p):
    if p.startswith('/var/home/'):
        return '/home/' + p[10:]
    return p

k8s_yaml(helm(
    'helm/comfyui-nvidia',
    name='comfyui-nvidia',
    namespace='mmrender',
    values=['helm/comfyui-nvidia/values.yaml', 'helm/comfyui-nvidia/values.local.yaml'],
    set=[
        'volumes.run.hostPath=' + kind_path(os.path.abspath('run')),
        'volumes.basedir.hostPath=' + kind_path(os.path.abspath('basedir')),
    ]
))

# 3. Configure the Kubernetes resource in Tilt
k8s_resource(
    'comfyui-nvidia',
    port_forwards=['8188:8188'],
    labels=['comfyui'],
    links=[
        link('http://localhost:8188', 'UI: ComfyUI Web Interface'),
        link('http://localhost:8188/api/prompt', 'API: Prompt Endpoint'),
        link('http://localhost:8188/system_stats', 'API: System Stats'),
        link('http://comfyui-nvidia.mmrender.svc.cluster.local:8188', 'Internal DNS: comfyui-nvidia.mmrender.svc.cluster.local:8188')
    ]
)
