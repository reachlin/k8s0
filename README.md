# k8s0: Kubernetes Ground Zero [![Build Status](https://travis-ci.org/reachlin/k8s0.svg)][travis]

It's a minimal kubernetes installation on a fixed configuration.

But *WHY* ? Many people told me there's alreday [MiniKube](https://kubernetes.io/docs/getting-started-guides/minikube/). Unlike MiniKube, which is a layer of golang code wrapped around the real hyperkube, k8s0 uses the exact hyperkube image. It's more like a minimal version of [Kargo](https://github.com/kubernetes-incubator/kargo), but with the capability to run on Travis.

So the user can try this on local host or VM, or use it on Travis for kubernetes DevOps. Developers can verify their service deployment before going to the real production env. And the ansible script make it easy to tweak the installation steps for more usages. On the other hand, if you want to change anything in MiniKube, you have to write golang code and compile it.

Addtionally, people use MiniKube mainly for education purposes, but the one giant piece golang code of MiniKube makes it difficult to understand how k8s is installed and how each component interacts. Within k8s0, each installation step is nicely organized as `role` of ansible script. It is very straight-forward to follow.

One more thing, all components are installed as containers including etcd, kubelet, calico, ...

**********************

Current status: installed etcd, kubelet, api server, controller, scheduler, proxy, and calico as network plugin.

Impediments:
* ~~[configmap issue](https://github.com/kubernetes/kubernetes/issues/46768)~~
* ~~[calico issue](https://github.com/projectcalico/calico/issues/825)~~
* [travis docker issue](https://github.com/travis-ci/travis-ci/issues/8104)

**********************

## Prerequisites

For now, only support ubuntu >=14.04 with docker and ansible installed.

## Usage

### Deploy using Vagrant
Use vagrant `vagrant up` in the source root folder.

### Deploy on Travis
Check .travis.yml for more details.

### Deploy on localhost
```
# pull images and put into ./images folder
ansible-playbook -i inventory/local images.yml

# start kubernetes on localhost
ansible-playbook -i inventory/local site.yml
```

### Deploy to a remote host
```
# change ip to the remote host
https://github.com/reachlin/k8s0/blob/master/inventory/single/inventory

# pull images and put into ./images folder
ansible-playbook -i inventory/single images.yml

# start kubernetes on localhost
ansible-playbook -i inventory/single site.yml
```
### Deploy to multi remote hosts
```
# change ips
https://github.com/reachlin/k8s0/blob/master/inventory/multi/inventory

# pull images and put into ./images folder
ansible-playbook -i inventory/multi images.yml

# start kubernetes on localhost
ansible-playbook -i inventory/multi site.yml
```

## Features

* Minimal kubernetes on travis for DevOps.

* Include a local registry or repo, so k8s can be deployed without an internet connection.

* All k8s components running as containers

## Dev. Notes

### Basic steps

etcd -> kubelet -> api server as static pod -> other k8s pods -> calico

### Treak the versions

If you want to change the version of various container images of k8s, pls refer to `registry/defaults/main.yml`.


### References

* k8s install: https://kubernetes.io/docs/getting-started-guides/scratch/#software-binaries
* kargo: https://github.com/kubernetes-incubator/kargo
* etcd: https://github.com/coreos/etcd/releases/
* k8s: https://kubernetes.io/docs/getting-started-guides/scratch/#cluster-naming
* k8s hack: https://github.com/kubernetes/kubernetes/blob/master/hack/local-up-cluster.sh

### Create certificates

```
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=k8s0-ca" -days 99999 -out ca.crt -sha256
openssl x509 -in ca.crt -text -noout
openssl genrsa -out kubelet.key 2048
openssl req -new -key kubelet.key -subj "/CN=kubelet" -out kubelet.csr
openssl x509 -req -in kubelet.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kubelet.crt -days 99999 -sha256
openssl x509 -in kubelet.crt -text -noout
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=apiserver" -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 99999 -sha256 -extensions req_ext -extfile openssl.conf
openssl x509 -in server.crt -text -noout
```

### Useful commands

```
# clean all
docker ps -a --format "{{.ID}}"|xargs -L1 docker rm -f
```

[travis]: https://travis-ci.org/reachlin/k8s0
