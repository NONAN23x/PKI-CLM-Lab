# PKI Home Lab Writeup

## Overview

This repository documents my journey through building a comprehensive Public Key Infrastructure (PKI) home lab. The lab explores various aspects of PKI implementation, from traditional Microsoft Active Directory Certificate Services to open-source solutions like EJBCA, and includes hands-on experience with cryptographic tools and hardware security modules.

## Lab Components

### 1. Microsoft Active Directory PKI Environment
- **Infrastructure**: Deployed 2 Domain Controllers and 2 Domain Joined Workstations
- **Certificate Authority Setup**:
  - Root Certificate Authority (CA)
  - Subordinate Certificate Authority
- **Certificate Issuance**: Microsoft IIS Server Client TLS Certificate via AD Certificate Services

### 2. OpenSSL Study Environment
- **Platform**: Ubuntu VM dedicated to cryptographic tool exploration
- **Key Management**:
  - Generated public/private key pairs (RSA, ECDSA)
  - Created digital certificates
  - Generated Certificate Signing Requests (CSRs)
- **PKI Components**: Certificate Revocation Lists (CRLs), Online Certificate Status Protocol (OCSP)

### 3. EJBCA Docker Deployment
- **Installation**: EJBCA deployed via Docker on Ubuntu VM
- **CA Hierarchy**: Root CA and Subordinate CA configuration
- **Certificate Services**: Client and Server TLS certificate issuance
- **Security Features**: Crypto token creation and management

### 4. Advanced PKI Tools Research
- **HashiCorp Vault**: Secret management and PKI backend exploration
- **Hardware Security Modules (HSMs)**: Study of dedicated cryptographic hardware
- **Cloud HSM Solutions**: Amazon AWS CloudHSM analysis
- **Enterprise HSMs**: Thales Luna HSM investigation

## Objectives

This lab serves as a practical learning platform for:
- Understanding PKI fundamentals and architecture
- Implementing certificate authorities and hierarchies
- Hands-on experience with certificate lifecycle management
- Exploring various PKI tools and technologies
- Comparing traditional vs. modern PKI solutions

## Technologies Covered

- **Microsoft Active Directory Certificate Services**
- **OpenSSL**
- **EJBCA (Enterprise Java Bean Certificate Authority)**
- **Docker containerization**
- **HashiCorp Vault**
- **Hardware Security Modules (HSMs)**
- **Cloud-based security services**

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

This project is for educational purposes. Please refer to individual tool licenses for any code or configurations included.

## Disclaimer

This lab is intended for educational and research purposes only. Ensure compliance with applicable laws and regulations when implementing PKI in production environments.