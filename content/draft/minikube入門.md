---
title: minikube入門
tags:
- Kubernetes
- k8s
id: k8s-minikube
draft: true
---

minikube を構築してみる。 Mac 前提で書きます。

- minikube の意義
- minikube の構築

# minikube の意義

[Docker for Mac](https://docs.docker.com/docker-for-mac/) あるからいらねーじゃん、と思っていた。  
だがしかし、minikube と Docker for Mac では仮想マシンの種類が異なる。  
仮想マシンの種類は以下。（呼び方がいろいろあるみたいなので多少の違いはご愛嬌）

- ファームウェア型： LPAR 、 vPars
    - ハードウェアにファームウェアをインストールし、ハードウェア資源を区切ってOSに使わせる方式
- ハイパーバイザ型： Hyper-V 、 Xen 、 bHyve
    - ハードウェアにゲスト OS を直接コントロールするソフトウェアをインストールし、ハードウェア資源を小さなハードウェアの形で標準化・抽象化してOSに使わせる方式
- アプリケーション型： VMwarePlayer 、 VirtualBox
    - ハードウェアに何らかのホスト OS （ Linux など）がインストールされていて、その上にゲスト OS をコントロールするソフトウェアをインストールする方式

それぞれ以下のようなイメージ。

ファームウェア型
<table>
<tr>
<td align="center">ゲストOS</td><td align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="2" align="center">ファームウェア</td>
</tr>
<tr>
<td colspan="2" align="center">ハードウェア</td>
</tr>
</table>

ハイパーバイザ型
<table>
<tr>
<td align="center">管理OS</td><td align="center">ゲストOS</td><td align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="3" align="center">仮想マシンソフトウェア</td>
</tr>
<tr>
<td colspan="3" align="center">ハードウェア</td>
</tr>
</table>

アプリケーション型
<table>
<tr>
<td align="center">ゲストOS</td><td align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="2" align="center">仮想マシンソフトウェア</td>
</tr>
<tr>
<td colspan="2" align="center">ホストOS</td>
</tr>
<tr>
<td colspan="2" align="center">ハードウェア</td>
</tr>
</table>

Docker for Mac はハイパーバイザー型で、 minikube はアプリケーション型になる。  
といっても今回はコンテナ文脈なので、以下のようになる。

Docker for Mac
<table>
<tr>
<td align="center">コンテナ</td><td align="center">コンテナ</td>
</tr>
<tr>
<td colspan="2" align="center">ゲストOS（Alpine Linux？）</td>
</tr>
<tr>
<td colspan="2" align="center">MacOS（Hypervisor.framework + HyperKit + LinuxKit）</td>
</tr>
<tr>
<td colspan="2" align="center">ハードウェア</td>
</tr>
</table>

minikube
<table>
<tr>
<td align="center">コンテナ</td><td align="center">コンテナ</td>
</tr>
<tr>
<td colspan="2" align="center">ゲストOS</td>
</tr>
<tr>
<td colspan="2" align="center">仮想マシンソフトウェア（ VirtualBox とか）</td>
</tr>
<tr>
<td colspan="2" align="center">MacOS</td>
</tr>
<tr>
<td colspan="2" align="center">ハードウェア</td>
</tr>
</table>

>Docker for Mac は MacOS においてLinux KVMに相当する仮想化ハイパーバイザを利用する `Hypervisor.framework` を使って実現してるらしい。  
>Docker for Mac のゲスト OS へは SSH できないので `screen` でアクセスする必要がある。（ [参考](https://forums.docker.com/t/host-path-of-volume/12277/10) ）  
>`screen` は `ctrl + k + a` で抜ける。
>
>```
>$ screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
>linuxkit-025000000001:~# hostname
>linuxkit-025000000001
>linuxkit-025000000001:~# uname
>Linux
>linuxkit-025000000001:~# uname -a
>Linux linuxkit-025000000001 4.9.125-linuxkit #1 SMP Fri Sep 7 08:20:28 UTC 2018 x86_64 Linux
>linuxkit-025000000001:~# cat /etc/os-release
>PRETTY_NAME="Docker for Mac"
>linuxkit-025000000001:~# docker -v
>sh: docker: not found
>linuxkit-025000000001:~# cat /etc/passwd
>root:x:0:0:root:/root:/bin/sh
>linuxkit-025000000001:~# whoami
>root
>linuxkit-025000000001:~# find / -name docker -type f
>/containers/services/docker/lower/usr/local/bin/docker
>/containers/services/docker/rootfs/usr/local/bin/docker
>linuxkit-025000000001:~# /containers/services/docker/lower/usr/local/bin/docker -v
>Docker version 18.09.2, build 6247962
>linuxkit-025000000001:~# /containers/services/docker/rootfs/usr/local/bin/docker -v
>Docker version 18.09.2, build 6247962
>```
>
>なぞに `docker` が 2 つあってパスが通っていないが、確認はできそう。
>
>```
>linuxkit-025000000001:~# pstree
>init-+-containerd-+-containerd-shim---acpid
>     |            |-containerd-shim---diagnosticsd
>     |            |-containerd-shim-+-docker-init---entrypoint.sh-+-logwrite---+
>     |            |                 |                             |-logwrite---+
>     |            |                 |                             `-start-docke+
>     |            |                 |-rpc.statd
>     |            |                 |-rpcbind
>     |            |                 `-transfused.sh---transfused
>     |            |-containerd-shim---host-timesync-d
>     |            |-containerd-shim---kmsg
>     |            |-containerd-shim---sntpc
>     |            |-containerd-shim
>     |            |-containerd-shim---trim-after-dele
>     |            |-containerd-shim---vpnkit-forwarde
>     |            |-containerd-shim---vsudd
>     |            `-containerd-shim---logwrite
>     |-memlogd
>     `-rungetty.sh---login---sh---sh---sh---sh---pstree
>```
>
>`ps` とかも打ったけどなんかいろいろ動いてる。今度もうちょい見ようかな、、、  
>何が言いたいかというと、 Mac 上で **コンテナランタイムを構築したホスト OS ( Linux ) にログインして何か検証したい場合は minikube 使う** 、だ。

自分の理解だと、以下。

- Docker for Mac は、k8s でコンテナを動かす環境
    - 軽くちゃっちゃと Pod とか Service とか動かしたい人
- minikube は、ノード上の k8s オーケストレーションレイヤも意識した検証をしたい場合に使う環境
    - cnitool とか使って CNI Plugins 自作してみたい人
    - ローカルPC （ Mac ）しか環境ない人

# minikube の構築

前置きが長くなったが、 minikube を構築してみる。  
すでに Docker for Mac を使っている人は注意が必要。

1. Docker for Mac の設定（ Preference ） -> Kubernetes -> Enable Kubernetes のチェックを外す
    - 先に Docker for Mac の Kubernetes を有効にしていると minikube のインストール時にエラーが出るので無効にする
2. `$ brew install docker-machine-driver-hyperkit`
    - いろいろあるが、ここでは HyperKit を使うことにする
    - Hyperkit 入ってない人は `brew install hyperkit` を先に
3. `$ brew cask install minikube`
    - `$ which minikube`
4. `$ minikube start --vm-driver=hyperkit`
    - エラーが出たら案内に従って `$ sudo chown root:wheel /usr/local/bin/docker-machine-driver-hyperkit && sudo chmod u+s /usr/local/bin/docker-machine-driver-hyperkit` を実行
5. `$ kubectl config get-contexts`
    - `minikube` の存在を確認
6. `$ kubectl config use-context minikube`
    `set-context` じゃないよ

`$ kubectl config view` で kubeconfig の内容を確認する。

```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://localhost:6443
  name: docker-for-desktop-cluster
- cluster:
    certificate-authority: /Users/xxxx/.minikube/ca.crt
    server: https://192.168.64.2:8443
  name: minikube
contexts:
- context:
    cluster: docker-for-desktop-cluster
    user: docker-for-desktop
  name: docker-for-desktop
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: docker-for-desktop
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: minikube
  user:
    client-certificate: /Users/xxxx/.minikube/client.crt
    client-key: /Users/xxxx/.minikube/client.key
```

ここでは、 minikube のサーバが `https://192.168.64.2:8443` であることがわかる。  
念のため `$ minikube status` でも確認。

```
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.64.2
```
  
`$ minikube ssh` してみる。

```
$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ uname -a
Linux minikube 4.15.0 #1 SMP Fri Feb 15 19:27:06 UTC 2019 x86_64 GNU/Linux

$ cat /etc/os-release
NAME=Buildroot
VERSION=2018.05
ID=buildroot
VERSION_ID=2018.05
PRETTY_NAME="Buildroot 2018.05"

$ cat /etc/passwd
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
rkt:x:1000:1000:-:/home/rkt:/bin/bash
docker:x:1001:1001:-:/home/docker:/bin/bash
dbus:x:1002:1002:DBus messagebus user:/var/run/dbus:/bin/false
sshd:x:1003:1003:SSH drop priv user:/:/bin/false
systemd-bus-proxy:x:1004:1008:Proxy D-Bus messages to/from a bus:/:/bin/false
systemd-journal-gateway:x:1005:1009:Journal Gateway:/var/log/journal:/bin/false
systemd-journal-remote:x:1006:1010:Journal Remote:/var/log/journal/remote:/bin/false
systemd-journal-upload:x:1007:1011:Journal Upload:/:/bin/false
systemd-network:x:1008:1012:Network Manager:/:/bin/false
systemd-resolve:x:1009:1013:Network Name Resolution Manager:/:/bin/false

$ whoami
docker

$ docker info
Containers: 18
 Running: 18
 Paused: 0
 Stopped: 0
Images: 9
Server Version: 18.06.2-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
runc version: N/A (expected: 69663f0bd4b60df09991c08812a60108003fa340)
init version: N/A (expected: )
Security Options:
 seccomp
  Profile: default
Kernel Version: 4.15.0
Operating System: Buildroot 2018.05
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 1.944GiB
Name: minikube
ID: KW2Q:FAVR:DHAI:Y3PN:K5UT:DHDN:QZVW:T6IP:OFCD:BUYM:WC5M:JAH2
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
 provider=hyperkit
Experimental: false
Insecure Registries:
 10.96.0.0/12
 127.0.0.0/8
Live Restore Enabled: false

$ docker image ls
REPOSITORY                                TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy                     v1.13.3             98db19758ad4        2 weeks ago         80.3MB
k8s.gcr.io/kube-apiserver                 v1.13.3             fe242e556a99        2 weeks ago         181MB
k8s.gcr.io/kube-controller-manager        v1.13.3             0482f6400933        2 weeks ago         146MB
k8s.gcr.io/kube-scheduler                 v1.13.3             3a6f709e97a0        2 weeks ago         79.6MB
k8s.gcr.io/coredns                        1.2.6               f59dcacceff4        3 months ago        40MB
k8s.gcr.io/etcd                           3.2.24              3cab8e1b9802        5 months ago        220MB
k8s.gcr.io/kube-addon-manager             v8.6                9c16409588eb        12 months ago       78.4MB
k8s.gcr.io/pause                          3.1                 da86e6ba6ca1        14 months ago       742kB
gcr.io/k8s-minikube/storage-provisioner   v1.8.1              4689081edb10        15 months ago       80.8MB

$ docker container ls
CONTAINER ID        IMAGE                                     COMMAND                  CREATED             STATUS              PORTS               NAMES
c34ac54ebe30        gcr.io/k8s-minikube/storage-provisioner   "/storage-provisioner"   12 minutes ago      Up 12 minutes                           k8s_storage-provisioner_storage-provisioner_kube-system_0c2f62e2-34d5-11e9-a0fb-367202d85048_0
aca4e6f0e1cd        f59dcacceff4                              "/coredns -conf /etc…"   12 minutes ago      Up 12 minutes                           k8s_coredns_coredns-86c58d9df4-flw7v_kube-system_0b902db5-34d5-11e9-a0fb-367202d85048_0
8d5dd803d5d0        f59dcacceff4                              "/coredns -conf /etc…"   12 minutes ago      Up 12 minutes                           k8s_coredns_coredns-86c58d9df4-ww7k7_kube-system_0b8eea03-34d5-11e9-a0fb-367202d85048_0
75627a005c8b        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_storage-provisioner_kube-system_0c2f62e2-34d5-11e9-a0fb-367202d85048_0
55de73523719        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_coredns-86c58d9df4-flw7v_kube-system_0b902db5-34d5-11e9-a0fb-367202d85048_0
9e9d1374f0ea        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_coredns-86c58d9df4-ww7k7_kube-system_0b8eea03-34d5-11e9-a0fb-367202d85048_0
c329451e8b9d        98db19758ad4                              "/usr/local/bin/kube…"   12 minutes ago      Up 12 minutes                           k8s_kube-proxy_kube-proxy-86ksf_kube-system_0b8961e8-34d5-11e9-a0fb-367202d85048_0
f9caf5d0fbfc        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_kube-proxy-86ksf_kube-system_0b8961e8-34d5-11e9-a0fb-367202d85048_0
7712fc308727        k8s.gcr.io/kube-addon-manager             "/opt/kube-addons.sh"    12 minutes ago      Up 12 minutes                           k8s_kube-addon-manager_kube-addon-manager-minikube_kube-system_5c72fb06dcdda608211b70d63c0ca488_0
dc13194120b8        3cab8e1b9802                              "etcd --advertise-cl…"   12 minutes ago      Up 12 minutes                           k8s_etcd_etcd-minikube_kube-system_83f9772c4727dcf100ba52affff6c2b7_0
db8adb52fcb2        3a6f709e97a0                              "kube-scheduler --ad…"   12 minutes ago      Up 12 minutes                           k8s_kube-scheduler_kube-scheduler-minikube_kube-system_b734fcc86501dde5579ce80285c0bf0c_0
3c48fde7aceb        0482f6400933                              "kube-controller-man…"   12 minutes ago      Up 12 minutes                           k8s_kube-controller-manager_kube-controller-manager-minikube_kube-system_e3e5fe69ae31c6c79585cd978784a674_0
7bb2bb6939fe        fe242e556a99                              "kube-apiserver --au…"   12 minutes ago      Up 12 minutes                           k8s_kube-apiserver_kube-apiserver-minikube_kube-system_05f3a1757b5a11deaae7be3ff5eb938a_0
cf52409c9376        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_kube-apiserver-minikube_kube-system_05f3a1757b5a11deaae7be3ff5eb938a_0
383f64e4d928        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_etcd-minikube_kube-system_83f9772c4727dcf100ba52affff6c2b7_0
94faf895f1d0        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_kube-controller-manager-minikube_kube-system_e3e5fe69ae31c6c79585cd978784a674_0
fb89e384f035        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_kube-addon-manager-minikube_kube-system_5c72fb06dcdda608211b70d63c0ca488_0
4c5c87a806f1        k8s.gcr.io/pause:3.1                      "/pause"                 12 minutes ago      Up 12 minutes                           k8s_POD_kube-scheduler-minikube_kube-system_b734fcc86501dde5579ce80285c0bf0c_0

$ su -
# whoami
root
# exit
logout
$ exit
logout
```

Docker for Mac の k8s 環境と違い、 Docker ベースでコンポーネントが構築されている模様。  
以下で minikube を停止する。

```
$ minikube stop

$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop   
*         minikube             minikube                     minikube

$ kubectl config use-context docker-for-desktop
```

context を切り替えれば、 Docker for Mac の k8s 環境も使える。  
`minikube delete` でホストOS しばける？

```
minikube start \
--vm-driver=hyperkit \
--network-plugin=cni \
--enable-default-cni \
--container-runtime=containerd \
--bootstrapper=kubeadm
```

# 参考

- https://kubernetes.io/docs/setup/minikube/