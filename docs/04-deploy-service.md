# Deploy an SSL secured service to the ingress

In this final lab we will create a deployment using the default nginx inage and verify that it can be browsed over HTTPS without a warning about untrusted certificates. Note that you must run the final test from a machine where you have correctly installed the certificates into the machine's trust store.

For ingress to really work on your internal network, ideally you will have a DNS infrastructure deployed into which you can add hostnames for your services. If you're running Active Directory, you already have it. If you only have one computer that you will use to access the deployed services, then you can add the host names you're going to use to the machine's `hosts` file. For the purpose of this exercise, I'll assume a private domain of `.example.local`, but you can use whatever you like, as long as you follow it through the remaining steps.

## Deploy the Ingress

1. Use the default namespace

    ```bash
    kubectl config set-context --current --namespace default
    ```
1. Create a deployment

    ```bash
    kubectl create deployment nginx-ssl --image nginx --replicas 1
    ```
1. Create a service to expose it

    ```bash
    kubectl expose deployment nginx-ssl --type ClusterIP --port 80
    ```
1. Create the ingress resource.</br>Note here we supply an annotation referring to the cluster issuer we created in the previous lab, which will be picked up by `cert-manager` in order to issue a web-serving certificate. There is also a `tls` section which names a secret that will be created to hold the serving certificate along with the Subject Alternate Names (SANs) that will be written into the certificate under the `hosts` section.</br>Create a manifest file `ingress.yaml` with the following:

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      annotations:
        cert-manager.io/cluster-issuer: ca-issuer
      name: nginx-ssl
      namespace: default
    spec:
      ingressClassName: nginx
      rules:
      - host: test-nginx.example.local
        http:
          paths:
          - backend:
              service:
                name: nginx-ssl
                port:
                  number: 80
            path: /
            pathType: Prefix
      tls:
      - hosts:
        - test-nginx.example.local
        secretName: nginx-ssl
    ```

    And apply it

    ```bash
    kubectl create -f ingress.yaml
    ```

1. Verify the resources

    ```bash
    kubectl get ingress nginx-ssl
    ```

    > Output

    ```
    NAME        CLASS   HOSTS                      ADDRESS           PORTS     AGE
    nginx-ssl   nginx   test-nginx.example.local   192.168.230.128   80, 443   4m41s
    ```

    ```bash
    kubectl get secret nginx-ssl
    ```

    > Output

    ```
    NAME        TYPE                DATA   AGE
    nginx-ssl   kubernetes.io/tls   3      5m33s
    ```

## Add the new host

We have now deployed a service which should be accessed via `https://test-nginx.example.local/`. Assuming you configured the ingress controller as per [installing MetalLB and Ingress](https://github.com/fireflycons/howto-install-metallb), then the ingress controller will be deployed in the `ingress-nginx` namespace and its external IP address can be seen by examining the controller's service

```bash
kubectl get service -n ingress-nginx  ingress-nginx-controller
```

If you do have DNS and you [set up a record for the ingress controller](https://github.com/fireflycons/howto-install-metallb/blob/master/docs/02-install-ingress.md#setting-up-dns), then we can create a `CNAME` record in DNS for our new service:

```
test-nginx CNAME mycluster-ingress.example.local
```

If you do not have a DNS, then add the record to the hosts file at all machines that will be connecting, using the IP address of the ingress controller service.

## Verify the deployment

From a browser on a machine that has the CA certificates, and knows how to find the host (either via DNS or via `hosts` file), you should now be able to go to `https://test-nginx.example.local` and see the default nginx page.

From a properly configured Linux host, you should be able to curl the site without the `-k` switch which disables certificate validation

```bash
curl https://test-ingress.example.local/
```

> Output

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```