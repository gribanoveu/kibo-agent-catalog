# A TLS certificate is rejected

Sign: "x509: certificate has expired or is not yet valid", "certificate signed by unknown authority", "tls: bad certificate", or the ingress serving its default fake certificate.

1. Which side refuses: the caller's log names the host it was calling (kubeLogs, grep x509 or tls).
   - The caller is in the cluster and calls another service → the callee's certificate, or the caller's trust store.
   - Users see it in a browser → the ingress certificate.
2. The ingress certificate: kubeGet Ingress — spec.tls lists the hosts and the secretName.
   - No tls entry for the host → the controller serves its default self-signed certificate.
   - kubeGet Secret <secretName>: it must exist in the Ingress's namespace and hold tls.crt and tls.key.
3. cert-manager: kubeList certificates in the namespace; kubeGet the Certificate — status.conditions
   Ready and status.notAfter. Not Ready → kubeList certificaterequests and orders, kubeEvents on them;
   the reason is there (a failed HTTP-01 challenge is usually an ingress or DNS problem).
4. Unknown authority on an internal call → the caller does not trust the issuing CA: a mounted CA bundle
   (ConfigMap or Secret in the pod spec) missing or outdated; a JVM needs it in its truststore.
5. Do not turn verification off (insecureSkipVerify, -k) as a fix — say what to renew or mount instead.
