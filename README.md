# CZERTAINLY-argocd-demo

A repo for demonstrating troubles with deploing CZERTAINLY using argoCD.

Create App in argoCD:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: czertainly-demo
spec:
  destination:
    name: ''
    namespace: czertainly-demo
    server: 'https://kubernetes.default.svc'
  source:
    path: .
    repoURL: 'https://github.com/semik/CZERTAINLY-argocd-demo.git'
    targetRevision: HEAD
  sources: []
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true

```

Check App Parameters:
```
...
czertainly.messagingService.additionalVolumeMounts[0].mountPath /nic
czertainly.messagingService.additionalVolumeMounts[0].name      libsemik
czertainly.messagingService.additionalVolumes[0].name           libsemik
..
czertainly.pyAdcsConnector.additionalVolumeMounts[0].mountPath  /nic
czertainly.pyAdcsConnector.additionalVolumeMounts[0].name       libsemik
czertainly.pyAdcsConnector.additionalVolumes[0].name            libsemik
...
```

Check `pyadcs-connector-deployment`, examine `volumeMounts` section and note it contains mounting of directory `/nic`:
```yaml
          volumeMounts:
            - mountPath: /tmp
              name: ephemeral
            - mountPath: /etc/ssl/certs/pyadcs-trusted-cas.crt
              name: trusted-certificates-volume
              readOnly: true
              subPath: ca.crt
            - mountPath: /nic
              name: libsemik
```

Next check `messaging-statefulset` and again examine `messaging-statefulset` section and note that mounting of directory `/nic` is **missing**:
```yaml
          volumeMounts:
            - mountPath: /etc/rabbitmq
              name: configuration
            - mountPath: /var/lib/rabbitmq/mnesia
              name: data
              subPath: rabbitmq
            - mountPath: /tmp
              name: definitions
            - mountPath: /var/log/rabbitmq
              name: ephemeral
```

Both manifests for [pyadcs-connector-deployment](https://github.com/CZERTAINLY/CZERTAINLY-Helm-Charts/blob/2.14.0/charts/pyadcs-connector/templates/pyadcs-connector-deployment.yaml#L170) and [messaging-statefulset](https://github.com/CZERTAINLY/CZERTAINLY-Helm-Charts/blob/2.14.0/charts/messaging-rabbitmq/templates/messaging-statefulset.yaml#L249)
are using same call to [library](https://github.com/CZERTAINLY/CZERTAINLY-Helm-Charts/blob/2.14.0/charts/czertainly-lib/templates/_customizations.yaml#L8):
```yaml
{{- define "pyadcs-connector.customization.volumeMounts" -}}
{{- include "czertainly-lib.customizations.render.yaml" ( dict "parts" (list .Values.global.additionalVolumeMounts .Values.additionalVolumeMounts) "context" $ ) }}
{{- end -}}
```
respective:
```yaml
{{- define "messaging-rabbitmq.customization.volumeMounts" -}}
{{- include "czertainly-lib.customizations.render.yaml" ( dict "parts" (list .Values.global.additionalVolumeMounts .Values.additionalVolumeMounts) "context" $ ) }}
{{- end -}}
```
How is possible that it is working only for that deployment and not for statefulset?

How is possible that this is prefectly working when using `helm` on cmdline?
```
cat values.yaml | yq --yaml-output '.czertainly' > /tmp/values.yaml
helm -n czertainly-demo template czertainly-tlm oci://harbor.3key.company/czertainly-helm/czertainly --values=/tmp/values.yaml --version=2.14.0
...
# Source: czertainly/charts/pyAdcsConnector/templates/pyadcs-connector-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pyadcs-connector-deployment
...
          volumeMounts:
            - mountPath: /tmp
              name: ephemeral
            - name: trusted-certificates-volume
              mountPath: /etc/ssl/certs/pyadcs-trusted-cas.crt
              readOnly: true
              subPath: ca.crt
            - mountPath: /nic
              name: libsemik
...
# Source: czertainly/charts/messagingService/templates/messaging-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: messaging-statefulset
...
          volumeMounts:
            - name: configuration
              mountPath: "/etc/rabbitmq"
            - name: data
              mountPath: "/var/lib/rabbitmq/mnesia"
              subPath: rabbitmq
            - name: definitions
              mountPath: "/tmp"
            - mountPath: /var/log/rabbitmq
              name: ephemeral
            - mountPath: /nic
              name: libsemik
```
