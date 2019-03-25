# Reference

https://github.com/kubernetes/kubernetes/issues/54665#issuecomment-340960398


# Etcd Cluster

https://coreos.com/etcd/docs/latest/op-guide/security.html

## Client-to-server communication

--cert-file=<path>: Certificate used for SSL/TLS connections to etcd. When this option is set, advertise-client-urls can use the HTTPS schema.

--key-file=<path>: Key for the certificate. Must be unencrypted.

--client-cert-auth: When this is set etcd will check all incoming HTTPS requests for a client certificate signed by the trusted CA, requests that don't supply a valid client certificate will fail. If authentication is enabled, the certificate provides credentials for the user name given by the Common Name field.

--trusted-ca-file=<path>: Trusted certificate authority.

--listen-client-urls: list of URLs to listen on for client traffic. This flag tells the etcd to accept incoming requests from the clients on the specified scheme://IP:port combinations

## Peer (server-to-server / cluster) communication

The peer options work the same way as the client-to-server options:

--peer-cert-file=<path>: Certificate used for SSL/TLS connections between peers. This will be used both for listening on the peer address as well as sending requests to other peers.

--peer-key-file=<path>: Key for the certificate. Must be unencrypted.

--peer-client-cert-auth: When set, etcd will check all incoming peer requests from the cluster for valid client certificates signed by the supplied CA.

--peer-trusted-ca-file=<path>: Trusted certificate authority.


## Example

```
name: __HOSTNAME__
client-transport-security:
  cert-file: /etc/etcd/etcd.pem
  key-file: /etc/etcd/etcd-key.pem
  trusted-ca-file: /etc/etcd/ca.pem
  client-cert-auth: true
peer-transport-security:
  cert-file: /etc/etcd/etcd.pem
  key-file: /etc/etcd/etcd-key.pem
  trusted-ca-file: /etc/etcd/ca.pem
  client-cert-auth: true
initial-advertise-peer-urls: https://__IP__:2380
listen-peer-urls: https://__IP__:2380
listen-client-urls: https://__IP__:2379,https://127.0.0.1:2379
advertise-client-urls: https://__IP__:2379
initial-cluster-token: etcd-cluster-0
initial-cluster: controller-0=https://__IP__:2380,controller-1=https://__IP1__:2380,controller-2=https://__IP2__:2380
initial-cluster-state: new
data-dir: /var/lib/etcd
```


# Apiserver

## API server’s TLS certificate

With this option, we secure the communications with the kubernetes Api.

In your kubeconfig when you're add a new cluster, the associated CA must be the CA used to sign the tls-cert-file

```
kubectl config set-cluster training-cluster-0 --server="https://endpoint" --certificate-authority="ca.crt" --embed-certs=true --kubeconfig="test-kubecfg"
```

### tls-cert-file

File containing the default x509 Certificate for HTTPS
We will name the CA that issues the API server’s TLS cert : **Kubertes Api Ca**

### tls-private-key-file

File containing the default x509 private key matching --tls-cert-file

## Etcd

apiserver -> etcd

### etcd-cafile

Trusted certificate authority  (in the etcd configuration client-transport-security:trusted-ca-file)

### etcd-certfile

A certificate signed by the etcd CA (in the etcd configuration client-cert-auth is set)

### etcd-keyfile

The pricate key associated to the etcd-certfile

### etcd-servers

List of the etcd servers (for example https://10.240.0.10:2379,https://10.240.0.11:2379,https://10.240.0.12:2379)

## Kubelet certificate authorities

apiserver -> kubelet

Used by the api server for connection to a kubelet (kubectl exec, kubectl logs)

### kubelet-certificate-authority string

Path to a cert file for the certificate authority (same CA used for signing the tls-cert-file of the kubelet)

### kubelet-client-key string

Path to a client key file for TLS.

### kubelet-client-certificate string

Path to a client cert file for TLS (must be signeed by the client-ca-file of the kubelet)

## Aggregator

apiserver -> aggregated servers

Configuring the aggregation layer allows the Kubernetes apiserver to be extended with additional APIs, which are not part of the core Kubernetes APIs.

Unlike Custom Resource Definitions (CRDs), the Aggregation API involves another server - your Extension apiserver - in addition to the standard Kubernetes apiserver.

The Kubernetes apiserver will need to communicate with your extension apiserver, and your extension apiserver will need to communicate with the Kubernetes apiserver.

In order for this communication to be secured, the Kubernetes apiserver uses x509 certificates to authenticate itself to the extension apiserver.

The Aggregator certificate is signed using the CA file specified in requestheader-client-ca-file

The following parameters are used for client authentification in the aggrated api server

https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/

### proxy-client-cert-file

### proxy-client-key-file

### requestheader-client-ca-file


## API server client certificate authority

One way for Kubernetes components to authenticate to the API server is with client certificates.
All of these client certs should be issued by the same CA (which, again, doesn’t need to be the same as the CA that issued the API server’s server TLS cert).

### client-ca-file

If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated with an identity.

## Serviceaccount private keys (which aren’t signed by a certificate authority)

### service-account-key-file

File containing PEM-encoded x509 RSA or ECDSA private or public keys, used to verify ServiceAccount tokens

The controller manager signs serviceaccount tokens with a private key.

Unlike every other private key that Kubernetes supports, the serviceaccount key doesn’t support “hey use a CA to check if this is the right key”.

This means you have to give exactly the same private key file to every controller manager.

## The request header certificate authority (or: using an authenticating proxy)

### requestheader-client-ca-file

Root certificate bundle to use to verify client certificates on incoming requests before trusting usernames in headers specified by --requestheader-username-headers


# Kubelet

## kubelet arguments

Define the certificates used by the kubelet https Api and the CA used to authenticate client request 

### client-ca-file string
If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated

### tls-cert-file string
File containing x509 Certificate used for serving HTTPS 

### tls-private-key-file string
File containing x509 private key matching --tls-cert-file.

# Controller manager

## service-account-private-key-file

Filename containing a PEM-encoded private RSA or ECDSA key used to sign service account tokens.

