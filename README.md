# SecureCorp — Zero Trust IAM · Identity Layer (VM1)

Configuration of the **identity layer** of *SecureCorp*, an academic Zero Trust IAM
architecture built by a team of four. This repository contains **my part of the
project — VM1**: the central directory, the identity provider and its monitoring.

▶️ **Demo video (4 test scenarios):** https://www.youtube.com/watch?v=MxU6_hBwiV4

---

## The architecture

SecureCorp applies the *Never Trust, Always Verify* principle across three virtual
machines. Every request follows the same path, whatever its origin:

```
User → Nginx (gateway) → Keycloak (SSO + MFA) → RBAC → Vault (secrets) → Resource (Guacamole)
                                   │
                              OpenLDAP (LDAPS)          All access logs → Wazuh SIEM
```

| VM | Responsibility | Components |
|---|---|---|
| **VM1 — this repo** | **Identity management** | **OpenLDAP (LDAPS), Keycloak, Wazuh agent** |
| VM2 | Secrets & SIEM | HashiCorp Vault, Wazuh manager |
| VM3 | Secure access gateway | Nginx, Apache Guacamole |

Brute-force attempts are blocked automatically by Fail2Ban.

## My part — VM1

| Component | Version | Role |
|---|---|---|
| OpenLDAP | — | Central identity directory, exposed over **LDAPS (636)** only |
| Keycloak | 26.2.4 | Identity provider: **LDAP user federation**, **OIDC SSO**, **TOTP MFA** |
| Wazuh agent | 4.7.5 | Ships identity-layer logs (auth, Keycloak) to the SIEM on VM2 |

- **Directory** — `dc=securecorp,dc=local`, one organizational unit per department
  (IT, Dev, HR), the structure the RBAC policies rely on for least privilege.
- **Identity provider** — `securecorp` realm federated with OpenLDAP, an OIDC client
  for the application, and TOTP required as a second factor.
- **Monitoring** — the Wazuh agent collects `/var/log/auth.log` and the Keycloak log,
  so failed logins and identity events reach the SIEM in real time.
- **SSH hardening** — no legacy SHA-1 algorithms, no root login, limited auth tries.

## Repository contents

| File | Purpose |
|---|---|
| [`ous.ldif`](ous.ldif) | Organizational units: IT, Dev, HR |
| [`users.ldif`](users.ldif) | One test user per department — **no passwords stored** |
| [`tls.ldif`](tls.ldif) | Enables TLS on OpenLDAP (LDAPS) |
| [`ossec.conf`](ossec.conf) | Wazuh agent configuration |
| [`sshd_config`](sshd_config) | Hardened SSH server configuration |
| [`INSTALL.md`](INSTALL.md) | Step-by-step setup of VM1 |

## Test scenarios

Validated by the team in the demo video:

1. **Legitimate access** — full chain Nginx → Keycloak (SSO/MFA) → RBAC → Vault.
2. **Access segregation** — IT, Dev and HR users are denied resources outside their role.
3. **External attack** — intrusion detected by Wazuh, blocked by Fail2Ban, analyzed with a honeypot.
4. **Disaster recovery** — Vault secrets backup and restore after a simulated crash.

## Security notes

- This is a **lab environment** on a private host-only network (`192.168.56.0/24`).
- **No secrets are committed**: LDAP passwords are set after import (see `INSTALL.md`),
  and the TLS private key stays on the server.
- Keycloak listens on port 8080 inside the lab network only; users reach it through
  the Nginx gateway on VM3.

---

*Academic project — École Polytechnique de Sousse. Team of four; VM2 and VM3 were
built by my teammates and are not included here.*
