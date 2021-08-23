# kubermatic-hetzner

This repository is used to create a kuberentes cluster with terraform and kubeon. After the creation kubermatic will be installed in this cluster.

## Table of Contents

- [Dependencies](#dependencies)
- [Setting Up](#setting-up)
  - [Create Ressources](#create-ressources)
  - [Install Kubernetes](#install-kubernetes)
  - [Install Kubermatic](#install-kubermatic)
- [Scale Base Cluster](#scale-base-cluster)
- [Seed Cluster](#seed-cluster)

## Dependencies

- terraform
- kubeon
- kubermatic

## Setting Up

### Create Ressources

The creation of the ressources will be made with terraform. After adding the Hetzner token in the env variable HCLOUD\_TOKEN you can use the following command to create all the ressources.

```bash
cd terraform
terraform apply -var ssh_public_key_file=~/.ssh/id_juw1.pub -var cluster_name=kubermatic
```

With the variable ssh_public_key_file you can specify the used ssh key file. Be careful it is not allowed, that the key file already exists in the hetzner system.

### Install Kubernetes

To Install kubernetes (k8s) we are using kubeon. To use this we have to crate the configuration file. An example confiugration file can be found in the kubeon folder. The following commands can be used to install the kubernetes cluster.

```bash
cd terraform 
terrafrom output --json > output.json
cd ..
kubeon apply --manifest kubeone/kubeone.yaml --tfjson output.json
```

### Install Kubermatic

To Install kubermatic you have to configure the files `kubermatic.yaml` and `values.yaml`. Examples of those files can be found in the directory `kubermatic/examples/`.

To install kubermatic, you can use the following command:

```bash
export KUBECONFIG=$(realpath kubermatic-kubeconfig)
cd kubermatic
./kubermatic-installer deploy --config kubermatic.yaml --helm-values values.yaml --storageclass hetzner --kubeconfig ../kubermatic-kubeconfig
```

To enable the creation of the load balancer, you have to set the location, where the load balancer is located.
You can use the following command set that this to Nuremberg.

```bash
kubectl annotate service/nginx-ingress-controller -n nginx-ingress-controller load-balancer.hetzner.cloud/location=nbg1
```

If you have any issues by executing the above command, you can enable the debug log by using the follwing command:

```bash
./kubermatic-installer -v deploy --config kubermatic.yaml --helm-values values.yaml --storageclass hetzner --kubeconfig ../kubermatic-kubeconfig
```

## Scale Base Cluster

When you have Problems by deploying dex or another programm, the issue can be, that not enough resources can be used in the cluster. In the setup we have used kubeone to setup our cluster.
Because we have used that, we can scale the number of nodes in a node pool with kubectl. The next command shows, how this can be made:

```bash
kubectl -n kube-system scale machinedeployments.cluster.k8s.io kubermatic-pool1 --replicas=2
```

The node pool has the name kubermatic-pool1 in this case and the number of worker nodes will now change to two.

After adding an new worker different pods uses the public ip address of the server. The pods are all located in the kube-system and matching the regex: `^canal*|^kube-proxy*|^node-local*`
My solution was to restart the pods and they using the correct IP-Address, which is a private network in hetzner.

## Seed Cluster

The seed cluster contains the kubernetes api server, the etcd and the control plan of the user clusters. As a seed cluster you can add the base cluster or a different manual created cluster.
In this description we are using the base cluster as the seed cluster.

the output of the following command should be placed in the field kubeconfig of the secret. The secret an the seed must be added in one time. I had big trubbles, when I done this in two steps.

```bash
cd kubermatic
cat ../kubermatic-kubeconfig | base64 -w 0 
```

With the exmaple yaml above you create a seed cluster, which can be used to use a seed cluster by hetzner.

```yaml
apiVersion: v1
kind: Secret
metadata:
   name: kubeconfig-cluster-example
   namespace: kubermatic
type: Opaque
  data:
    kubeconfig: <kubeconfig>
---
apiVersion: kubermatic.k8s.io/v1
kind: Seed
metadata:
  name: kubermatic
  namespace: kubermatic
spec:
  # these two fields are only informational
  country: DE
  location: Nueremberg

  datacenters:
    hetzner-nbg1:
      location: Nuremberg
      country: DE
      spec:
        hetzner:
          datacenter: nbg1-dc5
          network: kubermatic

  kubeconfig:
    name: kubeconfig-cluster-example
    namespace: kubermatic
```

In this example I'm using the network kubermatic, which is also used by the base cluster.
