# k8s-homelab

This repository holds configuration for ArgoCD to administer a Kubernetes homelab.

While any k8s distribution should be fine, [Talos Linux](https://www.talos.dev) is recommended.  Specifically, documentation for a [simple cluster](https://www.talos.dev/v1.10/introduction/getting-started/) or a [production cluster](https://www.talos.dev/v1.10/introduction/prodnotes/) should be followed.

## Quickstart

Some charts depend on others.  You will need to set up ArgoCD, followed by cert-manager, and finally Traefik before you can deploy charts and securely expose services via Gateway API.

TODO: ArgoCD setup

## Ingress

Gateway API is used for this repository.

Kubernetes has traditionally used the Ingress API for exposing services; while the transition to Gateway API is underway, many Helm charts do not natively support it.  Instead, a simple template has been included with the charts in this repository to enable Gateway API in a straightforward way.

Key dependencies for exposing services via Gateway API are:
- Traefik
- cert-manager
- metallb

Here is the template for Gateway API:

```
{{- if .Values.gateway.enabled -}} # Check if gateway is enabled
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: {{ .Chart.Name }}-https-gateway
  namespace: {{ .Release.Namespace }}
spec:
  gatewayClassName: traefik
  listeners:
  - name: https
    protocol: HTTPS
    port: 8443
    hostname: {{ .Values.gateway.domain }}
    tls:
      mode: Terminate
      certificateRefs:
      - name: {{ .Chart.Name }}-tls
        namespace: {{ .Release.Namespace }}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: {{ .Chart.Name }}-tls
  namespace: {{ .Release.Namespace }}
spec:
  dnsNames:
  - {{ .Values.gateway.domain }}
  issuerRef:
    name: {{ coalesce .Values.gateway.issuer "cloudflare-issuer" }}
    kind: ClusterIssuer
  secretName: {{ .Chart.Name }}-tls
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: {{ .Chart.Name }}-https-route
  namespace: {{ .Release.Namespace }}
spec:
  parentRefs:
  - name: {{ .Chart.Name }}-https-gateway
    namespace: {{ .Release.Namespace }}
  hostnames:
  - {{ .Values.gateway.domain }}
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: {{ .Values.gateway.path }}
    backendRefs:
    - kind: Service
      name: {{ .Values.gateway.service }}
      namespace: {{ .Release.Namespace }}
      port: {{ .Values.gateway.port }}
{{- end -}} # End of the if condition
```

## Storage

The default storage class for this cluster is `nfs-subdir-external-provisioner`, which allows the use of NFS for provisioning PVCs.

You will need to set up an NFS routable from your Kubernetes cluster and override the appropriate values for the Helm chart.  Currently, the default values are:

```yaml
nfs-subdir-external-provisioner:
  nfs:
    path: /path/to/nfs
    server: 192.168.1.123
  storageClass:
    defaultClass: true
```

The `nfs.path` and `nfs.server` values can easily be set using ArgoCD.

To set up an NFS, there are widely available instructions (for example, from [Ubuntu](https://documentation.ubuntu.com/server/how-to/networking/install-nfs/)).

## OIDC/SSO

Authentik is used for OIDC/SSO.

## Services

Currently supported services include:
- ray
- valkey
- flink
- openwebui
