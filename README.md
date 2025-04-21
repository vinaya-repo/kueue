helm install kueue oci://registry.k8s.io/kueue/charts/kueue \
  --version=0.11.3 \
  --namespace  kueue-system \
  --create-namespace \
  --wait --timeout 300s \
  
  
Add args - '--feature-gates=ManagedJobsNamespaceSelector=true' in kueue deployment file (kueue-controller-manager)

kubectl -n kueue-system patch deployment kueue-controller-manager \
  --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--feature-gates=ManagedJobsNamespaceSelector=true"}]'



in kueue configmap (kueue-manager-config)

uncomment this section

managedJobsNamespaceSelector:
 matchExpressions:
   - key: kubernetes.io/metadata.name
     operator: NotIn
     values: [ kube-system, kueue-system ]


and also uncomment pod from this section

integrations:
  frameworks:
  - "batch/job"
  - "kubeflow.org/mpijob"
  - "ray.io/rayjob"
  - "ray.io/raycluster"
  - "jobset.x-k8s.io/jobset"
  - "kubeflow.org/paddlejob"
  - "kubeflow.org/pytorchjob"
  - "kubeflow.org/tfjob"
  - "kubeflow.org/xgboostjob"
  - "workload.codeflare.dev/appwrapper"
  - "pod"

****** Scale down and scale up deployment after config file changes, do not restart pod *******
create a namespace kueue-test -> Save namespace.yaml file

kubectl apply -f ClusterQueue.yaml -f LocalQueue.yaml -f ResourceFlavor.yaml
helm uninstall kueue -n kueue-system    