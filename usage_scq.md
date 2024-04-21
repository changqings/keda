## example usage by scq `2024-04-20`

if use metadata.password, it will not work, so use secret is a better and correct way.

## hpa use external
```bash
https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/external-metrics-api.md,`/apis/external.metrics.k8s.io/v1beta1/namespaces/<namespace_name>/<metric_name>?labelSelector=<selector>`
```
## 获取注册的external.metrics
```bash
shenchangqing@master:~$ kubectl get --raw '/apis/external.metrics.k8s.io/v1beta1/namespaces/default/s0-postgresql?labelSelector=scaledobject.keda.sh/name=nginx'
{"kind":"ExternalMetricValueList","apiVersion":"external.metrics.k8s.io/v1beta1","metadata":{},"items":[{"metricName":"s0-postgresql","metricLabels":null,"timestamp":"2024-04-20T08:04:30Z","value":"2"}]}
```
注册metrics服务
```yaml
shenchangqing@master:~$ kubectl get apiservices.apiregistration.k8s.io v1beta1.external.metrics.k8s.io -oyaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  annotations:
    cert-manager.io/inject-ca-from: keda/keda-operator-tls-certificates
    meta.helm.sh/release-name: keda
    meta.helm.sh/release-namespace: keda
  creationTimestamp: "2024-04-19T11:32:11Z"
  labels:
    app.kubernetes.io/component: operator
    app.kubernetes.io/instance: keda
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: v1beta1.external.metrics.k8s.io
    app.kubernetes.io/part-of: keda-operator
    app.kubernetes.io/version: 2.13.1
    helm.sh/chart: keda-2.13.2
  name: v1beta1.external.metrics.k8s.io
  resourceVersion: "948240"
  uid: bb949fdc-2e16-4df5-a17f-40090aa31a7d
spec:
  caBundle: <bas64 code>
  group: external.metrics.k8s.io
  groupPriorityMinimum: 100
  service:
    name: keda-operator-metrics-apiserver
    namespace: keda
    port: 443
  version: v1beta1
  versionPriority: 100
status:
  conditions:
  - lastTransitionTime: "2024-04-20T06:30:40Z"
    message: all checks passed
    reason: Passed
    status: "True"
    type: Available
```

```yaml
## create xx wiht per line k=v
## kubectl create secret generic pg-dev --from-env-file=xx
---
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: postgres-dev
spec:
  secretTargetRef:
    - parameter: userName
      name: pg-dev
      key: USER
    - parameter: password
      name: pg-dev
      key: PASSWORD
    - parameter: host
      name: pg-dev
      key: HOST
    - parameter: dbName
      name: pg-dev
      key: DB_NAME
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: nginx
  annotations:
spec:
  scaleTargetRef:
    name: nginx
  pollingInterval: 30 # Optional. Default: 30 seconds
  cooldownPeriod: 300 # Optional. Default: 300 seconds
  minReplicaCount: 1 # Optional. Default: 0
  maxReplicaCount: 4 # Optional. Default: 100
  triggers:
  - type: postgresql
    authenticationRef:
      name: postgres-dev
    metadata:
      sslmode: disable
      port: "5432"
      query: "SELECT value FROM public.keda_dev where id = 1;"
      targetQueryValue: "2"
```
