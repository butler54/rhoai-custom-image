## Custom notebook images for Red Hat OpenShift AI (RHO-AI)

This repository provides a helm chart to build a custom notebook image into Red Hat OpenShift AI.
It's a simple demonstration of the art of the possible.
It uses imagebuild streams and build configurations to build a `Containerfile` image hosted in this repository.
Please be aware that you may need to trigger the build manually from the cluster console or with `oc`.


## Using this chart:
You can either run this as a helm install locally or using argocd. 

**IMPORTANT** this repo contains a self-reference in order to build the container. If you fork this repo you must change this reference.


### Local install
Quickest way to get going 

```shell
oc login ...  ## Login to Openshift with rights to the redhat-ods-applications namespace
cd rhoai-custom-image
helm install ./helm
```

The container file should be built within a few minutes from first install.

### With OpenShift GitOps (ArgoCD)

- [Install OpenShift GitOps](https://docs.openshift.com/gitops/1.12/installing_gitops/installing-openshift-gitops.html) in the `openshift-gitops` namespace (the default).

- Deploy the application to target the helm chart. For example if using the `argocd` `cli` for this repository:

```shell
argocd app create rhoai-custom-image \
    --repo https://github.com/butler54/rhaoi-custom-image \
    --path helm \
    --revision main \
    --dest-server  https://kubernetes.default.svc \
    --dest-namespace redhat-ods-applications \
    --directory-recurse \
    --sync-policy automated \
    --self-heal \
    --sync-option Prune=true
```


### Changing the base image
In order to work with offline environments (and remove credential barriers), the helm chart refers to the images streams which RHO-AI creates.
You can customize which stream and version is used by this chart.

To find out what is available on your OpenShift cluster

```
oc get imagestream -n redhat-ods-applications
NAME                                IMAGE REPOSITORY                                                                                                            TAGS                                   UPDATED
code-server-notebook                default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/code-server-notebook                2023.2,2024.1                          27 hours ago
cuda-rhel9                          default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/cuda-rhel9
cuda-rstudio-rhel9                  default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/cuda-rstudio-rhel9
habana-notebook                     default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/habana-notebook                     2023.2,2024.1                          27 hours ago
minimal-gpu                         default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/minimal-gpu                         1.2,2023.1,2023.2,2024.1               27 hours ago
odh-trustyai-notebook               default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/odh-trustyai-notebook               2023.1,2023.2,2024.1                   27 hours ago
pytorch                             default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/pytorch                             1.2,2023.1,2023.2,2024.1               27 hours ago
rstudio-rhel9                       default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/rstudio-rhel9
s2i-generic-data-science-notebook   default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/s2i-generic-data-science-notebook   1.2,2023.1,2023.2,2024.1               27 hours ago
s2i-minimal-notebook                default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/s2i-minimal-notebook                1.2,2023.1,2023.2,2024.1               27 hours ago
tensorflow                          default-route-openshift-image-registry.apps.myclusterdomain.com/redhat-ods-applications/tensorflow                          1.2,2023.1,2023.2,2024.1               27 hours ago
```

Use the name and the tags and change in `helm/values.yaml`. Note *latest* will not work on Red Hat provided charts.


## Forcing an image build
If you need to force an image build the following command wil kick one off:

```oc start-build -n redhat-ods-applications custom-image-build```