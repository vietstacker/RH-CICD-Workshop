# RH-CICD-Workshop

This repo is designed to demo an end-to-end flow of cloud-native release cycle development using Red Hat Openshift, K8S-native CI as Red Hat Pipeline (Tekton), K8S-native CD as Argo and other Red Hat tools such as buildah, skopeo, etc.

### Instruction to prepare

1. Environment preparation

- Clone this repo for ArgoCD deployment and its application: https://github.com/vietstacker/RH-Helloworld-ops
- Clone this repo for creation of Tekton objects such as pipeline, tasks, etc.: https://github.com/vietstacker/RH-CICD-Workshop

2. Creating a namespace for CI/CD

```bash
cd ~/RH-Helloworld-ops
oc apply -k config/cicd_env
```

3. Running ArgoCD

```bash
cd ~/RH-Helloworld-ops
oc apply -k argo/operator_and_subscription
```

- Wait for ArgoCD operator to be subscribed and available
```bash
oc get pods -n argocd

NAME                                             READY   STATUS              RESTARTS   AGE
argocd-operator-6d5d7bf9c5-kppd6                 1/1     Running             0          4d18h
```
- Create an ArgoCD instance and assign the cluster-admin role to its serviceaccount

```bash
oc apply -k argo/infrastructure
oc adm policy add-cluster-role-to-user cluster-admin -n argocd -z argocd-application-controller
```

- Check the ArgoCD up and running

```bash
oc get pods -n argocd

NAME                                             READY   STATUS    RESTARTS   AGE
argocd-application-controller                    1/1     Running   0          1m
argocd-dex-server                                1/1     Running   0          1m
argocd-redis                                     1/1     Running   0          1m
argocd-repo-server                               1/1     Running   0          1m
argocd-server                                    1/1     Running   0          1m
```

- Create an CICD environment

```bash
cd ~/RH-Helloworld-ops
oc apply -k config/cicd_env
```


- Deploy Argo applications that are test enviroment and production environment

```bash
cd ~/RH-Helloworld-ops
oc apply -f deployments/build_and_test/argo-application.yaml
oc apply -f deployments/rh-helloworld-prod/argo-application.yaml
```

4. Running Tekton

- Access to OperatorHub on Openshift, search for Tekton and install Red Hat Pipeline.
- Provide pipeline ServiceAccount the admin role in the rh-helloworld-ci namespace: 

```bash
oc adm policy add-cluster-role-to-user cluster-admin -n rh-helloworld-ci -z pipeline
```

5. Change the password of ArgoCD

```bash
cd ~/RH-CICD-Workshop/secrets
sed -i -e  "s/ARGOCD_PASSWORD:.*/ARGOCD_PASSWORD: $(oc get secret argocd-cluster -n argocd -o jsonpath='{.data.*}')/" argocd-secret.yaml

```

### Running demo
1. Creating Pipeline objects:
```bash
cd ~/RH-CICD-Workshop
oc apply -k secrets -n rh-helloworld-ci
oc apply -k . -n rh-helloworld-ci
```

2. Running an end-to-end "build, test" on a CICD environment and then "deploy" on a production environment
```bash
cd ~/RH-CICD-Workshop
oc apply -f pipelines-run/build-helloworld.yaml
```
