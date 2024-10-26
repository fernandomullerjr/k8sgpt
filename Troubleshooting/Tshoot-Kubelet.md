


- TSHOOT com base no k8sgpt:
node zoado

~~~~bash

> kubectl get node
NAME   STATUS   ROLES           AGE   VERSION
wsl2   Ready    control-plane   29d   v1.29.9
>
>
> kubectl get node
NAME   STATUS     ROLES           AGE   VERSION
wsl2   NotReady   control-plane   29d   v1.29.9
> date
Sun Oct 20 20:24:00 -03 2024


> k8sgpt analyze --explain
 100% |██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (1/1, 5 it/min)
AI Provider: localai

0: Node wsl2()
- Error: wsl2 has condition of type Ready, reason KubeletNotReady: [container runtime status check may not have completed yet, PLEG is not healthy: pleg has yet to be successful]
 Error: The Kubernetes node (WSL2) is not ready due to an unhealthy container runtime (Kubelet). The issue might be related to the Container Runtime Status Check not completing yet, or PLEG (Pod sandbox live migration engine) not being healthy.

   Solution:
   1. Verify that Docker Desktop is installed and running on WSL2 if using it as the container runtime. If not, install and start Docker Desktop.
   2. Check if the Docker service is running in WSL2 by executing `sudo systemctl status docker`. If it's not running, restart the service with `sudo systemctl start docker`.
   3. Ensure that the Kubernetes daemonset for kubelet is deployed and running correctly: `kubectl get pods -n kube-system | grep kubelet`. If there are any issues, try to delete and recreate the kubelet pod with `kubectl delete -f /etc/kubernetes/manifests/kubelet.yaml` followed by `kubectl apply -f /etc/kubernetes/manifests/kubelet.yaml`.
   4. If PLEG is not healthy, try to restart the Kubernetes control plane component responsible for live migration: `sudo systemctl restart kube-apiserver`.
   5. After making these changes, wait a few minutes and check if the node status has changed with `kubectl get nodes`. If the issue persists, consider checking the logs of the affected components for more information on the problem.

~~~~







Since you don't have the `kubelet.yaml` manifest file, I will provide you with the necessary steps to create and apply it.

1. First, create the `kubelet.yaml` manifest file with the following content:

   ```yaml
   apiVersion: kubelet.config.k8s.io/v1beta1
   kind: KubeletConfiguration
   address: 0.0.0.0
   clusterDNS:
   - 10.96.0.10
   clusterDomain: cluster.local
   authentication:
     x509:
       clientCAFile: "/etc/kubernetes/pki/ca.crt"
   authorization:
     mode: AlwaysAllow
   cgroupDriver: systemd
   cpuManagerReconcilePeriod: 10s
   evictionHard:
     memory.available: "100Mi"
     nodefs.available: "10%"
     nodefs.inodesFree: "5%"
   hairpinMode: promiscuous-bridge
   healthzBindAddress: 127.0.0.1
   healthzPort: 10248
   httpCheckFrequency: 20s
   imageMinimumGCAge: 2m
   kind: KubeletConfiguration
   loggingDriver: json-file
   makeIPTablesUtilChains: true
   maxOpenFiles: 1000000
   maxPods: 110
   nodeStatusUpdateFrequency: 10s
   registerNode: true
   resolvConf: /etc/resolv.conf
   rotateCertificates: true
   runtimeRequestTimeout: 15m
   staticPodPath: /etc/kubernetes/manifests
   streamSerializer: json
   syncFrequency: 1m
   ```

2. Apply the `kubelet.yaml` manifest file with the following command:

   ```
   kubectl apply -f kubelet.yaml
   ```

3. Check the status of the kubelet pod:

   ```
   kubectl get pods -n kube-system | grep kubelet
   ```

   If the pod is not running, describe the pod to see the events and possible issues:

   ```
   kubectl describe pod kubelet -n kube-system
   ```

4. If the kubelet pod is still not running, check the logs of the kube-controller-manager component:

   ```
   kubectl logs kube-controller-manager-wsl2 -n kube-system
   ```

   Look for any errors or warnings related to the kubelet.

5. If you still cannot find the issue, you can try to restart the kube-controller-manager component:

   ```
   sudo systemctl restart kube-controller-manager
   ```

6. After making these changes, wait a few minutes and check if the node status has changed with:

   ```
   kubectl get nodes
   ```

If the issue persists, consider checking the logs of the affected components for more information on the problem.
























I see you're having an issue with your Kubernetes node in WSL2 being in a NotReady state. Let's help troubleshoot this systematically.

First, let's check if Docker is running properly in WSL2. Can you run these commands and share the output?

```bash
# Check Docker service status
sudo service docker status

# Check kubelet service status
sudo service kubelet status
```

Also, it would be helpful to see the kubelet logs. Please run:

```bash
sudo journalctl -u kubelet -n 50
```

From the k8sgpt output, there are two main issues:
1. Container runtime status check hasn't completed
2. PLEG (Pod Lifecycle Event Generator) is unhealthy

This often happens in WSL2 after:
- System hibernation/sleep
- Docker Desktop restart
- WSL2 restart

A quick fix that often resolves these issues is:

1. Stop Docker Desktop completely (if you're using it)
2. In WSL2, run:
```bash
sudo service docker stop
sudo service kubelet stop
sudo service docker start
sudo service kubelet start
```

Would you like to try these steps first and let me know the result? Then we can investigate further if needed.







> sudo service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Sun 2024-10-20 20:40:13 -03; 20ms ago
       Docs: https://kubernetes.io/docs/
    Process: 95277 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 95277 (code=exited, status=1/FAILURE)
> sudo journalctl -u kubelet -n 50
Oct 20 20:40:12 wsl2 kubelet[95277]: Flag --container-runtime-endpoint has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-conf>
Oct 20 20:40:12 wsl2 kubelet[95277]: Flag --pod-infra-container-image has been deprecated, will be removed in a future release. Image garbage collector will get sandbox image information from CRI.
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.903703   95277 server.go:204] "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.907008   95277 server.go:487] "Kubelet version" kubeletVersion="v1.29.9"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.907030   95277 server.go:489] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.907139   95277 server.go:919] "Client rotation is on, will bootstrap in background"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.907910   95277 certificate_store.go:130] Loading cert/key pair from "/var/lib/kubelet/pki/kubelet-client-current.pem".
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.908909   95277 dynamic_cafile_content.go:157] "Starting controller" name="client-ca-bundle::/etc/kubernetes/pki/ca.crt"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.919473   95277 server.go:745] "--cgroups-per-qos enabled, but --cgroup-root was not specified.  defaulting to /"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.920420   95277 container_manager_linux.go:265] "Container manager verified user specified cgroup-root exists" cgroupRoot=[]
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.920728   95277 container_manager_linux.go:270] "Creating Container Manager object based on Node Config" nodeConfig={"RuntimeCgroupsName":"","SystemCgroupsName":"","KubeletCgroupsName":"","KubeletOOMS>
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.920766   95277 topology_manager.go:138] "Creating topology manager with none policy"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.920774   95277 container_manager_linux.go:301] "Creating device plugin manager"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921123   95277 state_mem.go:36] "Initialized new in-memory state store"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921215   95277 kubelet.go:396] "Attempting to sync node with API server"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921235   95277 kubelet.go:301] "Adding static pod path" path="/etc/kubernetes/manifests"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921257   95277 kubelet.go:312] "Adding apiserver pod source"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921276   95277 apiserver.go:42] "Waiting for node sync before watching apiserver pods"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921650   95277 kuberuntime_manager.go:260] "Container runtime initialized" containerRuntime="containerd" version="1.7.22" apiVersion="v1"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.921797   95277 kubelet.go:809] "Not starting ClusterTrustBundle informer because we are in static kubelet mode"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.922092   95277 server.go:1256] "Started kubelet"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.922229   95277 server.go:162] "Starting to listen" address="0.0.0.0" port=10250
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.922255   95277 ratelimit.go:55] "Setting rate limiting for endpoint" service="podresources" qps=100 burstTokens=10
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.922408   95277 server.go:233] "Starting to serve the podresources API" endpoint="unix:/var/lib/kubelet/pod-resources/kubelet.sock"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.923180   95277 server.go:461] "Adding debug handlers to kubelet server"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.923773   95277 fs_resource_analyzer.go:67] "Starting FS ResourceAnalyzer"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.923889   95277 volume_manager.go:291] "Starting Kubelet Volume Manager"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.924520   95277 desired_state_of_world_populator.go:151] "Desired state populator starts to run"
Oct 20 20:40:12 wsl2 kubelet[95277]: E1020 20:40:12.924972   95277 kubelet.go:1462] "Image garbage collection failed once. Stats initialization may not have completed yet" err="invalid capacity 0 on image filesystem"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.925289   95277 reconciler_new.go:29] "Reconciler: start to sync state"
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.925446   95277 factory.go:221] Registration of the systemd container factory successfully
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.925528   95277 factory.go:219] Registration of the crio container factory failed: Get "http://%2Fvar%2Frun%2Fcrio%2Fcrio.sock/info": dial unix /var/run/crio/crio.sock: connect: no such file or direct>
Oct 20 20:40:12 wsl2 kubelet[95277]: I1020 20:40:12.926487   95277 factory.go:221] Registration of the containerd container factory successfully
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.024771   95277 kubelet_node_status.go:73] "Attempting to register node" node="wsl2"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.031090   95277 kubelet_node_status.go:112] "Node was previously registered" node="wsl2"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.031140   95277 kubelet_node_status.go:76] "Successfully registered node" node="wsl2"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.031846   95277 kuberuntime_manager.go:1541] "Updating runtime config through cri with podcidr" CIDR="192.168.0.0/24"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.032243   95277 kubelet_network.go:61] "Updating Pod CIDR" originalPodCIDR="" newPodCIDR="192.168.0.0/24"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.036191   95277 cpu_manager.go:214] "Starting CPU manager" policy="none"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.036218   95277 cpu_manager.go:215] "Reconciling" reconcilePeriod="10s"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.036230   95277 state_mem.go:36] "Initialized new in-memory state store"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.036315   95277 state_mem.go:88] "Updated default CPUSet" cpuSet=""
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.036329   95277 state_mem.go:96] "Updated CPUSet assignments" assignments={}
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.036334   95277 policy_none.go:49] "None policy: Start"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.037149   95277 memory_manager.go:170] "Starting memorymanager" policy="None"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.037176   95277 state_mem.go:35] "Initializing new in-memory state store"
Oct 20 20:40:13 wsl2 kubelet[95277]: I1020 20:40:13.037268   95277 state_mem.go:75] "Updated machine memory state"
Oct 20 20:40:13 wsl2 kubelet[95277]: E1020 20:40:13.037945   95277 kubelet.go:1550] "Failed to start ContainerManager" err="system validation failed - wrong number of fields (expected 6, got 7)"
Oct 20 20:40:13 wsl2 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Oct 20 20:40:13 wsl2 systemd[1]: kubelet.service: Failed with result 'exit-code'.
lines 1-50/50 (END)
















I see the root cause of the issue from the logs. The kubelet is failing to start due to two main errors:

1. Container Manager error:
```
E1020 20:40:13.037945   95277 kubelet.go:1550] "Failed to start ContainerManager" err="system validation failed - wrong number of fields (expected 6, got 7)"
```

2. Image garbage collection error:
```
E1020 20:40:12.924972   95277 kubelet.go:1462] "Image garbage collection failed once. Stats initialization may not have completed yet" err="invalid capacity 0 on image filesystem"
```

Let's fix this step by step:

1. First, let's check your kubelet configuration. Please run:
```bash
sudo cat /var/lib/kubelet/config.yaml
```

2. Also, let's verify the Docker/containerd status:
```bash
sudo service docker status
sudo service containerd status
```

3. Let's also check if your cgroup configuration is correct. Run:
```bash
mount | grep cgroup
```

Based on the error about system validation failing with wrong number of fields, this suggests there might be a mismatch in your kubelet configuration. This often happens in WSL2 when the cgroup hierarchy isn't properly configured or when there's a mismatch between the container runtime and kubelet configurations.

After you share the output of these commands, we can proceed with the appropriate fixes. The most likely solution will involve:
1. Fixing the cgroup configuration
2. Ensuring containerd is properly configured
3. Possibly resetting the kubelet configuration

Would you please run these commands and share the output?














I see there are multiple issues here:

1. The container runtime connection is failing - crictl can't connect to any of the runtime sockets
2. There's a permission denied error for containerd.sock: `connect: permission denied`

Let's fix this step by step:

1. First, let's check if containerd is actually running:
```bash
sudo service containerd status
```

2. If containerd isn't running, start it:
```bash
sudo service containerd start
```

3. Fix the permissions for the containerd socket:
```bash
sudo chmod 666 /run/containerd/containerd.sock
```

4. Set the correct container runtime endpoint for kubelet. Edit the kubelet configuration:
```bash
sudo nano /var/lib/kubelet/config.yaml
```
Add or update these lines:
```yaml
containerRuntimeEndpoint: unix:///run/containerd/containerd.sock
```

5. Also, let's check the kubelet arguments in the systemd service file:
```bash
sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

If you're using containerd (which is recommended for newer Kubernetes versions), make sure you don't have any Docker-specific settings in there.

6. After making these changes, restart the services in order:
```bash
sudo service containerd restart
sudo systemctl daemon-reload
sudo service kubelet restart
```

Could you try these steps and share:
1. The output of `sudo service containerd status`
2. The contents of your kubelet config file (`/var/lib/kubelet/config.yaml`)
3. The contents of the kubelet service configuration (`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`)

This will help us ensure everything is properly configured before proceeding further.




sudo vi /var/lib/kubelet/config.yaml









Let's fix the containerd issues first. The error messages indicate that containerd either isn't running or has permission issues.

1. First, let's ensure containerd is installed and running:

```bash
# Check if containerd is installed
which containerd

# Start containerd service
sudo service containerd start

# Check status
sudo service containerd status
```

2. Let's properly configure the crictl tool to use containerd by default:

```bash
# Configure crictl to use containerd
sudo crictl config --set \
    runtime-endpoint=unix:///run/containerd/containerd.sock \
    --set image-endpoint=unix:///run/containerd/containerd.sock
```

3. Now let's fix the permissions:

```bash
# Check if the socket exists
ls -l /run/containerd/containerd.sock

# Fix permissions if it exists
sudo chmod 666 /run/containerd/containerd.sock

# Make sure you're in the correct group
sudo usermod -aG containerd $USER
```

4. If containerd isn't starting properly, let's check its configuration:

```bash
# Check containerd config
sudo cat /etc/containerd/config.toml

# If it doesn't exist or is empty, generate a default one
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Can you run these commands and share:
1. The output of `sudo service containerd status`
2. Whether the containerd socket exists (`ls -l /run/containerd/containerd.sock`)
3. Any error messages you get when running these commands

This will help us ensure containerd is properly configured before we move on to fixing kubelet.


























Good, containerd is running properly now. I notice there's a warning about CNI config, but that's not critical at this moment.

Let's fix the kubelet configuration to work with containerd:

1. First, let's set the correct runtime endpoint for crictl:
```bash
# Configure crictl to use containerd explicitly
sudo crictl config runtime-endpoint unix:///run/containerd/containerd.sock
sudo crictl config image-endpoint unix:///run/containerd/containerd.sock
```

2. Now we need to check and modify the kubelet configuration. Please show the contents of:
```bash
sudo cat /var/lib/kubelet/config.yaml
```

And:
```bash
sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

3. While we wait for those configs, let's verify the containerized control plane components:
```bash
# Check if control plane pods exist
sudo crictl ps | grep kube
```

After you share those configuration files, we can:
1. Make sure kubelet is properly configured to use containerd
2. Fix any misconfigurations in the service files
3. Restart kubelet with the correct settings

Could you please share the output of these commands?







  Normal   NodeHasSufficientPID     21s                kubelet          Node wsl2 status is now: NodeHasSufficientPID
  Normal   Starting                 12s                kubelet          Starting kubelet.
  Warning  InvalidDiskCapacity      12s                kubelet          invalid capacity 0 on image filesystem
  Normal   NodeHasSufficientMemory  12s                kubelet          Node wsl2 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    12s                kubelet          Node wsl2 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     12s                kubelet          Node wsl2 status is now: NodeHasSufficientPID
  Normal   Starting                 2s                 kubelet          Starting kubelet.
  Warning  InvalidDiskCapacity      2s                 kubelet          invalid capacity 0 on image filesystem
  Normal   NodeHasSufficientMemory  <invalid>          kubelet          Node wsl2 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    <invalid>          kubelet          Node wsl2 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     <invalid>          kubelet          Node wsl2 status is now: NodeHasSufficientPID









  I see the core issue now. The kubelet is reporting "invalid capacity 0 on image filesystem" which is a common issue in WSL2 environments. This happens because WSL2 doesn't properly report disk capacity to kubelet.

Let's fix this with these steps:

1. First, let's modify the kubelet configuration to ignore disk capacity checks:

```bash
# Create or edit the kubelet configuration file
sudo nano /var/lib/kubelet/config.yaml
```

Add or modify these lines:
```yaml
imageGCHighThresholdPercent: 100
imageGCLowThresholdPercent: 80
evictionHard:
  imagefs.available: "1%"
  nodefs.available: "1%"
  nodefs.inodesFree: "1%"
evictionMinimumReclaim:
  imagefs.available: "5%"
  nodefs.available: "5%"
```

2. We also need to ensure the kubelet has the correct runtime configuration:
```bash
sudo nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Make sure the `KUBELET_KUBEADM_ARGS` line includes:
```
Environment="KUBELET_KUBEADM_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
```

3. Apply the changes:
```bash
# Reload systemd
sudo systemctl daemon-reload

# Restart kubelet
sudo systemctl restart kubelet
```

4. After making these changes, let's verify the node status:
```bash
kubectl get nodes
kubectl describe node wsl2
```

Could you try these steps and share:
1. Any errors you see in `journalctl -u kubelet -n 50`
2. The output of `kubectl get nodes` after making these changes

The goal is to get the kubelet to ignore the disk capacity reporting issues that are common in WSL2 while still maintaining proper node operation.










sudo cat /var/lib/kubelet/config.yaml
sudo vi /var/lib/kubelet/config.yaml









1. Any errors you see in `journalctl -u kubelet -n 50`

Oct 20 20:57:55 wsl2 kubelet[71256]: I1020 20:57:55.054254   71256 state_mem.go:96] "Updated CPUSet assignments" assignments={}
Oct 20 20:57:55 wsl2 kubelet[71256]: I1020 20:57:55.054259   71256 policy_none.go:49] "None policy: Start"
Oct 20 20:57:55 wsl2 kubelet[71256]: I1020 20:57:55.054928   71256 memory_manager.go:170] "Starting memorymanager" policy="None"
Oct 20 20:57:55 wsl2 kubelet[71256]: I1020 20:57:55.054956   71256 state_mem.go:35] "Initializing new in-memory state store"
Oct 20 20:57:55 wsl2 kubelet[71256]: I1020 20:57:55.055054   71256 state_mem.go:75] "Updated machine memory state"
Oct 20 20:57:55 wsl2 kubelet[71256]: E1020 20:57:55.055635   71256 kubelet.go:1550] "Failed to start ContainerManager" err="system validation failed - wrong number of fields (expected 6, got 7)"
Oct 20 20:57:55 wsl2 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Oct 20 20:57:55 wsl2 systemd[1]: kubelet.service: Failed with result 'exit-code'.
lines 1-50/50 (END)




The error has changed to a container manager validation error: `"Failed to start ContainerManager" err="system validation failed - wrong number of fields (expected 6, got 7)"`. This is typically related to the cgroup configuration.

Let's check and fix the cgroup configuration:

1. First, let's verify how cgroups are mounted:
```bash
ls -l /sys/fs/cgroup/
```

2. Let's check your kubelet systemd service configuration:
```bash
sudo cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

3. Next, create a new file to ensure proper cgroup configuration:
```bash
sudo nano /etc/systemd/system/kubelet.service.d/11-cgroup.conf
```

Add these contents:
```ini
[Service]
CPUAccounting=true
MemoryAccounting=true
ExecStartPre=/bin/mkdir -p /sys/fs/cgroup/systemd
ExecStartPre=/bin/mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=systemd --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice"
```

4. Also, let's ensure containerd is using the same cgroup driver. Check the containerd config:
```bash
sudo cat /etc/containerd/config.toml
```

If it doesn't exist or needs updating:
```bash
sudo mkdir -p /etc/containerd
cat <<EOF | sudo tee /etc/containerd/config.toml
version = 2
[plugins."io.containerd.grpc.v1.cri"]
  systemd_cgroup = true
[plugins."io.containerd.grpc.v1.cri".containerd]
  snapshotter = "overlayfs"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      runtime_type = "io.containerd.runc.v2"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true
EOF
```

5. After making these changes:
```bash
# Restart containerd
sudo systemctl restart containerd

# Reload systemd and restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Could you try these steps and share:
1. The output of `ls -l /sys/fs/cgroup/`
2. The contents of your current `/etc/containerd/config.toml`
3. Any new errors from `journalctl -u kubelet -n 50`

This should help us identify and fix the cgroup configuration issue that's preventing the container manager from starting properly.







### Dia 21/10/2024


- TSHOOT do Kubelet:
`journalctl -u kubelet -n 50`
kubelet.go Failed to start ContainerManager err= system validation failed - wrong number of fields (expected 6, got 7



- Iniciando o WSL2 e Docker novamente, node já estava ok:

OK:

> kubectl get node
NAME   STATUS   ROLES           AGE   VERSION
wsl2   Ready    control-plane   30d   v1.29.9
> kubectl get node -w
NAME   STATUS   ROLES           AGE   VERSION
wsl2   Ready    control-plane   30d   v1.29.9
^C%
> date
Mon Oct 21 21:05:43 -03 2024

~~~~bash

> systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2024-10-21 21:02:42 -03; 3min 36s ago
       Docs: https://kubernetes.io/docs/
   Main PID: 1635 (kubelet)
      Tasks: 24 (limit: 14999)
     Memory: 139.1M
     CGroup: /system.slice/kubelet.service
             └─1635 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/ku>

Oct 21 21:04:55 wsl2 kubelet[1635]: E1021 21:04:55.896439    1635 pod_workers.go:1298] "Error syncing pod, skipping" er>
Oct 21 21:05:10 wsl2 kubelet[1635]: I1021 21:05:10.896998    1635 scope.go:117] "RemoveContainer" containerID="4982e85e>
Oct 21 21:05:10 wsl2 kubelet[1635]: E1021 21:05:10.897172    1635 pod_workers.go:1298] "Error syncing pod, skipping" er>
Oct 21 21:05:24 wsl2 kubelet[1635]: I1021 21:05:24.902605    1635 scope.go:117] "RemoveContainer" containerID="4982e85e>
Oct 21 21:05:24 wsl2 kubelet[1635]: E1021 21:05:24.903023    1635 pod_workers.go:1298] "Error syncing pod, skipping" er>
Oct 21 21:05:39 wsl2 kubelet[1635]: I1021 21:05:39.903511    1635 scope.go:117] "RemoveContainer" containerID="4982e85e>
Oct 21 21:05:39 wsl2 kubelet[1635]: E1021 21:05:39.903687    1635 pod_workers.go:1298] "Error syncing pod, skipping" er>
Oct 21 21:05:54 wsl2 kubelet[1635]: I1021 21:05:54.899372    1635 scope.go:117] "RemoveContainer" containerID="4982e85e>
Oct 21 21:05:54 wsl2 kubelet[1635]: E1021 21:05:54.899540    1635 pod_workers.go:1298] "Error syncing pod, skipping" er>
Oct 21 21:06:08 wsl2 kubelet[1635]: I1021 21:06:08.900626    1635 scope.go:117] "RemoveContainer" containerID="4982e85e>

~~~~




### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
# PENDENTE

- TSHOOT do Kubelet:
`journalctl -u kubelet -n 50`
kubelet.go Failed to start ContainerManager err= system validation failed - wrong number of fields (expected 6, got 7