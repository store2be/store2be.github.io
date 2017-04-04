---
date: 2017-04-04
title: "Kubernetes: Redirect HTTP to HTTPS with ELB and the nginx ingress controller"
categories:
  - AWS
  - Kubernetes
  - ELB
  - SSL
author_staff_member: 03_tom
medium_link: https://medium.com/@tom.houle/kubernetes-redirect-http-to-https-with-elb-and-the-nginx-ingress-controller-911c3f4f605
practical_dev_link: https://dev.to/tomhoule/kubernetes---redirect-http-to-https-with-elb-and-the-nginx-ingress-controller
---

TL;DR Read the last paragraph for the most recent way of implementing HTTPS with an nginx ingress controller while letting ELB handle the certificates.

In our exploratory work with Kubernetes, one of the first hurdles was configuring a proper HTTP ingress. At first, we wanted three properties from our setup:

*All unencrypted http traffic is immediately redirected to https*.
*The ingress controller does not handle TLS certificates*. instead we have TLS termination at the ELB level.
*The redirection is handled by the ingress controller*, not the individual ingress resources, since we always want HTTP traffic to be encrypted, and it limits the potential for errors.

It turns out that combining those three was more complicated than expected, since [ELB does not do HTTP redirects](https://aws.amazon.com/de/premiumsupport/knowledge-center/redirect-http-https-elb/), and neither [træfik](https://docs.traefik.io/toml/#kubernetes-ingress-backend) nor [nginx](https://github.com/kubernetes/ingress/tree/master/controllers/nginx) ingress controllers supported redirecting unless they handled TLS termination themselves.

## Our solution

The DNS (Route 53) and ELB configurations were the simplest.

We first created a LoadBalancer service for our ingress:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: kube-system
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: <the arn for our wildcard TLS cert>
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
spec:
  selector:
    k8s-app: nginx-ingress-lb
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: https
    port: 443
    targetPort: 80
```
When applied, the service will create the ELB load balancer for you.

Our DNS configuration is very simple: a “gateway” domain that points to the load balancer. The other subdomains are just CNAME records that point to it. The “gateway” domain is updated manually as a part of the cluster creation, and we just add the CNAME records when needed. We plan on trying out [external-dns](https://github.com/kubernetes-incubator/external-dns) for the “gateway” domain to automate this in the future.

The last part was less ideal, since the nginx ingress controller from kubernetes (there is [another implementation](https://github.com/nginxinc/kubernetes-ingress) by the nginx people) did not support redirecting to https unless it is configured to handle the https traffic itself (which we really don’t want).

Fortunately, the controller lets you completely override its nginx config file, so we just copied the default file from the source, which is of course not ideal, and added what we wanted, which is a 301 redirect based on the `X-Forwarded-Proto` header set by ELB:

```
if ( $http_x_forwarded_proto = http ) {
  rewrite ^(.*) https://$host$1 permanent;
}
```

We can then just put the modified config template in a ConfigMap (in our case nginx-config in the kube-system namespace) and pass it to nginx by placing this in the `container` section of the Deployment:

```yaml
args:
  - /nginx-ingress-controller
  - --default-backend-service=kube-system/default-http-backend
  - --publish-service=kube-system/nginx-service
  - --configmap=kube-system/nginx-config
```

## How to do it *today*

This story is a testimony to how fast things are moving in the k8s ecosystem. We came up with this solution on the nginx ingress controller version [0.9.0-beta.1](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/Changelog.md#09-beta1), exactly two weeks ago. Now the current version is [0.9.0-beta.3](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/Changelog.md#09-beta3) and the `ingress.kubernetes.io/force-ssl-redirect` annotation was added, making the same configuration achievable by adding it in each ingress resource.

This option ticks 2 of our 3 boxes, and we found that the last point, having this setting centralized, was not important enough to justify the hack.
