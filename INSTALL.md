# Installation Guide — VM1 (Identity Manager)

**Host:** VM1 — `192.168.56.10` (lab host-only network)

## 1. OpenLDAP

- Base DN: `dc=securecorp,dc=local`
- Organizational units: IT, Dev, RH (HR)

```bash
# Organizational units, then users
ldapadd -x -D "cn=admin,dc=securecorp,dc=local" -W -f ous.ldif
ldapadd -x -D "cn=admin,dc=securecorp,dc=local" -W -f users.ldif

# Set a distinct password for each user (never stored in the repo)
ldappasswd -x -D "cn=admin,dc=securecorp,dc=local" -W -S "uid=alice,ou=IT,dc=securecorp,dc=local"
ldappasswd -x -D "cn=admin,dc=securecorp,dc=local" -W -S "uid=bob,ou=Dev,dc=securecorp,dc=local"
ldappasswd -x -D "cn=admin,dc=securecorp,dc=local" -W -S "uid=carol,ou=RH,dc=securecorp,dc=local"
```

### Enable LDAPS (port 636)

Place the certificate and key in `/etc/ldap/certs/` (the key readable by the
`openldap` user only), then:

```bash
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f tls.ldif
```

## 2. Keycloak 26

- Admin console: `http://192.168.56.10:8080` (lab network only)
- Realm: `securecorp`
- User federation: LDAP provider `openldap`
- Authentication: **TOTP required** as a second factor
- OIDC client: `securecorp-app`

## 3. Wazuh agent 4.7.5

Install the agent, copy [`ossec.conf`](ossec.conf) to `/var/ossec/etc/ossec.conf`
(manager on VM2), then:

```bash
sudo systemctl restart wazuh-agent
```

Monitored logs include `/var/log/auth.log` and the Keycloak log.

## 4. SSH

Copy [`sshd_config`](sshd_config) to `/etc/ssh/sshd_config`, validate it, then reload:

```bash
sudo sshd -t && sudo systemctl reload ssh
```
