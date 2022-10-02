# Install cert-manager in the cluster

This tutorial is based on the assumption that your private cluster is set up for [load balanced ingress](https://github.com/fireflycons/howto-install-metallb), so that now we can configure ingress to serve trusted sites over https.

To do this, we will now install [cert-manager](https://cert-manager.io/) such that it will issue web serving certificates automatically to any ingress configured to use SSL.

## Prepare Intermediate CA certificate and key

There is a small amount of preparation to do to prepare the keys for use by cert-manager. Kubernetes secrets cannot handle private keys that have a passphrase (which is enforced by easy-rsa). Thus we have to remove it

Log onto the the server where you created the PKI.

1. Copy the intermediate CA keys to tmp
    ```bash
    cd /tmp
    cp /var/lib/pki/intermediate-ca/ca.crt .
    cp /var/lib/pki/intermediate-ca/private/ca.key ca-encrypted.key
1. Remove the passphrase from the key. You will be asked for it when you perform the following command
    ```bash
    openssl rsa -in ca-encrypted.key -out ca.key
    ```
1. Copy the resulting `ca.crt` and `ca.key` onto the machine and into the directory from where you will be running `kubectl`
1. Delete these keys from the tmp directory on your PKI box
    ```bash
    rm /tmp/ca*.*
    ```

## Install cert-manager

1. Add the Helm repo for cert-manager
    ```bash
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    ```
1. Create a namespace for it
    ```bash
    kubectl create namespace cert-manager
    kubectl config set-context --current cert-manager
    ```
1. Install the chart
    ```
    helm install \
    cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v1.9.1 \
    --set installCRDs=true
1. Verify all three services are running
    ```bash
    kubectl get deployments
    ```

    > Output

    ```
    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    cert-manager              1/1     1            1           56s
    cert-manager-cainjector   1/1     1            1           55s
    cert-manager-webhook      1/1     1            1           56s
    ```
1. Create the secret for your CA keypair
    ```bash
    kubectl create secret tls ca-key-pair --cert ca.crt --key ca.key
    ```
1. Create a `ClusterIssuer` resource that will be referenced by ingress resources to get certificates
    1. Create a manifest `cluster-issuer.yaml` in your favourite editor and paste the following YAML

        ```yaml
        apiVersion: cert-manager.io/v1
        kind: ClusterIssuer
        metadata:
          name: ca-issuer
        spec:
          ca:
            secretName: ca-key-pair
        ```

    1. Apply the manifest
        ```bash
        kubectl create -f cluster-issuer.yaml

Next: [Deploy a secured service](./04-deploy-service.md)
