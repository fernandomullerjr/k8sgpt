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