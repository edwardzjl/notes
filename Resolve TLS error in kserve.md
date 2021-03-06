# Resolve TLS error in kserve

Recently we experienced a wierd error, `kfserving-controller-manager-0` pod complains about `x509: certificate signed by unknown authority`.

According to [this comment](https://github.com/kserve/kserve/issues/1359#issuecomment-872931884) I found that the `caBundle` values in
`mutatingwebhookconfiguration` and `validatingwebhookconfiguration` were just the init value (`Cg==`), not equal to the ca.crt field of the `kfserving-webhook-server-cert` secret.

I don't know whether it's a bug or it's due to my re-deployment of `knative-serving` and `kfserving`.

Following that commit, I copied the caBundle value and fill it in `mutatingwebhookconfiguration` and `validatingwebhookconfiguration`, then the error message disappeared.

*!! IMPORTANT !!*

Do not download the `webhooks.yaml` template, as it may be incompatible with your version.

Use `kubectl edit` instead.
