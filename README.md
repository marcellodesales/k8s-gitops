# NEEDS REWORK

# Prerequisites

Suitable rights to create resources
access token

# Create EKS Cluster
```
eksctl create cluster -f eksctl-cluster.yaml
```

# Enable GitOps

Integrate with FluxCD (https://fluxcd.io/)

Setup gitops repo 

```
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:robparrott/k8s-gitops.git
git push -u origin master
```

Enable ssh auth

```
ssh-keygen -f ~/.ssh/gitops-k8s
cat ~/.ssh/gitops-k8s.pub 

```

Add that key as a deploy key in GitHub or wherever.

Then enable gitops 
```
EKSCTL_EXPERIMENTAL=true eksctl \
    enable repo -f eksctl-cluster.yaml \
    --git-url=git@github.com:robparrott/k8s-gitops.git \
    --git-email=[ email ] \
    --git-private-ssh-key-path ~/.ssh/gitops-k8s
```

and confirm things are running.

You may need to get the newly created key and also add it as a deploy key in GitHub ... little flaky. Install `fluxcyl` and run `fluxctl identity --k8s-fwd-ns flux`. See https://docs.fluxcd.io/en/latest/tutorials/get-started.html#giving-write-access. YMMV.

# Scaling Manually

Determine the relevant nodegroup name (should be the name from the yaml file):
```
eksctl get nodegroup \
        --region=us-east-2 \
        --cluster=parrott-confluent-kafka
```

And then scale up or down:

```
eksctl scale nodegroup \
        --region=us-east-2 \
        --cluster=parrott-confluent-kafka \
        --name=general \
        --nodes=6
```

# Getting the cluster admin bearer token

Service Account "eks-admin" has a bearer token for access, so you need to dig that out to access Kubernetes Dashboard:

```
SECRET=$(kubectl -n kube-system get secret | grep eks-admin-token | awk '{print $1}')
TOKEN=$( kubectl -n kube-system describe secret ${SECRET} | grep "^token:" | awk '{print $2}' )
echo $TOKEN

```

# Tailing Logs 

You can get the `kail` tool to selectively tail logs from pods locally:

* https://github.com/boz/kail

# Forwarding Services:

Vault:

```
kubectl port-forward service/vault-server -n vault 8200 
```

Kubernetes Dashboard:

```
kubectl port-forward service/kube-dashboard-kubernetes-dashboard -n kube-system 8443:443 
```

Kibana:

```
kubectl port-forward service/kibana-kibana -n logging 5601
```

MySQL:

```
kubectl port-forward service/mysql-vault-integration-demo 3306 
```




