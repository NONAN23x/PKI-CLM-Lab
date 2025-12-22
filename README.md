# Public Key Infrastructure Lab Writeup

<div align=center>

![Image](./assets/PKI-CLM-Lab.jpg)
</div>

This repository documents my journey through building a comprehensive Public Key Infrastructure (PKI) home lab. The lab explores various aspects of PKI implementation, from traditional Microsoft Active Directory Certificate Services to open-source solutions like EJBCA, and includes hands-on experience with cryptographic tools and hardware security modules

## Lab Components

### 1. Microsoft Active Directory PKI Environment
- **Infrastructure**: Deployed 2 Domain Controllers and 2 Domain Joined Workstations
- **Certificate Authority Setup**:
  - Root Certificate Authority (CA)
  - Subordinate Certificate Authority
- **Certificate Issuance**: Microsoft IIS Server Client TLS Certificate via AD Certificate Services

### 2. Linux CA
- **OpenSSL Environment**: Ubuntu VM for cryptographic exploration
  - RSA and ECDSA key pair generation
  - Digital certificate and CSR creation
  - CRL and OCSP implementation
- **EJBCA Docker Deployment**: Root and Subordinate CA configuration
  - Client and Server TLS certificate issuance
  - Crypto token creation and management

### 3. Advanced PKI Tools Research
- **HashiCorp Vault**: Secret management and PKI backend exploration
- **Hardware Security Modules (HSMs)**: Study of dedicated cryptographic hardware
- **Cloud HSM Solutions**: Amazon AWS CloudHSM analysis
- **Enterprise HSMs**: Thales Luna HSM investigation

## CA Implementation Tasks

### Microsoft Active Directory Certificate Services
- Deployed infrastructure: 2 Domain Controllers and 2 Domain Joined Workstations
- Established Root Certificate Authority (CA)
- Configured Subordinate Certificate Authority
- Issued Microsoft IIS Server Client TLS Certificate

### Linux Certificate Authorities

#### OpenSSL
- Generated public/private key pairs (RSA, ECDSA)
- Created digital certificates
- Generated Certificate Signing Requests (CSRs)
- Produced Certificate Revocation Lists (CRLs)
- Implemented Online Certificate Status Protocol (OCSP)

#### EJBCA
- Deployed EJBCA via Docker on Ubuntu VM
- Configured Root CA and Subordinate CA hierarchy
- Issued Client and Server TLS certificates
- Created and managed Crypto Tokens

### Advanced PKI Research
- Explored HashiCorp Vault for secret management and PKI backends
- Studied Hardware Security Modules (HSMs)
- Analyzed Amazon AWS CloudHSM solutions
- Investigated Thales Luna enterprise HSMs

## Objectives

This writeup serves as a practical learning platform for:
- Understanding PKI fundamentals and architecture
- Implementing certificate authorities and hierarchies
- Hands-on experience with certificate lifecycle management
- Exploring various PKI tools and technologies
- Comparing traditional vs. modern PKI solutions

## Technologies Covered

- **Microsoft Active Directory Certificate Services**
- **OpenSSL**
- **EJBCA (Enterprise Java Bean Certificate Authority)**
- **Hardware Security Modules (HSMs)**


## Getting Started

### Prerequisites
- Virtualization platform (VMware, VirtualBox, Hyper-V)
- Basic understanding of networking and system administration
- Familiarity with Linux command line

### Lab Setup
1. Deploy Microsoft Active Directory environment
2. Configure Domain Controllers and join workstations
3. Set up Certificate Authorities in AD CS
4. Deploy Ubuntu VM for OpenSSL experimentation
5. Install EJBCA via Docker
6. Explore additional PKI tools as needed

## Key Learnings

- Certificate Authority hierarchy design and implementation
- Public Key Infrastructure best practices
- Cryptographic key generation and management
- Certificate lifecycle (issuance, renewal, revocation)
- Integration of PKI with enterprise environments
- Comparison of commercial vs. open-source PKI solutions

## Future Enhancements

- Integration with cloud-based PKI services
- Automation of certificate deployment processes
- Implementation of certificate transparency
- Exploration of post-quantum cryptography
- Advanced HSM integration scenarios

## Contributing

This is a personal learning project, but feel free to:
- Open issues for questions or suggestions
- Fork and adapt for your own PKI learning journey
- Share your own PKI lab experiences

## License

This project is for educational purposes. Please refer to individual tool licenses for any code or configurations included

## Disclaimer

This lab is intended for educational and research purposes only. Ensure compliance with applicable laws and regulations when implementing PKI in production environments