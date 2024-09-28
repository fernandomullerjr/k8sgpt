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