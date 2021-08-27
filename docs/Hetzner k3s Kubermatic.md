Hetzner k3s Kubermatic

## Installation HA-k3s
1. Add SSH Zugriff
2. install ruby (https://www.ruby-lang.org/en/documentation/installation/)
3. gem install hetzner-k3s (wie in der Doku beschrieben: https://github.com/vitobotta/hetzner-k3s#Installation)
4. gem install -i ./ hetzner-k3s (PWD ist ein passendes (leeres) Directory) (failed)
5. sudo gem install hetzner-k3s
6. Installation des Cluster, nutzen der Config:
```
---
hetzner_token: Vz1evJksjDLFXiz8DUb5zIakl3kePpqa6ZZbSO5LLOR95NmllN6TVzCoPSaBtXQk
cluster_name: kubeedge
kubeconfig_path: "./kubeconfig"
k3s_version: v1.21.3+k3s1
ssh_key_path: ~/.ssh/id_juw1.pub
verify_host_key: false
location: nbg1
masters:
  instance_type: cpx21
  instance_count: 3
worker_node_pools:
- name: small
  instance_type: cpx21
  instance_count: 3

```
in `hetzner-k3s create-cluster --config-file config.yaml`


## Probleme
- Serverlimit nach 5 Instanzen erreicht
- `ssh`  ist abgebrochen -> k3s Installation auf allen Systemen per Hand
	- liegt vermutlich direkt an der eingebundenen Lib
	- https://github.com/vitobotta/hetzner-k3s/issues/8

## Setting UP k3s HA per Hand mit Embedded DB
- erstellen eines Scriptes zur installation
- da k3s noch Probleme hat mit dem internen ETCD hat
	- nutzung einer externen DB
	- Dokumentation: https://rancher.com/docs/k3s/latest/en/installation/ha/
	- https://github.com/k3s-io/k3s/issues/2850


-> kein HA setup

## Setup Kubermatic
1. Konfiguration, wie beschrieben
	- Password ``
2.  Ausfüren des Installers:
 ```
./kubermatic-installer deploy --config examples/kubermatic.example.ce.yaml --helm-values examples/values.example.yaml --storageclass hetzner --kubeconfig ~/.kube/config
```
3. Load Balancer wird nicht fertig -> erstellen eines Loadbalancers und der Labels
	```
	kubectl annotate service/nginx-ingress-controller -n nginx-ingress-controller load-balancer.hetzner.cloud/location=ngb1
	```
	dabei sollte ngb1 durch das entsprechende kürzel des Datenzenters ersetzt werden:
	- nbg1 == Nürnberg
	- fsn1 == Falkenstein/Vogtland
	- hel1 == Tuusula/Helsinki
4. Adding Hetzner Cloud-Controller-Manager (https://github.com/hetznercloud/hcloud-cloud-controller-manager)
 ```
	kubectl -n kube-system create secret generic hcloud --from-literal=token=<token>
	kubectl apply -f  https://github.com/hetznercloud/hcloud-cloud-controller-manager/releases/latest/download/ccm.yaml
```
ACHTUNG die die Firewall blockiert Zugriffe auf die benötigten Ports und es wird die Public IP-Adresse verwendet -> die NodePorts müssen explizit freigeschaltet werden!

### Adding Seed Cluster
- erstellen einer Seed Konfiguration
```
apiVersion: v1
kind: Secret
metadata:
  name: kubeconfig-cluster-example
  namespace: kubermatic
type: Opaque
data:
  kubeconfig: <base64 encoded kubeconfig>
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

  # List of datacenters where this seed cluster is allowed to create clusters in
  # In this example, user cluster will be deployed in eu-central-1 on AWS.
  datacenters:
    hetzner-nbg1: 
        location: Nueremberg
        country: DE
        spec:
          hetzner:
            datacenter: nbg1-dc5  

  # reference to the kubeconfig to use when connecting to this seed cluster
  kubeconfig:
    name: kubeconfig-cluster-example
    namespace: kubermatic

```
- einfïugen der kubeconfig, welche mit `base64 -w0` erstellt wurde
- erstellen eines Secrets mit dem Token und einfügen des csi-drivers: https://github.com/hetznercloud/csi-driver
- testen des drivers, wie beschrieben:
	- war erfolgreich

### Erstellen user Cluster
- durchcklicken durch die API + ACHTUNG bei einem Schritt wird das Token abgerufen, dieses muss per hand gesetzt werden (hetzner token)

## Deploy Cloudcore
- Untaint worker node:
 ```
 kubectl taint node kubeedge-cpx21-pool-small-worker1 node.cloudprovider.kubernetes.io/uninitialized=true:NoSchedule-
```
- Nutzung des Helm Charts aus kubeedge-on-aks repo und der annahme von einem lokalen helm v3
- Nutzung des NodePort anstatt LoadBalancer im Service
- `helm install -n kubeedge kubeedge ./kubeedge`
- Testen ob erfolgreich:
```
kubectl logs -n kubeedge cloudcore
```
- auslesen der Ports:
```
kubectl get svc -n kubeedge cloudcore

NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                           AGE
cloudcore   NodePort   10.43.55.163   <none>        10002:30002/TCP,10000:30000/TCP   30s
```
- Port 10002 wird auf 100000 und 30002 auf 30000 gemapt


### IP-Tables Regeln:
- `iptables -t nat -A OUTPUT -p tcp --dport 10350 -j DNAT --to 188.34.196.240:10350`
- https://kubeedge.io/en/docs/setup/keadm/
- kubectl get cm tunnelport -nkubeedge -oyaml
- Pod: apiserver-*-* Shell Exec auf DNAT-Cotnainer oder OPENVPN

## Starten Edge
