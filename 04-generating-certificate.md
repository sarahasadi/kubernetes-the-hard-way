# Provisioning a CA and Generating TLS Certificates
In this lab you will provision a PKI Infrastructure using CloudFlare's PKI toolkit, ```cfssl```, then use it to bootstrap a Certificate Authority, and generate TLS certificates for the following components: 
> -  **kube-apiserver**: The API server that exposes the Kubernetes API and acts as the front-end for the cluster
> -  **kube-controller-manager**: The component that runs the controller processes that manage the cluster state
> -  **kube-scheduler**: The component that assigns pods to nodes based on resource availability and scheduling policies
> -  **kubelet**: The agent that runs on each node and communicates with the API server
> -  **kube-proxy**: The network proxy that maintains network rules and enables service discovery for pods

Create a directory for the PKI files on the jumpbox. Run the following command:
## Certificate Authority.
In this section you will provision a Certificate Authority that can be used to generate additional TLS certificates for the other Kubernetes components.

Generate the CA configuration file, certificate, and private key:

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IR",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Tehran"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```
Results:
```
ca.crt ca.key
```
## Client and Server Certificates
In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes admin user.
### The Admin Client Certificate
Generate the admin client certificate and private key:
```
{

cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IR",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Tehran"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin

}
```
Results:
```
admin-key.pem
admin.pem
```
