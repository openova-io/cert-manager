# ADR: TLS Certificates with cert-manager

**Status:** Accepted
**Date:** 2024-03-01

## Context

Need automated TLS certificate management for all services.

## Decision

Use cert-manager with Let's Encrypt HTTP-01 challenge.

## Rationale

| Option | Automation | Cost | Wildcard |
|--------|------------|------|----------|
| Manual certs | ❌ | $$$ | ✅ |
| **cert-manager + LE** | ✅ | $0 | ❌ | **Selected** |
| Commercial CA | ❌ | $$$ | ✅ |

### Why HTTP-01 over DNS-01?

| Challenge | Wildcards | DNS Provider | Complexity |
|-----------|-----------|--------------|------------|
| HTTP-01 | ❌ | None needed | Low |
| DNS-01 | ✅ | Required | High |

**Key Decision Factors:**
- Zero cost (Let's Encrypt)
- Automatic renewal
- No DNS provider integration needed
- Individual certs per service (no wildcards needed)

## Configuration

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@openova.io
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
      - http01:
          ingress:
            class: istio
```

## Certificate Template

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: <tenant>-tls
  namespace: <tenant>-prod
spec:
  secretName: <tenant>-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
    - api.<tenant>.io
    - app.<tenant>.io
```

## Renewal Policy

| Setting | Value |
|---------|-------|
| Validity | 90 days |
| Renewal | 30 days before expiry |
| Retry | Automatic |

## Consequences

**Positive:** Free certificates, automatic renewal, no DNS integration
**Negative:** No wildcards, HTTP-01 requires ingress exposure

## Related

- [SPEC-CERTIFICATE-CONFIGURATION](./SPEC-CERTIFICATE-CONFIGURATION.md)
