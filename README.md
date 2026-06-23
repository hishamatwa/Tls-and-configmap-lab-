# Kubernetes Nginx TLS Lab

## Overview

This task deploys an Nginx application on Kubernetes and configures it to work with HTTPS using a TLS certificate.

Ingress was not used in this task.
Instead, Nginx itself was configured to listen on HTTPS port 443.

The application was exposed outside the cluster using a NodePort Service.

## What Was Created

* Namespace: `nginx-tls`
* TLS Secret: `nginx-tls-secret`
* ConfigMap: `nginx-tls-config`
* Deployment: `nginx-tls-app`
* Service: `nginx-tls-service`
* NodePort: `30443`

## How It Works

Nginx needs two files to work with HTTPS:

```text
tls.crt
tls.key
```

The certificate and private key were generated using OpenSSL.

Then they were stored inside a Kubernetes TLS Secret named:

```text
nginx-tls-secret
```

The Secret was mounted inside the Nginx container at:

```text
/etc/nginx/tls
```

Nginx was configured to read the certificate and key from:

```text
/etc/nginx/tls/tls.crt
/etc/nginx/tls/tls.key
```

## Why ConfigMap Was Used

The default Nginx image normally serves HTTP on port 80.

A ConfigMap was used to provide a custom Nginx configuration that makes Nginx listen on HTTPS port 443.

This avoids building a custom Docker image.

The important part of the Nginx configuration is:

```nginx
listen 443 ssl;

ssl_certificate /etc/nginx/tls/tls.crt;
ssl_certificate_key /etc/nginx/tls/tls.key;
```

## Why Service Was Used

The Deployment only runs the Nginx Pod.

A Service is required to expose the Pod.

In this task, a NodePort Service was used so the application can be accessed from outside the cluster using:

```text
https://NODE-IP:30443
```

Example:

```text
https://192.168.1.9:30443
```

## Port Explanation

`containerPort: 443` means Nginx listens on port 443 inside the container.

`targetPort: 443` means the Service forwards traffic to port 443 inside the Pod.

`port: 443` is the Service port inside the Kubernetes cluster.

`nodePort: 30443` is the external port opened on the Kubernetes node.

## Test Command

The application was tested using:

```bash
curl -k --resolve nginx.local:30443:192.168.1.9 https://nginx.local:30443
```

The command returned the default Nginx welcome page, which proves that HTTPS is working.

## Problems Faced and Fixes

### Problem 1: Browser showed "Your connection is not private"

The browser showed:

```text
NET::ERR_CERT_AUTHORITY_INVALID
```

This happened because the certificate was self-signed.

This is expected in a lab environment because the certificate was created manually using OpenSSL and was not issued by a public Certificate Authority.

The application still worked successfully using curl with the `-k` option.

### Problem 2: Pod was stuck in ImagePullBackOff

The Pod failed to start at first because Kubernetes tried to pull the `nginx:latest` image from Docker Hub, but the VM did not have working internet access.

The fix was to use:

```yaml
imagePullPolicy: IfNotPresent
```

This tells Kubernetes to use the local Nginx image if it already exists on the node.

### Problem 3: NodePort returned connection refused

At first, the NodePort test returned connection refused because the Nginx Pod was not running yet.

After fixing the image pull issue and the Pod became Running, the NodePort Service worked correctly.

### Problem 4: VM internet was not working

The VM had an incorrect default route after network changes.

The route was fixed so the VM could reach the internet again.

After the fix, `ping 8.8.8.8` worked and Docker Hub returned `HTTP/2 401`, which means Docker Hub was reachable.

## Important Security Note

The private key and certificate files should not be pushed to GitHub.

Do not push:

```text
nginx-local.key
nginx-local.crt
```

Only the Kubernetes YAML files and README should be pushed.

## Summary

In this task, Nginx was deployed on Kubernetes and configured to serve HTTPS directly without using Ingress.

A TLS certificate and private key were generated, stored in a Kubernetes TLS Secret, and mounted inside the Nginx container.

A ConfigMap was used to provide the HTTPS Nginx configuration.

A NodePort Service exposed the application externally on port `30443`.

The final HTTPS test returned the Nginx welcome page, proving that the setup worked successfully.

