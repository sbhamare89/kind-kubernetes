# kind-kubernetes
This repository created for creating "kubernetes in docker" on your local machine

Our configuration as followed, it may varry according to your existing setup.
Operating system : popos - Ubuntu 22.04
RAM : 8GB ( minimum )

Kubernetes cluster details:
- 1 master node
- 2 worker nodes

Steps to get it ready,
- Get Kind binary on your cli
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.18.0/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind
- Install kubectl on your cli
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo mv ./kubectl /usr/local/bin/kubectl
- Clone git repository to your local
    git clone git@github.com:sbhamare89/kind-kubernetes.git
- Create kuberenets cluster
    kind create cluster --config config.yaml

And that's it, your kubernetes local cluster is up and running.

You can access it via below commands,

    xlr8@pop-os:~$ kubectl version
    WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the     full version.
    Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean",     BuildDate:"2023-04-14T13:21:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    Kustomize Version: v5.0.1
    Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.3", GitCommit:"9e644106593f3f4aa98f8a84b23db5fa378900bd", GitTreeState:"clean",     BuildDate:"2023-03-30T06:34:50Z", GoVersion:"go1.19.7", Compiler:"gc", Platform:"linux/amd64"}
    xlr8@pop-os:~$ kubectl get nodes
    NAME                 STATUS   ROLES           AGE     VERSION
    kind-control-plane   Ready    control-plane   5d17h   v1.26.3
    kind-worker          Ready    <none>          5d17h   v1.26.3
    kind-worker2         Ready    <none>          5d17h   v1.26.3
    xlr8@pop-os:~$

Remember one important thing here,

Kind provide minimalist version of kubernetes based on docker / podman / any contaiener runtime environment due to which we wont be able to expose our deployments to reach from laptop native web browser.

To achive this goal we are going to install MetalLB load balancer to reach our deployments, here are steps for that.

- Install metallb on your kubernetes cluster.
    kubectl create -f metallb-native.yaml
- Get your docker kind network range to update in metallb-config.yaml
    xlr8@pop-os:~$ docker inspect kind | grep -i subnet
                    "Subnet": "172.18.0.0/16",
- Now you need to update you metallb-config.yaml in our case it 172.18.0.0/16
    spec:
      addresses:
      - 172.18.0.10-172.18.0.20
- Now you ready to create your IP pool and attach it to L2Advertisement
    kubectl create -f metallb-config.yaml

Now you are ready to create your first deployment with Loadbalancer Service, we provided sample yaml template to create pods and expose them via Loadbalancer type.

    xlr8@pop-os:/media/xlr8/data/study/workspace/kind-kubernetes$ kubectl get svc | grep -i foo
    NAME                  TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    foo-service           LoadBalancer   10.96.124.13   172.18.0.13   5678:32383/TCP   9s
    xlr8@pop-os:/media/xlr8/data/study/workspace/kind-kubernetes$ kubectl get svc | grep foo
    foo-service           LoadBalancer   10.96.124.13   172.18.0.13   5678:32383/TCP   22s
    xlr8@pop-os:/media/xlr8/data/study/workspace/kind-kubernetes$ while true; do curl 172.18.0.13:5678; sleep 1; done
    bar
    bar
    foo
    foo
    bar
    bar
    foo
    ^C
    xlr8@pop-os:/media/xlr8/data/study/workspace/kind-kubernetes$ 

That's it you successfully deployed your kubernetes cluster and exposed it to external world.

Test it on your local env. and share your feedback too :)
