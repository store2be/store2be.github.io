---
date: 2017-03-30
title: Multi AZ Kubernetes with kops in AWS regions with only 2 AZ
categories:
  - AWS
  - Kubernetes
  - kops
author_staff_member: 01_peter
medium_link: https://medium.com/@peterfication/multi-az-kubernetes-with-kops-in-aws-regions-with-only-2-az-69e2edcc441d
---

At the moment we are setting up Kubernetes at store2be. We are a company from Germany and most of our business is
taking place in Germany. Therefore, we want our AWS servers to be in Frankfurt.

We use [kops](https://github.com/kubernetes/kops) for our Kubernetes cluster management and we want our cluster to be a
multi availability zone (AZ) deployment. With this in mind we tried out the following command:

```
$ kops create cluster \
    --node-count 4 \
    --zones eu-central-1a,eu-central-1b \
    --master-zones eu-central-1a,eu-central-1b \
    --node-size t2.medium \
    --master-size m3.medium \
    --yes \
    kubernetes-cluster.example.org
```

But the result was not that promising:

```
There should be an odd number of master-zones, for etcd's quorum.  Hint: Use --zones and
--master-zones to declare node zones and master zones separately.
```

So unfortunately, at the moment it seems that it is not possible to use kops out of the box in an AWS region with only
2 AZ. But after some digging we found a [Github issue](https://github.com/kubernetes/kops/issues/732) that described
our problem and even had a [work around](https://github.com/kubernetes/kops/issues/732#issuecomment-275888160) for
that! So the following steps are based on the solution provided by [Kamil Hristov](https://github.com/kamilhristov).

---

## 1. Create a cluster.yml for a AWS region with 3 AZ

Create a cluster in eu-west (because it has 3 AZ):

```
$ kops create cluster \
    --node-count 4 \
    --zones eu-west-1a,eu-west-1b,eu-west-1c \
    --master-zones eu-west-1a,eu-west-1b,eu-west-1c \
    --node-size t2.medium \
    --master-size m3.medium \
    --yes \
    kubernetes-cluster.example.org
```

Dump the cluster and instancegroup configuration and append it:

```
$ kops get cluster --name=kubernetes-cluster.example.org --output yaml > cluster.yml && \
  echo "\n---\n" >> cluster.yml && \
  kops get instancegroups --name=kubernetes-cluster.example.org --output yaml >> cluster.yml
````

## 2. Adjust the cluster.yml

* Replace all the occurrences of eu-west with eu-central
* Replace all the zone definitions of eu-central-1c with eu-central-1a
* Replace all the name definitions of eu-central-1c with something different than eu-central-1a like for example
eu-central-1a2 (otherwise there will be name conflicts)

## 3. Create the cluster by using the adjusted cluster.yml

```
# We have to use replace here, because the cluster spec already has
# been created in step 1.
$ kops replace -f cluster.yml
$ kops create secret --name kubernetes-cluster.example.org sshpublickey admin -i ~/.ssh/id_rsa.pub
$ kops update cluster --yes
```

And ✨, there is the multi AZ cluster in Frankfurt 🎉

---

_I created this article because it took us a while to find out about this solution and I hope we could help someone
with it. Thanks to [Kamil Hristov](https://github.com/kamilhristov) for sharing the  solution. Thanks to
[Tom Houlé](https://github.com/tomhoule) for reviewing the article._
