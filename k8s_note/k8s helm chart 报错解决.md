* jenkins 调用 helm 部署应用到 k8s 失败，提示 missing key，服务在 k8s 内实例已 uninstall </br>

Error: rendered manifests contain a resource that already exists. Unable to continue with install: Service "ocr-server" in namespace "qas" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "ocr-server"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "qas"

```
# helm version
version.BuildInfo{Version:"v3.2.0", GitCommit:"e11b7ce3b12db2941e90399e874513fbd24bcb71", GitTreeState:"clean", GoVersion:"go1.13.10"}
```


https://github.com/helm/helm-2to3/issues/147 </br>
https://github.com/helm/helm/pull/7649
