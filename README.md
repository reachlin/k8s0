# k8s0: Kubernetes Ground Zero [![Build Status](https://travis-ci.org/reachlin/k8s0.svg)][travis]

It's a minimal kubernetes installation on a fixed configuration.

This project is like [MiniKube](https://kubernetes.io/docs/getting-started-guides/minikube/) or minimal version of [Kargo](https://github.com/kubernetes-incubator/kargo), but with the capability to run on Travis. 

So the user can try this on local host or VM, or use it on Travis for kubernetes DevOps. And the ansible script make it easy to tweak the installation steps for more usages.

Another difference from MiniKube is this project use the same image as the full-fledged k8s, no wrapper or any code changes. All source code is ansible script to deploy and configure k8s.

One more thing, all components are installed as containers including etcd, kubelet, calico, ...

**********************

Current status: installed etcd, kubelet, api server, controller, scheduler, and proxy, but calico still not working...

impediments:
* ~~[configmap issue](https://github.com/kubernetes/kubernetes/issues/46768)~~
* ~~[calico issue](https://github.com/projectcalico/calico/issues/825)~~
* have to add a flag for k8s0 to run on Travis, because its docker has problems with 'shared' flag of volume mounted. `travis_docker_bug: rw,shared` [travis docker issue](https://github.com/travis-ci/travis-ci/issues/8104)

**********************

## Prerequisites

For now, only support ubuntu >=14.04 with docker and ansible installed.

## Usage

### Deploy using Vagrant
Use vagrant `vagrant up` in the source root folder.

### deploy on Travis
Check .travis.yml for more details.

### deploy on localhost
```
# pull images and put into ./images folder
ansible-playbook -i inventory/local images.yml

# start kubernetes on localhost
ansible-playbook -i inventory/local site.yml
```

### deploy to a remote host
```
# change ip to the remote host
https://github.com/reachlin/k8s0/blob/master/inventory/single/inventory

# pull images and put into ./images folder
ansible-playbook -i inventory/single images.yml

# start kubernetes on localhost
ansible-playbook -i inventory/single site.yml
```

## Features

* Minimal kubernetes on travis for DevOps.

* Include a local registry or repo, so k8s can be deployed without an internet connection.

* All k8s components running as containers

## Dev. Notes

### Basic steps

etcd -> kubelet -> api server as static pod -> other k8s pods -> calico

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
