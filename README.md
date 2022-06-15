# tapp-gitops-ref

## Export the following vars

export CFG_r53__access_key=<AWS ACCESS KEY>
export CFG_r53__secret_key=<AWS_SECRET_KEY>

export CFG_tap__tanzunet__username=tnu
export CFG_tap__tanzunet__password=tnp
export CFG_tap__registry__username=tnr
export CFG_tap__registry__password=tnp
export CFG_tap__supplychainregistry__username=scu
export CFG_tap__supplychainregistry__password=scp

## Create the app

ytt -f gitops/common -f gitops/appref1 --data-values-env CFG --data-values-file=config-appref1.yaml

## Setup dev workspace

cat <<EOF | kubectl -n default apply -f -

apiVersion: v1
kind: Secret
metadata:
  name: tap-registry
  annotations:
    secretgen.carvel.dev/image-pull-secret: ""
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: e30K

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
secrets:
  - name: registry-credentials
imagePullSecrets:
  - name: registry-credentials
  - name: tap-registry

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: default
rules:
- apiGroups: [source.toolkit.fluxcd.io]
  resources: [gitrepositories]
  verbs: ['*']
- apiGroups: [source.apps.tanzu.vmware.com]
  resources: [imagerepositories]
  verbs: ['*']
- apiGroups: [carto.run]
  resources: [deliverables, runnables]
  verbs: ['*']
- apiGroups: [kpack.io]
  resources: [images]
  verbs: ['*']
- apiGroups: [conventions.apps.tanzu.vmware.com]
  resources: [podintents]
  verbs: ['*']
- apiGroups: [""]
  resources: ['configmaps']
  verbs: ['*']
- apiGroups: [""]
  resources: ['pods']
  verbs: ['list']
- apiGroups: [tekton.dev]
  resources: [taskruns, pipelineruns]
  verbs: ['*']
- apiGroups: [tekton.dev]
  resources: [pipelines]
  verbs: ['list']
- apiGroups: [kappctrl.k14s.io]
  resources: [apps]
  verbs: ['*']
- apiGroups: [serving.knative.dev]
  resources: ['services']
  verbs: ['*']
- apiGroups: [servicebinding.io]
  resources: ['servicebindings']
  verbs: ['*']
- apiGroups: [services.apps.tanzu.vmware.com]
  resources: ['resourceclaims']
  verbs: ['*']
- apiGroups: [scanning.apps.tanzu.vmware.com]
  resources: ['imagescans', 'sourcescans']
  verbs: ['*']

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: default
subjects:
  - kind: ServiceAccount
    name: default

EOF
export CFG_tap__registry__username=tap
export CFG_tap__registry__password=VMware*2

tanzu secret registry add registry-credentials --server harbor.one.appref.end2end.link --username $CFG_tap__registry__username --password $CFG_tap__registry__password --namespace default --export-to-all-namespaces

## don't really want to export to all namespaces,  but see ; need to identify solution

https://vmware.slack.com/archives/C02D60T1ZDJ/p1641505166079800


tanzu apps workload create tanzu-java-web-app \
   --git-repo https://github.com/jeffellin/tanzu-java-web-app \
   --git-branch main \
   --type web \
   --label app.kubernetes.io/part-of=tanzu-java-web-app \
   --label apps.tanzu.vmware.com/has-tests=true \
   --yes \
   --namespace default