# tapp-gitops-ref


export CFG_r53__access_key=<AWS ACCESS KEY>
export CFG_r53__secret_key=<AWS_SECRET_KEY>

export CFG_tap__tanzunet__username=tnu
export CFG_tap__tanzunet__password=tnp
export CFG_tap__registry__username=tnr
export CFG_tap__registry__password=tnp
export CFG_tap__supplychainregistry__username=scu
export CFG_tap__supplychainregistry__password=scp

ytt -f gitops/common -f gitops/appref1 --data-values-env CFG --data-values-file=config-appref1.yaml