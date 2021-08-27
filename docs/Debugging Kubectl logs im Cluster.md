Debugging Kubectl logs im Cluster

1. `yq e '.clusters.[0].cluster.certificate-authority-data' /home/subpath/Downloads/kubeconfig-admin-85mckh9xvn | base64 -d > cert.pem`
2. `curl --cacert cert.pem --header 'Authorization: Bearer g56pqk.dxxqkwgd9jkvd9sl' -i https://188.34.197.15:10250/containerLogs/kubeedge/cloudcore-774d78f6cb-pcqlf/cloudcore`
3. `kubectl run debug  --image debian:10 -n default -it --restart=Never -- bash` im Kubermatic base Cluster
	1. `apt update`
	2. `apt install -y curl nmap tshark`
	3. ``
