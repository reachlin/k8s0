# k8s0 [![Build Status](https://travis-ci.org/reachlin/k8s0.svg)][travis]

kubernetes installation on a fixed configuration

## Prerequisites

For now, only support ubuntu 16.04 with docker and ansible installed.

## Usage

check .travis.yml

## Features

* Minimal kubernetes on travis for DevOps.

* Include a local registry or repo, so k8s can be deployed without an internet connection.

## Dev. Notes

etcd: https://github.com/coreos/etcd/releases/
k8s: https://kubernetes.io/docs/getting-started-guides/scratch/#cluster-naming

### create certificates
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

[travis]: https://travis-ci.org/reachlin/k8s0
