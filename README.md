# Cloud Native Toolkit Deployment Guides

## Apply demo sealedsecret key to all clusters
Download the private key [sealed-secrets-ibm-demo-key.yaml](https://bit.ly/demo-sealed-master) used to seal any secret contained in this demonstration and apply it to the cluster.
```
oc apply -f sealed-secrets-ibm-demo-key.yaml
```
If you have the Sealed Secrets Operator already running in your cluster, restart it so that the private key that was used to seal the secrets used for this demonstration is picked up by the Sealed Secrets Operator. This will result in the Sealed Secrets Operator being able to decrypt any sealed secret included in this demonstration.
```
oc delete pod -n sealed-secrets -l app.kubernetes.io/name=sealed-secrets
```
**IMPORTANT: DO NOT CHECK INTO GIT**. The private key **MUST NOT** be checked into GitHub under any circumstances. Please, remove the private key from your workstation to avoid any issues.
```
rm sealed-secrets-ibm-demo-key.yaml
```

For the Cloud Pak Platform Navigator to be able to consume the IBM Entitlement Key we are providing in this demonstration and, as a result, be able to download IBM software in the form of official IBM Docker Images, restart the Cloud Pak Platform Navigator pods.
```
oc delete pod -n tools -l app.kubernetes.io/name=ibm-integration-platform-navigator
```


## Install OpenShfit GitOps (ArgoCD)
To get started setup gitops operator and rbac on each cluster

- For OpenShift 4.7+ use the following:
  ```
  oc apply -f setup/ocp47/
  while ! kubectl wait --for=condition=Established crd applications.argoproj.io; do sleep 30; done
  ```
  Once ArgoCD is deploy get the `admin` password
  ```
  oc extract secrets/openshift-gitops-cluster --keys=admin.password -n openshift-gitops --to=-
  ```

- For OpenShift 4.6 use the following:
  ```
  oc apply -f setup/ocp46/
  while ! kubectl wait --for=condition=Established crd applications.argoproj.io; do sleep 30; done
  ```
  Once ArgoCD is deploy get the `admin` password
  ```
  oc extract secrets/argocd-cluster-cluster --keys=admin.password -n openshift-gitops --to=-
  ```

## Install the ArgoCD Application Bootstrap
Apply the bootstrap profile, to use the default `single-cluster` scenario use the following command:
```
oc apply -n openshift-gitops -f 0-bootstrap/argocd/bootstrap.yaml
```

For other profile clusters set environment variable `TARGET_CLUSTER` then apply the profile

**shared-cluster**:
```
TARGET_CLUSTER=0-bootstrap/argocd/others/1-shared-cluster/bootstrap-cluster-1-cicd-dev-stage-prod.yaml

TARGET_CLUSTER=0-bootstrap/argocd/others/1-shared-cluster/bootstrap-cluster-n-prod.yaml
```
Now apply the profile
```
echo TARGET_CLUSTER=${TARGET_CLUSTER}
oc apply -n openshift-gitops -f ${TARGET_CLUSTER}
```


This repository shows the reference architecture for gitops directory structure for more info https://cloudnativetoolkit.dev/learning/gitops-int/gitops-with-cloud-native-toolkit