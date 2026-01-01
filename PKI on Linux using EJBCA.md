# Linux Certificate Authorities

## Overview

This section documents comprehensive hands-on experience with OpenSSL and EJBCA, exploring cryptographic fundamentals and enterprise-grade certificate authority solutions on Linux.

---

## Part 1: OpenSSL - Cryptographic Foundation

### Objectives
- Generate public/private key pairs (RSA and ECDSA)
- Create digital certificates and Certificate Signing Requests (CSRs)
- Implement Certificate Revocation Lists (CRLs)
- Deploy Online Certificate Status Protocol (OCSP)
- Understand PKI fundamentals from first principles

### Environment Setup

```bash
# Ubuntu VM Requirements
sudo apt update && sudo apt upgrade -y
sudo apt install openssl openssl-dev -y

# Verify installation
openssl version
```

### Key Generation

#### RSA Key Pairs

```bash
# Generate 2048-bit RSA private key
openssl genrsa -out private_key.pem 2048

# Generate 4096-bit RSA private key (stronger)
openssl genrsa -out private_key_4096.pem 4096

# Extract public key from private key
openssl rsa -in private_key.pem -pubout -out public_key.pem

# View private key details
openssl rsa -in private_key.pem -text -noout

# View public key details
openssl pkey -in public_key.pem -text -noout
```

#### ECDSA Key Pairs

```bash
# List available elliptic curves
openssl ecparam -list_curves

# Generate ECDSA key pair (P-256)
openssl ecparam -name prime256v1 -genkey -noout -out ec_private.pem

# Generate ECDSA key pair (P-384)
openssl ecparam -name secp384r1 -genkey -noout -out ec_private_384.pem

# Extract public key from ECDSA private key
openssl ec -in ec_private.pem -pubout -out ec_public.pem

# View ECDSA key details
openssl ec -in ec_private.pem -text -noout
```

### Certificate Creation

#### Self-Signed Root Certificate

```bash
# Generate self-signed certificate valid for 365 days
openssl req -new -x509 -key private_key.pem -out rootca.crt -days 365 \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=Root CA"

# Verify certificate
openssl x509 -in rootca.crt -text -noout

# View certificate validity period
openssl x509 -in rootca.crt -noout -dates
```

#### Certificate Signing Request (CSR)

```bash
# Create CSR without interactive prompts
openssl req -new -key private_key.pem -out server.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=www.example.com"

# Create CSR with Subject Alternative Names (SAN)
openssl req -new -key private_key.pem -out server_san.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=www.example.com" \
  -addext "subjectAltName=DNS:www.example.com,DNS:example.com,IP:192.168.1.100"

# Verify CSR
openssl req -in server.csr -text -noout
```

#### Signing a CSR with Root CA

```bash
# Create a config file for extensions (extensions.cnf)
cat > extensions.cnf << EOF
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
extendedKeyUsage = serverAuth
keyUsage = digitalSignature, keyEncipherment
subjectAltName = DNS:www.example.com,DNS:example.com
EOF

# Sign CSR with root CA
openssl x509 -req -in server.csr -CA rootca.crt -CAkey private_key.pem \
  -CAcreateserial -out server.crt -days 365 -sha256 \
  -extfile extensions.cnf

# Verify signed certificate
openssl x509 -in server.crt -text -noout
```

### Certificate Chain and Verification

```bash
# Create certificate chain (Root + Intermediate + Server)
cat server.crt rootca.crt > cert_chain.pem

# Verify certificate chain
openssl verify -CAfile rootca.crt server.crt

# Verify with chain
openssl verify -CAfile cert_chain.pem server.crt
```

### Certificate Revocation Lists (CRL)

```bash
# Create CRL configuration file (crl.cnf)
cat > crl.cnf << EOF
[ca]
default_ca = CA_default

[CA_default]
database = index.txt
crldir = crldir
new_certs_dir = certs
serial = serial.txt
RANDFILE = private/.rand
default_crl_days = 30
default_md = sha256
policy = policy_anything

[policy_anything]
countryName = optional
stateOrProvinceName = optional
organizationName = optional
organizationalUnitName = optional
commonName = supplied
emailAddress = optional
EOF

# Initialize CRL infrastructure
mkdir -p certs crldir
touch index.txt
echo "01" > serial.txt

# Generate CRL
openssl ca -config crl.cnf -gencrl -out crl.pem

# View CRL
openssl crl -in crl.pem -text -noout

# Add certificate to revocation list
openssl ca -config crl.cnf -revoke server.crt

# Regenerate CRL with revoked certificate
openssl ca -config crl.cnf -gencrl -out crl_updated.pem

# Verify revoked certificate in CRL
openssl crl -in crl_updated.pem -text -noout | grep -A 5 "Revoked Certificates"
```

### Online Certificate Status Protocol (OCSP)

```bash
# Generate OCSP responder certificate
openssl req -new -key ocsp_private.pem -out ocsp.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=OCSP Responder"

# Sign OCSP responder certificate (must have OCSP signing extension)
openssl x509 -req -in ocsp.csr -CA rootca.crt -CAkey private_key.pem \
  -CAcreateserial -out ocsp.crt -days 365 -sha256 \
  -extfile <(echo "extendedKeyUsage = OCSPSigning")

# Start OCSP responder
openssl ocsp -port 8888 -text -sha256 \
  -index index.txt \
  -CA rootca.crt \
  -rkey ocsp_private.pem \
  -rsigner ocsp.crt \
  -ndays 1

# Query OCSP responder (from another terminal)
openssl ocsp -issuer rootca.crt -cert server.crt \
  -url http://localhost:8888 -header "HOST" "localhost:8888"
```

### Practical Exercises

#### Exercise 1: Create Multi-Tier CA Hierarchy

```bash
# 1. Create Root CA
openssl genrsa -out root_key.pem 4096
openssl req -new -x509 -key root_key.pem -out root_ca.crt -days 3650 \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=Root CA"

# 2. Create Intermediate CA
openssl genrsa -out intermediate_key.pem 4096
openssl req -new -key intermediate_key.pem -out intermediate.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=Intermediate CA"

# 3. Sign Intermediate with Root
openssl x509 -req -in intermediate.csr -CA root_ca.crt -CAkey root_key.pem \
  -CAcreateserial -out intermediate_ca.crt -days 1825 -sha256 \
  -extfile <(echo "basicConstraints = CA:TRUE")

# 4. Create Server Certificate signed by Intermediate
openssl genrsa -out server_key.pem 2048
openssl req -new -key server_key.pem -out server.csr \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=server.example.com"

openssl x509 -req -in server.csr -CA intermediate_ca.crt -CAkey intermediate_key.pem \
  -CAcreateserial -out server.crt -days 365 -sha256 \
  -extfile <(echo "extendedKeyUsage = serverAuth")

# 5. Build chain file
cat server.crt intermediate_ca.crt root_ca.crt > chain.pem

# 6. Verify entire chain
openssl verify -CAfile root_ca.crt -untrusted intermediate_ca.crt server.crt
```

#### Exercise 2: TLS Server Configuration

```bash
# Create private key and certificate for HTTPS server
openssl req -x509 -newkey rsa:2048 -keyout tls_key.pem -out tls_cert.pem \
  -days 365 -nodes \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=localhost"

# Create combined PEM file (for nginx/apache)
cat tls_cert.pem tls_key.pem > tls_combined.pem

# Test with simple Python HTTPS server
python3 -m http.server 443 --certfile tls_combined.pem

# Test from another terminal
curl --cacert tls_cert.pem https://localhost
```

---

## Part 2: EJBCA - Enterprise Certificate Authority

### Overview

EJBCA (Enterprise Java Beans Certificate Authority) is a comprehensive, scalable PKI solution deployed via Docker for enterprise-grade certificate management.

### Installation and Setup

#### Prerequisites

```bash
# Install Docker and Docker Compose
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose -y

# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Pull EJBCA Community Edition image
docker pull keyfactor/ejbca-ce
```

#### Quick Start: Ephemeral Test Instance

For rapid testing and experimentation:

```bash
# Start ephemeral EJBCA instance (anyone can manage via network)
docker run -it --rm -p 80:8080 -p 443:8443 -h localhost \
  -e TLS_SETUP_ENABLED="simple" \
  keyfactor/ejbca-ce
```

**Access Points**:
- Admin GUI: `https://localhost:8443/ejbca/adminweb/`
- RA Portal: `https://localhost:8443/ejbca/ra/`

### Production-Like Environment Setup

#### Docker Compose Configuration

```bash
# Create directory structure
mkdir -p container/datadbdir
cd container
```

Create `docker-compose.yml`:

```yaml
version: '3.8'

networks:
  access-bridge:
    driver: bridge
  application-bridge:
    driver: bridge

services:
  ejbca-database:
    container_name: ejbca-database
    image: "library/mariadb:latest"
    networks:
      - application-bridge
    environment:
      - MYSQL_ROOT_PASSWORD=foo123
      - MYSQL_DATABASE=ejbca
      - MYSQL_USER=ejbca
      - MYSQL_PASSWORD=ejbca
    volumes:
      - ./datadbdir:/var/lib/mysql:rw
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  ejbca-node1:
    hostname: ejbca-node1
    container_name: ejbca
    image: keyfactor/ejbca-ce:latest
    depends_on:
      ejbca-database:
        condition: service_healthy
    networks:
      - access-bridge
      - application-bridge
    environment:
      - DATABASE_JDBC_URL=jdbc:mariadb://ejbca-database:3306/ejbca?characterEncoding=UTF-8
      - LOG_LEVEL_APP=INFO
      - LOG_LEVEL_SERVER=INFO
      - TLS_SETUP_ENABLED=simple
    ports:
      - "80:8080"
      - "443:8443"
    volumes:
      - ./logs:/opt/jboss/wildfly/standalone/log:rw
```

#### Start Production Instance

```bash
# Start containers
docker-compose up -d

# Wait for EJBCA to initialize (2-3 minutes)
docker logs -f ejbca

# Access EJBCA
# Admin: https://localhost:8443/ejbca/adminweb/
# RA Portal: https://localhost:8443/ejbca/ra/
```

---

## Certificate Authority Configuration in EJBCA

### Step 1: Create Certificate Profiles

#### Root CA Certificate Profile

Navigate to **CA Functions → Certificate Profiles**:

1. Click **ROOTCA** → **Clone**
2. Give it a name: `CORP-ROOT-CA`
3. Edit the cloned profile:
   - **Validity**: 10 years (3650 days)
   - **Key Algorithm**: RSA 4096-bit
   - **Signature Algorithm**: SHA256withRSA
   - **Basic Constraints**: CA:TRUE, pathLenConstraint:0
   - **Key Usage**: Certificate Sign, CRL Sign
   - Save

#### Subordinate CA Certificate Profile

1. Clone **ROOTCA** again
2. Name: `CORP-SUB-CA`
3. Configure:
   - **Validity**: 5 years
   - **Basic Constraints**: CA:TRUE, pathLenConstraint:0
   - **Key Usage**: Certificate Sign, CRL Sign
   - **Authority Info Access**: Include CA Issuer URL
   - Save

#### Server TLS Certificate Profile

1. Clone **SERVER** template
2. Name: `TLS-Server`
3. Configure:
   - **Validity**: 2 years
   - **Key Algorithm**: ECDSA P-256
   - **Extended Key Usage**: Server Authentication
   - **Key Usage**: Digital Signature, Key Encipherment
   - **Subject Alternative Name**: Enable, Allow multiple SANs
   - **Authority Info Access**: Use CA Defined OCSP Locator
   - Save

#### Client TLS Certificate Profile

1. Clone **ENDUSER** template
2. Name: `TLS-Client`
3. Configure:
   - **Validity**: 1 year
   - **Key Algorithm**: ECDSA P-256
   - **Extended Key Usage**: Client Authentication
   - **Key Usage**: Digital Signature, Key Agreement
   - **Allow DN Override**: Enable
   - Save

### Step 2: Create Crypto Tokens

Navigate to **CA Functions → Crypto Token**:

#### Root CA Crypto Token

1. Click **Create new Crypto Token**
2. **Token name**: `CORP-ROOT-TOKEN`
3. **Token type**: Soft Token
4. Click **Create**
5. Generate key pairs:
   - **Sign Key** (Alias: `signKey`)
     - Key Algorithm: RSA 4096
     - Click **Generate**
   - **Encrypt Key** (Alias: `encKey`)
     - Key Algorithm: RSA 4096
     - Click **Generate**
   - **Test Key** (Alias: `testKey`)
     - Key Algorithm: RSA 2048
     - Click **Generate**

#### Subordinate CA Crypto Token

1. Click **Create new Crypto Token**
2. **Token name**: `CORP-SUB-TOKEN`
3. **Token type**: Soft Token
4. Click **Create**
5. Generate same key pairs as Root CA

### Step 3: Create Certificate Authorities

Navigate to **CA Functions → Certificate Authority**:

#### Create Root CA

1. Click **Create CA**
2. **CA name**: `CORP-ROOT-CA`
3. **CA Type**: Root CA
4. Click **Create**
5. Configure the created CA:
   - **Crypto Token**: Select `CORP-ROOT-TOKEN`
   - **Certificate Profile**: Select `CORP-ROOT-CA`
   - **Sign Key**: `signKey`
   - **Encrypt Key**: `encKey`
   - **Test Key**: `testKey`
   - **Validity**: 10 years
   - Configure CA Services (OCSP, CRL distribution points)
   - Click **Save**

#### Create Subordinate CA

1. Click **Create CA**
2. **CA name**: `CORP-SUB-CA`
3. **CA Type**: Subordinate CA
4. Click **Create**
5. Configure:
   - **Crypto Token**: Select `CORP-SUB-TOKEN`
   - **Certificate Profile**: Select `CORP-SUB-CA`
   - **Sign Key**: `signKey`
   - **Encrypt Key**: `encKey`
   - Leave for signing by Root CA
   - Click **Save**

6. **Sign Subordinate CA**:
   - Navigate back to CA Functions
   - Right-click `CORP-SUB-CA` → **Edit**
   - Under CA Services, find the pending CSR
   - Request signature from Root CA
   - Download signed certificate and import

### Step 4: Create End Entity Profiles

Navigate to **RA Functions → End Entity Profiles**:

#### Server End Entity Profile

1. Click **Add End Entity Profile**
2. **Profile name**: `TLS-Server-EE`
3. Click **Edit**
4. **Main Certificate Data**:
   - **DN Attributes**: Enable CN, O, C
   - **Default Certificate Profile**: `TLS-Server`
   - **Available Certificate Profiles**: `TLS-Server`
   - **Default CA**: `CORP-SUB-CA`
   - **Available CAs**: `CORP-SUB-CA`
5. **Other Data**:
   - **Default Token**: User generated
   - **Available Tokens**: PKCS#12, PEM, JKS
6. Save

#### Client End Entity Profile

1. Click **Add End Entity Profile**
2. **Profile name**: `TLS-Client-EE`
3. Click **Edit**
4. **Main Certificate Data**:
   - **DN Attributes**: Enable CN, O, C, Email
   - **Default Certificate Profile**: `TLS-Client`
   - **Available Certificate Profiles**: `TLS-Client`
   - **Default CA**: `CORP-SUB-CA`
5. **Other Data**:
   - **Default Token**: User generated
   - **Available Tokens**: PKCS#12, PEM
6. Save

---

## Issuing Certificates

### Issue Client Certificate

#### Method 1: Using Admin GUI

1. Go to **RA Functions → Add End Entity**
2. Enter:
   - **Username**: `jsmith`
   - **Password**: (set strong password)
   - **Common Name**: `John Smith`
   - **Email**: `jsmith@example.com`
   - **End Entity Profile**: `TLS-Client-EE`
   - **CA**: `CORP-SUB-CA`
3. Click **Add**

#### Method 2: User Enrollment (RA Portal)

1. Access **RA Portal**: `https://localhost:8443/ejbca/ra/`
2. Click **Enroll** → **New Request**
3. **Request Type**: `TLS-Client`
4. **CA**: Select `CORP-SUB-CA`
5. Fill in enrollment data:
   - Common Name: `John Smith`
   - Organization: `CORP`
   - Country: `US`
   - Email: `jsmith@example.com`
6. **Key Pair Generation**: Select "Let me generate key pair"
7. **Key Algorithm**: ECDSA P-256
8. Click **Enroll**
9. Download **PKCS#12** keystore (`.p12` file)

#### Certificate Download and Installation

1. **Download CA Certificate**:
   - Go to **RA Web** → **CA Certificates and CRLs**
   - Download `CORP-ROOT-CA` and `CORP-SUB-CA` as PEM
   - Create chain: `cat CORP-SUB-CA.pem CORP-ROOT-CA.pem > ca-chain.pem`

2. **Install in Browser** (Firefox/Chrome):
   - Settings → Privacy → Certificates
   - Import the downloaded `.p12` file
   - Set private key password when prompted
   - Also import CA chain certificates as "Trusted Certificate Authorities"

### Issue Server Certificate

1. Navigate to **RA Web → Enroll**
2. **Request Type**: `TLS-Server`
3. **CA**: `CORP-SUB-CA`
4. Fill in server details:
   - **Common Name**: `www.example.com`
   - **Subject Alt Name**: `www.example.com`, `example.com`
   - **Organization**: `CORP`
5. **Key Pair Generation**: User generated
6. **Key Algorithm**: ECDSA P-256
7. Click **Enroll**
8. Download **PEM Full Chain** (certificate + chain)

#### Using Server Certificate with nginx

```bash
# Extract private key and certificate
# (If downloaded as PKCS#12)
openssl pkcs12 -in server.p12 -out server_key.pem -nocerts -nodes
openssl pkcs12 -in server.p12 -out server_cert.pem -nokeys

# nginx configuration
server {
    listen 443 ssl http2;
    server_name www.example.com;
    
    ssl_certificate /etc/nginx/certs/server_cert.pem;
    ssl_certificate_key /etc/nginx/certs/server_key.pem;
    ssl_trusted_certificate /etc/nginx/certs/ca-chain.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
}
```

---

## Advanced EJBCA Operations

### Configure OCSP Responder

1. Go to **System Functions → OCSP Configuration**
2. **OCSP Signing Certificate Profile**: Create/select appropriate profile
3. **Key Binding**:
   - Assign which CA should provide OCSP responses
   - Configure responder certificate details
4. Enable **Unsigned OCSP responses** for testing (disable in production)
5. Save and test:
   ```bash
   openssl ocsp -issuer ca.pem -cert client.pem \
     -url http://localhost:8080/ejbca/publicweb/status/ocsp \
     -header "HOST" "localhost"
   ```

### Configure CRL Distribution

1. Go to **CA Functions → CA → Select your CA**
2. **CRL Distribution Points**:
   - Add endpoint: `http://localhost:8080/ejbca/publicweb/crls/latest.crl`
   - Format: X.509 CRL
3. **CRL Update Frequency**: Every 24 hours
4. Generate and download CRL:
   ```bash
   curl http://localhost:8080/ejbca/publicweb/crls/latest.crl -o latest.crl
   openssl crl -in latest.crl -text -noout
   ```

### Export CA Certificate Chain

1. Go to **System Functions → Export System Configuration**
2. Download complete CA hierarchy
3. Distribute Root CA certificate to all systems

### Backup and Recovery

```bash
# Backup EJBCA database
docker exec ejbca-database mysqldump -u ejbca -pejbca ejbca > ejbca_backup.sql

# Backup entire container
docker commit ejbca ejbca-backup:latest

# Restore from backup
docker cp ejbca_backup.sql ejbca-database:/
docker exec ejbca-database mysql -u ejbca -pejbca ejbca < /ejbca_backup.sql
```

---

## Practical Experiments and Workflows

### Experiment 1: Multi-Tier CA Hierarchy

**Objective**: Create a realistic 3-tier PKI hierarchy

1. Create Root CA using `CORP-ROOT-CA` profile
2. Create Subordinate CA using `CORP-SUB-CA` profile  
3. Issue server certificates from Subordinate CA
4. Verify chain: `openssl verify -CAfile ca-chain.pem server.crt`
5. Test OCSP response from subordinate CA

### Experiment 2: Certificate Lifecycle Management

**Objective**: Understand certificate renewal and revocation

1. Issue a certificate with 30-day validity
2. Request certificate renewal before expiration
3. Revoke the old certificate
4. Verify revocation in CRL
5. Query OCSP responder to confirm revocation status

### Experiment 3: Key Rotation

**Objective**: Rotate crypto token keys while maintaining service

1. Create new crypto token with new keys
2. Create new CA using new token (same name, different keys)
3. Re-sign certificates with new CA
4. Update OCSP responder to use new CA
5. Retire old token

### Experiment 4: Certificate Enrollment Workflows

**Objective**: Test multiple enrollment methods

1. **Manual enrollment**: Admin creates user, user enrolls in RA portal
2. **Self-enrollment**: User self-registers in RA portal
3. **Batch enrollment**: Use REST API for bulk certificate requests
4. **SCEP enrollment**: Test Simple Certificate Enrollment Protocol
5. **EST enrollment**: Test Enrollment over Secure Transport

---

## Troubleshooting and Useful Commands

### EJBCA Logs

```bash
# View real-time logs
docker logs -f ejbca

# Export logs
docker exec ejbca cat /opt/jboss/wildfly/standalone/log/server.log > ejbca.log
```

### Database Queries

```bash
# Connect to EJBCA database
docker exec -it ejbca-database mysql -u ejbca -pejbca

# Check issued certificates
SELECT username, serialnumber, status FROM certificatedata LIMIT 10;

# Check CRL generation status
SELECT crlpartition, lastupdate FROM crldata WHERE crlpartition IS NOT NULL;
```

### Certificate Verification

```bash
# Verify server certificate with chain
openssl verify -CAfile ca-chain.pem -untrusted intermediate.pem server.crt

# Check certificate validity period
openssl x509 -in server.crt -noout -dates

# Extract certificate information
openssl x509 -in server.crt -noout -subject -issuer -dates

# Verify PKCS#12 keystore
openssl pkcs12 -in client.p12 -noout -info -passin pass:password
```

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Database connection failed | Ensure MariaDB is healthy: `docker ps` |
| EJBCA won't start | Check logs: `docker logs ejbca` |
| Certificate enrollment fails | Verify End Entity Profile settings and CA is online |
| OCSP responder not responding | Ensure OCSP signing cert exists and CA allows it |
| CRL download returns 404 | Verify CRL distribution points in CA settings |

---

## References

- [Keyfactor EJBCA Community Edition](https://www.ejbca.org/)
- [EJBCA Docker Repository](https://hub.docker.com/r/keyfactor/ejbca-ce)
- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [RFC 5280 - X.509 Certificates](https://tools.ietf.org/html/rfc5280)
- [OCSP Specification](https://tools.ietf.org/html/rfc6960)