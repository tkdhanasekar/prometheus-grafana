Here’s a structured list of **MariaDB Administration & Management Topics** you can use as a roadmap or reference for database learning and operations:

---

## 🔹 MariaDB Administration & Management Topics

### 1. **Introduction to MariaDB**

* What is MariaDB (history, relation to MySQL)
* Use cases: OLTP, web applications, analytics
* MariaDB architecture overview (clients, server, storage engines)

---

### 2. **Installation & Setup**

* Installing MariaDB (Debian/Ubuntu, RHEL/CentOS, source)
* Initial configuration (`my.cnf`)
* Running MariaDB as a service (`systemctl`, `service`)
* Securing installation (`mysql_secure_installation`)

---

### 3. **User & Access Management**

* Creating users and assigning passwords
* GRANT and REVOKE privileges
* Role-based access control (RBAC)
* Authentication plugins (PAM, LDAP, socket)
* Password policies (expiry, strength, rotation)

---

### 4. **Database & Schema Management**

* Creating and dropping databases
* Managing tables (create, alter, drop)
* Data types and constraints
* Views, stored procedures, functions, triggers, events

---

### 5. **Storage Engines**

* Default storage engines (InnoDB, Aria)
* MyISAM vs InnoDB differences
* Choosing the right storage engine
* Engine-specific tuning options

---

### 6. **Configuration Management**

* MariaDB configuration file locations (`/etc/mysql/my.cnf`, `/etc/my.cnf.d/`)
* Important parameters: buffer pool, cache, max connections, query cache
* Performance tuning with `mysqltuner`
* Dynamic vs static configuration changes

---

### 7. **Backup & Restore**

* Logical backups (`mysqldump`, `mariadb-dump`)
* Physical backups (file copy, Percona XtraBackup, Mariabackup)
* Point-in-time recovery with binary logs
* Automating scheduled backups

---

### 8. **Replication & Clustering**

* Master-Slave replication
* Semi-synchronous replication
* Multi-source replication
* MariaDB Galera Cluster (multi-master synchronous replication)
* Monitoring and troubleshooting replication

---

### 9. **Security Management**

* Securing MariaDB with firewalls (bind-address, TLS/SSL)
* Encryption at rest and in transit
* Preventing SQL injection (application-level)
* User privilege audits
* Security best practices

---

### 10. **Performance Monitoring & Optimization**

* Query performance analysis (`EXPLAIN`, `ANALYZE`)
* Slow query log
* Index management (BTREE, FULLTEXT, HASH)
* Buffer pool and cache optimization
* Locking and transaction isolation levels
* Optimizing joins and subqueries

---

### 11. **High Availability & Scalability**

* Load balancing with ProxySQL or MaxScale
* Failover strategies (automatic/manual)
* Sharding basics in MariaDB
* Replication + clustering for scalability

---

### 12. **Logging & Monitoring**

* Error log, general query log, slow query log
* Performance Schema / Information Schema
* Monitoring tools (phpMyAdmin, MySQL Workbench, Grafana, Percona Monitoring)

---

### 13. **Maintenance & Troubleshooting**

* Checking server status (`SHOW STATUS`, `SHOW PROCESSLIST`)
* Repairing corrupted tables (`CHECK TABLE`, `REPAIR TABLE`)
* Managing disk space (table optimization, archiving)
* Common errors (authentication, replication lag, lock contention)

---

### 14. **Advanced Topics**

* Partitioning tables
* Virtual columns and JSON support
* Window functions and CTEs
* Full-text search
* MariaDB plugins and extensions
* Migrating from MySQL to MariaDB

---

### 15. **Best Practices**

* Principle of least privilege for users
* Regular backups and restore testing
* Monitor slow queries and optimize
* Use InnoDB as default storage engine
* Keep MariaDB updated with security patches

---

Perfect! Let’s expand **SSL/TLS Certificate Management in Linux** into a **detailed practical guide with commands, examples, and best practices**.

---

# 🔹 SSL/TLS Certificate Management in Linux (Expanded Guide)

---

## 1. **Introduction to SSL/TLS**

* **SSL/TLS** encrypts data between clients and servers and ensures authenticity.
* **TLS handshake**: Client verifies server certificate → key exchange → symmetric encryption established.
* **Certificate components:**

  * Public key → used by clients to encrypt data
  * Private key → kept secret on server
  * Certificate → binds domain/organization to public key, signed by CA

---

## 2. **Types of SSL Certificates**

| Type                         | Purpose                             | Use Case                 |
| ---------------------------- | ----------------------------------- | ------------------------ |
| Domain Validation (DV)       | Verifies domain ownership           | Blogs, personal websites |
| Organization Validation (OV) | Verifies organization info          | Businesses, e-commerce   |
| Extended Validation (EV)     | Strongest validation                | Banks, financial apps    |
| Wildcard                     | Secures subdomains (\*.example.com) | Multi-subdomain apps     |
| Multi-domain (SAN)           | Secures multiple domains            | SaaS platforms           |
| Self-signed                  | For testing/dev                     | Internal servers         |

---

## 3. **Certificate Authorities (CA)**

* **Public CAs**: Let’s Encrypt, DigiCert, GoDaddy, GlobalSign
* **Private CAs**: Used in enterprise for internal services
* **Trust chain**: Root CA → Intermediate CA → Server certificate
* **Check trust chain:**

```bash
openssl verify -CAfile chain.pem server.crt
```

---

## 4. **Key Management**

* **Generate RSA private key:**

```bash
openssl genrsa -out server.key 2048
```

* **Generate ECC private key:**

```bash
openssl ecparam -genkey -name prime256v1 -out server.key
```

* **Protect private key:**

```bash
chmod 600 server.key
chown root:root server.key
```

* Keep keys off public directories and backups

---

## 5. **Creating a Certificate Signing Request (CSR)**

```bash
openssl req -new -key server.key -out server.csr
```

* Fields to fill: CN (Common Name = domain), O (Organization), OU (Org Unit), L (City), ST (State), C (Country)
* Optional: SAN (Subject Alternative Name)

```bash
openssl req -new -key server.key -out server.csr -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:example.com,DNS:www.example.com"))
```

---

## 6. **Installing Certificates**

### Apache

```apache
SSLEngine on
SSLCertificateFile /etc/ssl/certs/server.crt
SSLCertificateKeyFile /etc/ssl/private/server.key
SSLCertificateChainFile /etc/ssl/certs/chain.pem
```

```bash
sudo systemctl restart apache2
```

### Nginx

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/ssl/certs/server.crt;
    ssl_certificate_key /etc/ssl/private/server.key;
    ssl_trusted_certificate /etc/ssl/certs/chain.pem;
}
```

```bash
sudo systemctl restart nginx
```

---

## 7. **Renewing Certificates**

* **Manual renewal:** Generate new CSR → request new cert → replace
* **Automated renewal with Let’s Encrypt (Certbot):**

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d example.com -d www.example.com
```

* Cron job for automatic renewal:

```bash
0 3 * * * /usr/bin/certbot renew --quiet
```

* Reload services after renewal:

```bash
systemctl reload nginx
```

---

## 8. **Certificate Revocation**

* **Using CRL:**

```bash
openssl ca -revoke client.crt -crl_reason keyCompromise
openssl ca -gencrl -out crl.pem
```

* **OCSP (Online Certificate Status Protocol):**

```bash
openssl ocsp -issuer ca.crt -cert server.crt -url http://ocsp.example.com
```

* Revocation ensures compromised certificates are invalidated

---

## 9. **Trust & Chain Management**

* Always include **full chain** in server config (server + intermediate(s) + root optional)
* Test chain with:

```bash
openssl s_client -connect example.com:443 -showcerts
```

* Common errors: `self-signed`, `incomplete chain`, `hostname mismatch`

---

## 10. **Security Best Practices**

* Use strong protocols: TLS 1.2 or TLS 1.3
* Disable weak protocols: SSLv2, SSLv3, TLS 1.0/1.1
* Enable **Perfect Forward Secrecy (PFS)** with ECDHE:

```nginx
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
ssl_prefer_server_ciphers on;
```

* Enable **HSTS (HTTP Strict Transport Security):**

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

---

## 11. **Monitoring & Auditing**

* Check expiration dates:

```bash
openssl x509 -in server.crt -noout -dates
```

* Monitor certificates with scripts or tools like **Nagios, Zabbix, Prometheus**
* Test SSL configuration: [SSL Labs](https://www.ssllabs.com/ssltest/)

---

## 12. **Advanced Topics**

* **SAN certificates:** Multiple domains/subdomains in one cert
* **Wildcard certificates:** Secures all subdomains (`*.example.com`)
* **Automation:** Deploy certificates across multiple servers with **Ansible**, **Chef**, or **Puppet**
* **Docker/Kubernetes integration:** Mount certs as secrets/config maps

---

## 13. **Troubleshooting**

* Common errors:

  * `NET::ERR_CERT_COMMON_NAME_INVALID` → CN mismatch
  * `certificate not trusted` → missing intermediate certs
  * `expired certificate` → check validity dates
* Test server connection:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

* Check supported ciphers:

```bash
nmap --script ssl-enum-ciphers -p 443 example.com
```

---

## 14. **Compliance & Standards**

* PCI DSS requires TLS 1.2+ for sensitive data
* NIST and OWASP TLS guidelines
* Keep track of browser/OS trusted CA stores

---

✅ This guide now covers **key generation, CSR creation, installation, renewal, revocation, monitoring, security best practices, troubleshooting, and advanced deployment**.

I can also create a **visual flow diagram showing SSL/TLS certificate lifecycle (CSR → CA → install → renewal → revocation)** for Linux web servers if you want—it’s very helpful for training and quick reference.

Do you want me to create that diagram?


