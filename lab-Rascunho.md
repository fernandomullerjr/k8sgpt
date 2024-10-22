
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################

git status
git add .
git commit -m "Lab de k8sgpt"
git push
git status




### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################

# Welcome

Welcome to the K8sGPT Lab! In this lab, you'll explore how K8sGPT can help you analyze and troubleshoot Kubernetes clusters, focusing on a problematic NGINX deployment.

Prerequisites:

    Access to a Kubernetes cluster
    kubectl installed and configured
    K8sGPT installed
    Your own OpenAI API key (instructions on how to obtain one are provided by your instructor)



Step 1: Authenticate K8sGPT

    Add your OpenAI API key to K8sGPT using k8sgpt auth add command.
    When prompted, enter your OpenAI API key. This key will be securely stored for use with K8sGPT.
    If successful, you should see a confirmation message.


Is OpenAI API key added?







## LAB LOCAL

Ajustando o k8sgpt e chave api do openai no ambiente local.


## Installation
https://docs.k8sgpt.ai/getting-started/installation/


### DEB-based installation (Ubuntu/Debian)
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.24/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb






k8sgpt auth add

> k8sgpt auth add
Warning: backend input is empty, will use the default value: openai
Warning: model input is empty, will use the default value: gpt-3.5-turbo
Enter openai Key: openai added to the AI backend provider list



~~~~bash
> cd ~/cursos/k8sgpt
> kubectl apply -f broken-pod.yml
pod/broken-pod created
>
>
> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS             RESTARTS         AGE
default       broken-pod                         0/1     Pending            0                3s
kube-system   cilium-dwjdr                       1/1     Running            3                6d21h
kube-system   cilium-envoy-skd5w                 1/1     Running            3                6d21h
kube-system   cilium-operator-684498dfc4-dxmq6   1/1     Running            3                6d21h
kube-system   coredns-76f75df574-886g5           1/1     Running            0                68m
kube-system   coredns-76f75df574-mrkcx           1/1     Running            0                68m
kube-system   etcd-wsl2                          1/1     Running            4 (70m ago)      6d21h
kube-system   hubble-relay-6ccc94856b-ntrvf      1/1     Running            3                6d20h
kube-system   hubble-ui-647f4487ff-m7l78         2/2     Running            6                6d20h
kube-system   kube-apiserver-wsl2                1/1     Running            3 (70m ago)      6d21h
kube-system   kube-controller-manager-wsl2       0/1     CrashLoopBackOff   23 (2m28s ago)   6d21h
kube-system   kube-proxy-5t5dt                   1/1     Running            0                68m
kube-system   kube-scheduler-wsl2                0/1     CrashLoopBackOff   22 (107s ago)    6d21h
> k8sgpt analyze
AI Provider: openai

0 default/broken-pod(broken-pod)
- Error: 0/1 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.

1 kube-system/kube-controller-manager-wsl2(kube-controller-manager-wsl2)
- Error: back-off 5m0s restarting failed container=kube-controller-manager pod=kube-controller-manager-wsl2_kube-system(4ed986a66f86e71d65dd0cd33feca8c5)
- Error: the last termination reason is Error container=kube-controller-manager pod=kube-controller-manager-wsl2

2 kube-system/kube-scheduler-wsl2(kube-scheduler-wsl2)
- Error: back-off 5m0s restarting failed container=kube-scheduler pod=kube-scheduler-wsl2_kube-system(a48ed6d62b198647c54966785417e547)
- Error: the last termination reason is Error container=kube-scheduler pod=kube-scheduler-wsl2


> date
Sat Sep 28 16:10:47 -03 2024

 ~/cursos/k8sgpt  main ?2                                
~~~~






- Removendo a taint

~~~~bash

>
> kubectl describe node wsl2 | head -n 22
>
>
>
> kubectl taint nodes wsl2 node-role.kubernetes.io/control-plane:NoSchedule-
node/wsl2 untainted
>
>
> kubectl describe node wsl2 | head -n 22
Name:               wsl2
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=wsl2
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sat, 21 Sep 2024 18:23:12 -0300
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  wsl2
  AcquireTime:     <unset>
  RenewTime:       Sat, 28 Sep 2024 16:16:58 -0300
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS             RESTARTS         AGE
default       broken-pod                         0/1     ErrImagePull       0                6m43s
kube-system   cilium-dwjdr                       1/1     Running            3                6d21h
kube-system   cilium-envoy-skd5w                 1/1     Running            3                6d21h
kube-system   cilium-operator-684498dfc4-dxmq6   1/1     Running            3                6d21h
kube-system   coredns-76f75df574-886g5           1/1     Running            0                75m
kube-system   coredns-76f75df574-mrkcx           1/1     Running            0                75m
kube-system   etcd-wsl2                          1/1     Running            4 (76m ago)      6d21h
kube-system   hubble-relay-6ccc94856b-ntrvf      1/1     Running            3                6d20h
kube-system   hubble-ui-647f4487ff-m7l78         2/2     Running            6                6d20h
kube-system   kube-apiserver-wsl2                1/1     Running            3 (76m ago)      6d21h
kube-system   kube-controller-manager-wsl2       0/1     CrashLoopBackOff   24 (4m5s ago)    6d21h
kube-system   kube-proxy-5t5dt                   1/1     Running            0                75m
kube-system   kube-scheduler-wsl2                0/1     CrashLoopBackOff   23 (3m23s ago)   6d21h
>
>
> date
Sat Sep 28 16:17:15 -03 2024
>
>
> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS             RESTARTS         AGE
default       broken-pod                         0/1     ErrImagePull       0                6m49s
kube-system   cilium-dwjdr                       1/1     Running            3                6d21h
kube-system   cilium-envoy-skd5w                 1/1     Running            3                6d21h
kube-system   cilium-operator-684498dfc4-dxmq6   1/1     Running            3                6d21h
kube-system   coredns-76f75df574-886g5           1/1     Running            0                75m
kube-system   coredns-76f75df574-mrkcx           1/1     Running            0                75m
kube-system   etcd-wsl2                          1/1     Running            4 (77m ago)      6d21h
kube-system   hubble-relay-6ccc94856b-ntrvf      1/1     Running            3                6d20h
kube-system   hubble-ui-647f4487ff-m7l78         2/2     Running            6                6d20h
kube-system   kube-apiserver-wsl2                1/1     Running            3 (76m ago)      6d21h
kube-system   kube-controller-manager-wsl2       0/1     CrashLoopBackOff   24 (4m11s ago)   6d21h
kube-system   kube-proxy-5t5dt                   1/1     Running            0                75m
kube-system   kube-scheduler-wsl2                0/1     CrashLoopBackOff   23 (3m29s ago)   6d21h

 ~/cursos/k8sgpt  main      

> k8sgpt analyze
AI Provider: openai

0 default/broken-pod(broken-pod)
- Error: rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:1.a.b.c": failed to resolve reference "docker.io/library/nginx:1.a.b.c": docker.io/library/nginx:1.a.b.c: not found    
~~~~





> kubectl apply -f pod-corrigido.yaml
pod/broken-pod configured
> date
Sat Sep 28 16:18:46 -03 2024

 ~/cursos/k8sgpt  main !1 ?1              


>
> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS             RESTARTS         AGE
default       broken-pod                         1/1     Running            0                8m31s








# K8sGPT Lab - Using K8sGPT to Troubleshoot a Cluster

K8sGPT Lab - Using K8sGPT to Troubleshoot a Cluster

Welcome to the K8sGPT Lab! In this lab, you'll explore how K8sGPT can help you analyze and troubleshoot Kubernetes clusters, focusing on a problematic NGINX deployment.

Prerequisites:

    Access to a Kubernetes cluster
    kubectl installed and configured
    K8sGPT installed
    Your own OpenAI API key (instructions on how to obtain one are provided by your instructor)



### Step 1: Authenticate K8sGPT

    Add your OpenAI API key to K8sGPT using k8sgpt auth add command.
    When prompted, enter your OpenAI API key. This key will be securely stored for use with K8sGPT.
    If successful, you should see a confirmation message.


Is OpenAI API key added?

controlplane ~ ➜  k8sgpt auth add
Warning: backend input is empty, will use the default value: openai
Warning: model input is empty, will use the default value: gpt-3.5-turbo
Enter openai Key: openai added to the AI backend provider list

controlplane ~ ➜  




> k8sgpt analyze --explain
   0% |                                                                                                     | (0/1, 0 it/hr) [0s:0s]
Error: exhausted API quota for AI provider openai: error, status code: 429, message: You exceeded your current quota, please check your plan and billing details. For more information on this error, read the docs: https://platform.openai.com/docs/guides/error-codes/api-errors.




Google Gemini

Google Gemini allows generative AI capabilities with multimodal approach (it is capable to understand not only text, but also code, audio, image and video). With Gemini models, a new API was introduced, and this is what is now built-in K8sGPT. This API also works against the Google Cloud Vertex AI service. See also Google AI Studio to get started.

    NOTE: Gemini API might be still rolling to some regions. See the available regions for details.

    To use Google Gemini API in K8sGPT, obtain the API key.
    To configure Google backend in K8sGPT with gemini-pro model (see all models here) use auth command: bash k8sgpt auth add --backend googlevertexai --model gemini-pro --password "<Your API KEY>"
    Run the following command to analyze issues within your cluster with the Google provider: bash k8sgpt analyze --explain --backend google



bash k8sgpt auth add --backend googlevertexai --model gemini-pro --password "<Your API KEY>"

bash k8sgpt auth add --backend googlevertexai --model gemini-pro --password "<Your API KEY>"

k8sgpt analyze --explain --backend google


> k8sgpt auth add --backend googlevertexai --model gemini-pro --password  
Error: Backend AI accepted values are 'openai, localai, azureopenai, noopai, cohere, amazonbedrock, amazonsagemaker'

 ~/cursos/k8sgpt  main !1    





curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.41/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb


> k8sgpt auth add --backend --help
Error: Backend AI accepted values are 'openai, localai, ollama, azureopenai, cohere, amazonbedrock, amazonsagemaker, google, noopai, huggingface, googlevertexai, oci, watsonxai'
>





## PENDENTE
- Retomar após cessar bloqueio da cota do Openai.
- Ou, usar LocalAI ou outra AI.











~~~~bash
> journalctl -xeu kubelet
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.691887   17109 status_manager.go:217] "Starting to sync pod status >
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.691897   17109 kubelet.go:2347] "Starting kubelet main sync loop"
Oct 19 16:50:32 wsl2 kubelet[17109]: E1019 16:50:32.691925   17109 kubelet.go:2371] "Skipping pod synchronization" err=>
Oct 19 16:50:32 wsl2 kubelet[17109]: W1019 16:50:32.692179   17109 reflector.go:539] k8s.io/client-go@v0.0.0/tools/cach>
Oct 19 16:50:32 wsl2 kubelet[17109]: E1019 16:50:32.692230   17109 reflector.go:147] k8s.io/client-go@v0.0.0/tools/cach>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702091   17109 cpu_manager.go:214] "Starting CPU manager" policy="n>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702112   17109 cpu_manager.go:215] "Reconciling" reconcilePeriod="1>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702121   17109 state_mem.go:36] "Initialized new in-memory state st>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702186   17109 state_mem.go:88] "Updated default CPUSet" cpuSet=""
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702196   17109 state_mem.go:96] "Updated CPUSet assignments" assign>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702201   17109 policy_none.go:49] "None policy: Start"
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.702980   17109 memory_manager.go:170] "Starting memorymanager" poli>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.703002   17109 state_mem.go:35] "Initializing new in-memory state s>
Oct 19 16:50:32 wsl2 kubelet[17109]: I1019 16:50:32.703084   17109 state_mem.go:75] "Updated machine memory state"
Oct 19 16:50:32 wsl2 kubelet[17109]: E1019 16:50:32.703246   17109 kubelet.go:1550] "Failed to start ContainerManager" >
Oct 19 16:50:32 wsl2 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░
░░ An ExecStart= process belonging to unit kubelet.service has exited.
░░
░░ The process' exit code is 'exited' and its exit status is 1.
Oct 19 16:50:32 wsl2 systemd[1]: kubelet.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░
░░ The unit kubelet.service has entered the 'failed' state with result 'exit-code'.
~~~~


- Efetuado reboot do WSL.

- Iniciado kubeadm init novamente.

OK
Agora subiu k8s

~~~~bash

> kubectl get node -o wide
NAME   STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                       CONTAINER-RUNTIME
wsl2   Ready    control-plane   27d   v1.29.9   192.168.0.104   <none>        Ubuntu 22.04.5 LTS   5.15.153.1-microsoft-standard-WSL2   containerd://1.7.22
> kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS             RESTARTS        AGE
default       broken-pod                         0/1     CrashLoopBackOff   317 (71s ago)   21d
kube-system   cilium-dwjdr                       1/1     Running            13 (29m ago)    27d
kube-system   cilium-envoy-skd5w                 1/1     Running            13 (29m ago)    27d
kube-system   cilium-operator-684498dfc4-dxmq6   1/1     Running            14 (29m ago)    27d
kube-system   coredns-76f75df574-886g5           1/1     Running            9 (6d5h ago)    21d
kube-system   coredns-76f75df574-mrkcx           1/1     Running            9 (6d5h ago)    21d
kube-system   etcd-wsl2                          1/1     Running            14 (29m ago)    27d
kube-system   hubble-relay-6ccc94856b-ntrvf      1/1     Running            24 (6d5h ago)   27d
kube-system   hubble-ui-647f4487ff-m7l78         2/2     Running            24 (6d5h ago)   27d
kube-system   kube-apiserver-wsl2                1/1     Running            13 (29m ago)    27d
kube-system   kube-controller-manager-wsl2       1/1     Running            37 (29m ago)    27d
kube-system   kube-proxy-5t5dt                   1/1     Running            10 (29m ago)    21d
kube-system   kube-scheduler-wsl2                1/1     Running            36 (29m ago)    27d
> date
Sat Oct 19 17:05:52 -03 2024

 ~/kubeadm-wsl2/src/k8s-cri-dockerd  main:master >1 ?1             
~~~~



- Testando
k8sgpt analyze
k8sgpt analyse --explain

~~~~bash
> k8sgpt analyze
AI Provider: AI not used; --explain not set

0: Node wsl2()
- Error: wsl2 has condition of type Ready, reason KubeletNotReady: container runtime status check may not have completed yet

1: Pod default/broken-pod()
- Error: the last termination reason is Completed container=broken-pod pod=broken-pod


> k8sgpt analyse --explain
   0% |                                                                                                                    | (0/2, 0 it/hr) [0s:0s]
Error: exhausted API quota for AI provider openai: error, status code: 429, message: You exceeded your current quota, please check your plan and billing details. For more information on this error, read the docs: https://platform.openai.com/docs/guides/error-codes/api-errors.


~~~~




k8sgpt auth list


>
> k8sgpt auth list
Default:
> openai
Active:
> openai
> googlevertexai
Unused:
> localai
> ollama
> azureopenai
> cohere
> amazonbedrock
> amazonsagemaker
> google
> noopai
> huggingface
> oci
> watsonxai











<https://www.kubelynx.com/article/k8sgpt-troubleshoot-debug-kubernetes-cluster-with-openai-ollama>


<https://medium.com/@Tanzim/how-to-run-ollama-in-windows-via-wsl-8ace765cee12>

4. Update Packages:

Launch the Ubuntu distribution as an administrator and update the packages by running:

sudo apt update && sudo apt upgrade



5. Install Ollama: Now, it’s time to install Ollama! Execute the following command to download and install Ollama on your Linux environment: (Download Ollama on Linux)

curl https://ollama.ai/install.sh | sh

This process may take a couple of minutes as it downloads and sets up Ollama along with its dependencies.

6. Ready to Run Models: Congratulations! You’ve successfully installed Ollama on your WSL environment. You’re now ready to download and run any model of your choice. Check out the list of supported models available in the Ollama library at library (ollama.ai)

ollama run mistral

Replace mistral with the name of the model i.e llama2 llama2, phi, openhermes, codellama, llava, dolphinyou wish to run from the available options.

By following these steps, you’ve seamlessly integrated Ollama into your WSL environment, empowering you to explore various machine learning models effortlessly. Happy prompting!






====================================================================================================================================================
 Upgrading
====================================================================================================================================================
  Package:                                         Old Version:                     New Version:                                             Size:
  ansible                                          9.10.0-1ppa~jammy                10.5.0-1ppa~jammy                                      17.4 MB
  ansible-core                                     2.16.11-1ppa~jammy               2.17.5-1ppa~jammy                                       1.0 MB
  helm                                             3.16.1-1                         3.16.2-1                                               17.3 MB
  snapd                                            2.63+22.04ubuntu0.1              2.65.3+22.04                                           26.4 MB
  ubuntu-minimal                                   1.481.3                          1.481.4                                                   3 KB
  ubuntu-standard                                  1.481.3                          1.481.4                                                   3 KB
  ubuntu-wsl                                       1.481.3                          1.481.4                                                   3 KB
  wsl-setup                                        0.2                              0.5.4~22.04                                               4 KB








> ollama run mistral
zsh: command not found: ollama






<https://github.com/ollama/ollama/blob/main/docs/linux.md>

sudo curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz

Start Ollama:

ollama serve

In another terminal, verify that Ollama is running:

ollama -v

AMD GPU install

If you have an AMD GPU, also download and extract the additional ROCm package:

sudo curl -L https://ollama.com/download/ollama-linux-amd64-rocm.tgz -o ollama-linux-amd64-rocm.tgz
sudo tar -C /usr -xzf ollama-linux-amd64-rocm.tgz





> sudo tar -C /usr -xzf ollama-linux-amd64.tgz
> ollama serve
Couldn't find '/home/fernando/.ollama/id_ed25519'. Generating new private key.
Your new public key is:

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMtoS9HvAeVQKD2i5GWbFOqM0HY2BcZXcwUCdtiAHGxz

2024/10/19 17:50:27 routes.go:1158: INFO server config env="map[CUDA_VISIBLE_DEVICES: GPU_DEVICE_ORDINAL: HIP_VISIBLE_DEVICES: HSA_OVERRIDE_GFX_VERSION: HTTPS_PROXY: HTTP_PROXY: NO_PROXY: OLLAMA_DEBUG:false OLLAMA_FLASH_ATTENTION:false OLLAMA_GPU_OVERHEAD:0 OLLAMA_HOST:http://127.0.0.1:11434 OLLAMA_INTEL_GPU:false OLLAMA_KEEP_ALIVE:5m0s OLLAMA_LLM_LIBRARY: OLLAMA_LOAD_TIMEOUT:5m0s OLLAMA_MAX_LOADED_MODELS:0 OLLAMA_MAX_QUEUE:512 OLLAMA_MODELS:/home/fernando/.ollama/models OLLAMA_MULTIUSER_CACHE:false OLLAMA_NOHISTORY:false OLLAMA_NOPRUNE:false OLLAMA_NUM_PARALLEL:0 OLLAMA_ORIGINS:[http://localhost https://localhost http://localhost:* https://localhost:* http://127.0.0.1 https://127.0.0.1 http://127.0.0.1:* https://127.0.0.1:* http://0.0.0.0 https://0.0.0.0 http://0.0.0.0:* https://0.0.0.0:* app://* file://* tauri://*] OLLAMA_SCHED_SPREAD:false OLLAMA_TMPDIR: ROCR_VISIBLE_DEVICES: http_proxy: https_proxy: no_proxy:]"
time=2024-10-19T17:50:27.085-03:00 level=INFO source=images.go:754 msg="total blobs: 0"
time=2024-10-19T17:50:27.086-03:00 level=INFO source=images.go:761 msg="total unused blobs removed: 0"
time=2024-10-19T17:50:27.086-03:00 level=INFO source=routes.go:1205 msg="Listening on 127.0.0.1:11434 (version 0.3.13)"
time=2024-10-19T17:50:27.086-03:00 level=INFO source=common.go:135 msg="extracting embedded files" dir=/tmp/ollama4196636287/runners
time=2024-10-19T17:50:34.524-03:00 level=INFO source=common.go:49 msg="Dynamic LLM libraries" runners="[cuda_v12 rocm_v60102 cpu cpu_avx cpu_avx2 cuda_v11]"
time=2024-10-19T17:50:34.524-03:00 level=INFO source=gpu.go:199 msg="looking for compatible GPUs"
time=2024-10-19T17:50:36.220-03:00 level=INFO source=types.go:107 msg="inference compute" id=GPU-3afa55e0-67e2-a34e-d60d-2b69b20be42f library=cuda variant=v12 compute=8.9 driver=12.6 name="NVIDIA GeForce RTX 4060" total="8.0 GiB" available="6.9 GiB"
[GIN] 2024/10/19 - 17:51:05 | 200 |      37.791µs |       127.0.0.1 | HEAD     "/"
[GIN] 2024/10/19 - 17:51:05 | 404 |     139.022µs |       127.0.0.1 | POST     "/api/show"
time=2024-10-19T17:51:07.778-03:00 level=INFO source=download.go:175 msg="downloading ff82381e2bea in 16 257 MB part(s)"


> ollama run mistral
pulling manifest
pulling ff82381e2bea...   1% ▕                                                                                    ▏  32 MB/4.1 GB  6.8 MB/s   9m57s 


> ollama run mistral
pulling manifest
pulling ff82381e2bea... 100% ▕████████████████████████████████████████████████████████████████████████████████████▏ 4.1 GB
pulling 43070e2d4e53... 100% ▕████████████████████████████████████████████████████████████████████████████████████▏  11 KB
pulling 491dfa501e59... 100% ▕████████████████████████████████████████████████████████████████████████████████████▏  801 B
pulling ed11eda7790d... 100% ▕████████████████████████████████████████████████████████████████████████████████████▏   30 B
pulling 42347cd80dc8... 100% ▕████████████████████████████████████████████████████████████████████████████████████▏  485 B
verifying sha256 digest
writing manifest
success
>>> como efetuar instalacao do kubeadm
 Para realizar a instalação do `kubeadm`, você deverá seguir estas etapas:

1. **Atualize o sistema operacional**: Antes de começar, atualize seu sistema para garantir que as versões mais recentes dos pacotes estejam
instalados. No Ubuntu, você pode usar o comando `sudo apt-get update`.

2. **Instale as dependências**: Para instalar `kubeadm`, também serão necessários outros pacotes. Para isso, execute:

   ```
   sudo apt-get install -y apt-transport-https curl
   ```


> ss -tulp | grep ollama
tcp   LISTEN 0      4096       127.0.0.1:11434       0.0.0.0:*    users:(("ollama",pid=72111,fd=3))
tcp   LISTEN 0      5          127.0.0.1:39601       0.0.0.0:*    users:(("ollama_llama_se",pid=2241,fd=3))

 ~                





<https://www.kubelynx.com/article/k8sgpt-troubleshoot-debug-kubernetes-cluster-with-openai-ollama>
With my VM's configuration setting of 8GB RAM and 4 CPUs, it took around 70 seconds to start.

Set up the Ollama REST API as the back-end service for K8sGPT. Choose the backend category as localai, which works well with the OpenAI API, but the real provider in operation will be Ollama with Llama 3.

~~~~bash
$ k8sgpt auth add --backend localai --model llama3 --baseurl http://localhost:11434/v1
~~~~

Make it a default provider.

~~~~bash
$ k8sgpt auth default --provider localai
~~~~

K8sGPT add default authentication


This completes the setup needed to use Ollama as a backend for K8sGPT.



k8sgpt auth add --backend localai --model mistral --baseurl http://localhost:11434/v1




- ERRO


> k8sgpt analyze --explain
   0% |                                                                                                                    | (0/2, 0 it/hr) [0s:0s]
Error: failed while calling AI provider localai: error, status code: 404, message: model "llama3" not found, try pulling it first
> k8sgpt auth list
Default:
> localai
Active:
> openai
> localai
> googlevertexai
Unused:
> ollama
> azureopenai
> cohere
> amazonbedrock
> amazonsagemaker
> google
> noopai
> huggingface
> oci
> watsonxai



- Verificando detalhes:

> k8sgpt auth list --details
Default:
> localai
Active:
> openai
   - Model: gpt-3.5-turbo
> localai
   - Model: llama3
   - BaseURL: http://localhost:11434/v1
> googlevertexai
   - Model: gemini-pro
Unused:
> ollama
> azureopenai
> cohere
> amazonbedrock
> amazonsagemaker
> google
> noopai
> huggingface
> oci
> watsonxai

 ~                          


- Ajustado
> vi /home/fernando/.config/k8sgpt/k8sgpt.yaml


18:12h
18:13h


- Testando

OK

Trouxe detalhes sobre como tratar erros:

~~~~bash
> k8sgpt analyze --explain
 100% |██████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (2/2, 8 it/min)
AI Provider: localai

0: Node wsl2()
- Error: wsl2 has condition of type Ready, reason KubeletNotReady: container runtime status check may not have completed yet
 Error: The Kubernetes node (WSL2) is not ready due to the container runtime status check not being completed yet.
   Solution:
   1. Check if the Docker daemon is running on WSL2: `docker ps` or `systemctl status docker`. If it's not running, start it using `sudo systemctl start docker` or `docker start`.
   2. Verify that the container runtime is configured correctly in Kubernetes: `kubectl get csr`. Make sure the signing-contrroller and kubelet-serving are approved.
   3. If the above steps don't work, try restarting the kubelet service: `sudo systemctl restart kubelet`.
   4. Lastly, ensure that the Kubernetes cluster is functioning properly by checking its status with `kubectl get nodes` and verifying that the node (WSL2) is ready. If it's not, try updating the kubelet configuration file to automatically repair taints: `sudo nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf` and add or update the following lines:
   ```
   Environment="KUBELET_CONFIG_CHOKER=0"
   Environment="KUBELET_SYSTEMINODENI_WATCHDOG=15m"
   Environment="KUBELET_CADVISOR_START_TIME=1h"
   ```
   Save the file and restart the kubelet service: `sudo systemctl daemon-reload` followed by `sudo systemctl restart kubelet`.
1: Pod default/broken-pod()
- Error: the last termination reason is Completed container=broken-pod pod=broken-pod
 Error: The container within the pod named 'broken-pod' has terminated with a status of Completed, indicating that it may have encountered an issue.
   Solution: 1. Check the logs of the container using `kubectl logs broken-pod -c broken-container` to identify the error.
           2. If the error is not apparent from the logs, check the Pod's status using `kubectl get pods`. Look for any warning or error messages related to the container.
           3. If the issue persists, consider resolving the problem at its root cause. This could involve modifying the Docker image used in the container, adjusting resource limits, or updating the application code.
           4. After making changes, recreate the pod using `kubectl create -f <file-containing-pod-definition>`. Monitor the new pod to ensure that the issue has been resolved.


 ~      


> k8sgpt analyze --explain --filter=Pod
 100% |██████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (1/1, 3467 it/s)
AI Provider: localai

0: Pod default/broken-pod()
- Error: the last termination reason is Completed container=broken-pod pod=broken-pod
 Error: The container within the pod named 'broken-pod' has terminated with a status of Completed, indicating that it may have encountered an issue.
   Solution: 1. Check the logs of the container using `kubectl logs broken-pod -c broken-container` to identify the error.
           2. If the error is not apparent from the logs, check the Pod's status using `kubectl get pods`. Look for any warning or error messages related to the container.
           3. If the issue persists, consider resolving the problem at its root cause. This could involve modifying the Docker image used in the container, adjusting resource limits, or updating the application code.
           4. After making changes, recreate the pod using `kubectl create -f <file-containing-pod-definition>`. Monitor the new pod to ensure that the issue has been resolved.




>
> k8sgpt analyze --explain --filter=Pod --output=json
{
  "provider": "localai",
  "errors": null,
  "status": "ProblemDetected",
  "problems": 1,
  "results": [
    {
      "kind": "Pod",
      "name": "default/broken-pod",
      "error": [
        {
          "Text": "the last termination reason is Completed container=broken-pod pod=broken-pod",
          "KubernetesDoc": "",
          "Sensitive": []
        }
      ],
      "details": " Error: The container within the pod named 'broken-pod' has terminated with a status of Completed, indicating that it may have encountered an issue.\n   Solution: 1. Check the logs of the container using `kubectl logs broken-pod -c broken-container` to identify the error.\n           2. If the error is not apparent from the logs, check the Pod's status using `kubectl get pods`. Look for any warning or error messages related to the container.\n           3. If the issue persists, consider resolving the problem at its root cause. This could involve modifying the Docker image used in the container, adjusting resource limits, or updating the application code.\n           4. After making changes, recreate the pod using `kubectl create -f \u003cfile-containing-pod-definition\u003e`. Monitor the new pod to ensure that the issue has been resolved.",
      "parentObject": ""
    }
  ]
}
~~~~






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




### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
# Principais materiais

- Getting started:
<https://docs.k8sgpt.ai/getting-started/getting-started/>

- Como instalar Ollama:
<https://medium.aiplanet.com/implementing-rag-using-langchain-ollama-and-chainlit-on-windows-using-wsl-92d14472f15d>
como instalar no linux:
<https://github.com/ollama/ollama/blob/main/docs/linux.md>
obs: No meu caso uso NVIDIA, se fosse GPU da AMD, precisa instalar itens adicionais.

- Como setar o Ollama no k8sgpt, etc:
<https://www.kubelynx.com/article/k8sgpt-troubleshoot-debug-kubernetes-cluster-with-openai-ollama>


### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
# COMANDOS

- RESUMO, subindo Ollama e configurando k8sgpt para usar o Ollama:
ollama serve
ollama run mistral
k8sgpt auth add --backend localai --model mistral --baseurl http://localhost:11434/v1
k8sgpt auth default --provider localai
OBS: se for necessário ajustar o Modelo, é necessário editar o arquivo da conf do k8sgpt:
vi /home/fernando/.config/k8sgpt/k8sgpt.yaml

- Analisar problemas do Kubernetes e fazer a IA explicar a causa/solução:
k8sgpt analyze --explain

- Filtrando apenas por 1 tipo de recurso:
k8sgpt analyze --explain --filter=Pod

- Trazendo o resultado num JSON:
k8sgpt analyze --explain --filter=Pod --output=json


### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
### ###################################################################################################################################################
# PENDENTE

- TSHOOT do Kubelet.
- Retomar Lab da KodeKloud.

- Video do Veronez
<https://www.youtube.com/watch?v=WOoSXd_n_hU>

- Integração com Slack
<https://docs.k8sgpt.ai/tutorials/slack-integration/>



