# Prerequisites

- K8s cluster up and ready
- Helm installed
- Elasticsearch

# Demo
For demo purpose, we intentionlally use one node k8s cluster and simplify Elasticsearch resourses, but take into account acrhitecture details:
- Deploy all via Helm
- Parity dev/test/prod environments

For demo purpose, one can install k8s cluster 1.32 + Containerd + IPVS + Flannel

# Enable deploy 
```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

# Add non-productino PV dynamic provisioning  
```
helm repo add openebs https://openebs.github.io/openebs
helm repo update

mkdir -p /var/elk_data

helm install openebs openebs/openebs -n openebs --create-namespace \
 --set localpv-provisioner.hostpathClass.basePath=/var/elk_data

kubectl get pods -n openebs -l openebs.io/component-name=openebs-localpv-provisioner
kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

# Install Elasticsearch via official Helm chart
```
helm repo add elastic https://helm.elastic.co
helm repo update

cat <<EOF > values.yaml
resources:
  requests:
    cpu: "100m"
    memory: "100Mi"
  limits:
    cpu: "1000m"
    memory: "2Gi"

antiAffinity: "soft"
EOF

helm install elasticsearch elastic/elasticsearch -f values.yaml
```
 
