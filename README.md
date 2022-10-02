# Serving Cluster Workloads over SSL

**DISCLAIMER** This work is designed to help you serve applications over HTTPS within a _private_ cluster. It is in no way considered production grade. Moreover, we create a self-signed CA certificate which cannot be used for serving to the public Internet.

This is a follow on from my tutorial on [installing MetalLB and Ingress](https://github.com/fireflycons/howto-install-metallb).

In this guide, we will create a Public Key Infrastructure (PKI) using [easy-rsa](https://github.com/OpenVPN/easy-rsa) which is a wrapper for `openssl` written in Bourne Shell. We will create a Root CA which is kept locked away, and from that an Intermediate CA which will be used for issuing certificates to cluster workloads using [cert-manager](https://cert-manager.io/). I am not at this point covering management of Certificate Revocation Lists (CRLs), since you would not be passing any of generated certificates outside of your own direct control.

We will then proceed to distribute the certificates to trust store(s) on the network, then set up [cert-manager] in the cluster to create web-serving certificates for cluster services, then demonstrate serving a workload over HTTPS.

# Lab Steps

1. [Build the PKI](./docs/01-build-pki.md)
1. [Distribute Certificates](./docs/02-certificate-distribution.md)
1. [Install cert-manager](./docs/03-cert-manager.md)
1. [Deploy a secured service](./docs/04-deploy-service.md)


