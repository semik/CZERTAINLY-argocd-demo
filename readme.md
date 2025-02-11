# when defined and equal false, nothing is shown

```
semik@semik:~/3K/CZERTAINLY-argocd-demo/czertainly [develop|●5✚ 1…5]✘-1$ helm dependency update --skip-refresh; helm dependency build --skip-refresh;  helm install --set nic.nikde=false --debug --dry-run xx . |  grep -B 6 -A 1 'semik uz'
Saving 2 charts
Downloading czertainly from repo oci://harbor.3key.company/czertainly-helm
Pulled: harbor.3key.company/czertainly-helm/czertainly:2.14.0
Digest: sha256:f5734f7fc16805488c8715ae914c2af417ffb670235b5432a98e597883032c74
Deleting outdated charts
Saving 2 charts
Downloading czertainly from repo oci://harbor.3key.company/czertainly-helm
Pulled: harbor.3key.company/czertainly-helm/czertainly:2.14.0
Digest: sha256:f5734f7fc16805488c8715ae914c2af417ffb670235b5432a98e597883032c74
Deleting outdated charts
install.go:225: 2025-02-11 12:15:21.673384904 +0100 CET m=+0.036261213 [debug] Original chart version: ""
install.go:242: 2025-02-11 12:15:21.673454372 +0100 CET m=+0.036330676 [debug] CHART PATH: /home/semik/3K/CZERTAINLY-argocd-demo/czertainly
```

# when defined and equal true, expected manifest is added