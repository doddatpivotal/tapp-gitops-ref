# tapp-gitops-ref


export CFG_r53__access_key=<AWS ACCESS KEY>
export CFG_r53__secret_key=<AWS_SECRET_KEY>

kapp deploy -a tap-install-gitops -f <(ytt -f gitops)
