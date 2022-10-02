# Building the PKI

In this section we will build two PKIs, one for the root certificate whose purpose is only to sign intermediate certificates, and one for the intermediate certificate which will later be used as the signing certificate for serving certificates issued within the cluster.

## Create a VM for the PKIs

If this were a production grade CA, then it is best practice to create a machine that is totally isolated from any network, and transfer material to and from it with removable media. However, this is internal only simply create a VM which you'll leave powered down unless you need to work with the PKI.

How you deploy this VM is up to you, but I used [Alpine Linux](https://wiki.alpinelinux.org/wiki/Installation) since it's tiny and I only needed give it 512MB RAM, 1 vCPU and 2GB virtual disk. It could probably manage on less!

## Install EasyRSA

Once you've installed and booted up this VM, download appropriate version for your operating system from the [releases](https://github.com/OpenVPN/easy-rsa/releases) page.

Un-tar (or unzip if Windows) to some directory (I chose `/opt`), and it is ready to go.

## Initialize PKI for the root CA

1. Set environment variables. The first must point to the directory where the `easyrsa` shell script is, and the second to where you want the PKI to be, e.g.

    ```bash
    export EASYRSA=/opt/EasyRSA-3.1.0
    export EASYRSA_PKI=/var/lib/pki/root-ca
    ```

    For a full list of environment variables that can be configured, see [here](https://github.com/OpenVPN/easy-rsa/blob/master/doc/EasyRSA-Advanced.md). Ones you aren't going to change, you can set in `root`'s profile.
1. Initialize the PKI directory structure for Root CA

    ```bash
    cd $EASYRSA
    ./easyrsa init-pki
    ```

1. Generate the Root CA certificate and key

    ```bash
    ./easyrsa build-ca
    ```

    You will be asked for
    * A passphrase to protect the key
    * A common name (CN) for the CA certificate - generally you'll put something like `example.local Root CA`

    The certificate will be placed as `$EASYRSA_PKI/ca.crt` and the key as `$EASYRSA/private/ca.key`

## Initialize PKI for the intermediate CA

1. Initialize the PKI directory structure.</br>We first have to re-set the environment variable that points to where the PKI will be built, since we need a separate location for the intermediate.

    ```bash
    export EASYRSA_PKI=/var/lib/pki/intermediate-ca
    ./easyrsa init-pki
    ```
1. Create a signing request for the new Intermediate CA certificate.</br>We are preparing a signing request, as we will need to go back to the Root CA to get the certificate issued.

    ```bash
    ./easyrsa build-ca subca
    ```

    You will be asked for
    * A passphrase to protect the key
    * A common name (CN) for the CA certificate - generally you'll put something like `example.local Intermediate CA`

    This will generate a `ca.req` file and print the path to it
1. Issue the Intermediate CA certificate.</br>For this we return to the Root CA's PKI, import, then sign the request as a CA. Here `intermediate-ca` is an alias assigned to the request which we can use to refer to it in other operations (such as signing). You can call it what you like. This name will also be the filename of the generated certificate.

    ```bash
    export EASYRSA_PKI=/var/lib/pki/root-ca
    ./easyrsa import-req /var/lib/pki/intermediate-ca/reqs/ca.req intermediate-ca
    ./easyrsa sign-req ca intermediate-ca
    ```

    You will be asked for
    * Confirmation you want to proceed
    * The pass phrase you entered for the Root CA's key.

    The new cerficiate will be created
1. Copy the new intermediate CA certificate to it's own PKI as that PKI's CA

    ```bash
    cp /var/lib/pki/root-ca/issued/intermediate-ca.crt /var/lib/pki/intermediate-ca/ca.crt
    ```

At this point, we now have the following certificates and keys that we will need to copy elsewhere in the distribution step

| Cert/Key | Location |
|----------|----------|
| Root CA Certificate | `/var/lib/pki/root-ca/ca.crt` |
| Intermediate CA Certificate | `/var/lib/pki/intermediate-ca/ca.crt` |
| Intermediate CA Key| `/var/lib/pki/intermediate-ca/private/ca.key` |

Next: [Distribute Certificates](./02-certificate-distribution.md)