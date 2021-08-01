kubespray를 사용한 kubernetes 설정 및 배포
====

### 사전작업 - 모든 노드
root 로그인 허용
```
$ sudo passwd root
$ sudo vi /etc/ssh/sshd_config
```
```
...
PermitRootLogin yes
...
```

적용
```
sudo systemctl restart sshd
```


### 설치

#### SSH 키 등록
```shell
$ ssh-keygen -t rsa # 입력 없이 enter
$ ssh-copy-id root@192.168.0.200
$ ssh-copy-id root@192.168.0.201
```

#### kubespray 설치
```shell
$ git clone https://github.com/kubernetes-sigs/kubespray.git
$ cd ./kubespray
$ cp -rfp inventory/sample inventory/mycluster
```

#### kubespray 설정(1) - host이름을 바꾸지 않도록 변경
```shell
$ vi ./roles/bootstrap-os/defaults/main.yml
```
- as-is
```yml
## General
# Set the hostname to inventory_hostname
override_system_hostname: true
```
- to-be
```yml
## General
# Set the hostname to inventory_hostname
override_system_hostname: false
```

#### kubespray 환경 실행 및 인벤토리 파일 생성
kubespray의 docker container image를 활용하여 배포할 예정. 아래 두 경로의 mount 필요
1. mount /kubespray : 최신버전의 kubespray 필요. container에 내장된 kubespray 파일들로 배포 시 에러 발생
2. mount /root/.ssh : ansible이 각 node에 접근하기 위한 SSH 키 인증 정보
kubespray 환경 실행
```shell
$ docker pull quay.io/kubespray/kubespray:v2.16.0
$ docker run --rm -it \
  --mount type=bind,source="$(pwd)",dst=/kubespray \
  --mount type=bind,source="${HOME}"/.ssh,dst=/root/.ssh \
    quay.io/kubespray/kubespray:v2.16.0 bash
```
inventory 생성
```
# declare -a IPS=(192.168.0.200 192.168.0.201)
# CONFIG_FILE=/kubespray/inventory/mycluster/hosts.yaml python3 /kubespray/contrib/inventory_builder/inventory.py ${IPS[@]}
# exit # 설정 파일 수정 위해 잠시 종료
```

#### kubespray 설정(2) - 각 node 역할 조정
```sh
$ sudo vi ./inventory/mycluster/hosts.yaml
```
* as-is
```yml
...
  children:
    kube_control_plane:
      hosts:
        node1:
        node2: # 삭제
    kube_node:
      hosts:
        node1: # 삭제
        node2:
...
```
* to-be
```yml
...
  children:
    kube_control_plane:
      hosts:
        node1:
    kube_node:
      hosts:
        node2:
...
```

#### kubespray 설정(3) - Add-Ons 설정
```shell
$ vi ./inventory/mycluster/group_vars/k8s_cluster/addons.yml
```
```yml
...
dashboard_enabled: true
metrics_server_enabled: true
ingress_nginx_enabled: true
...
```

#### kubernetes 클러스터 배포
kubespray container 환경 재진입하여 아래 커맨드 수행
```
# ansible all -i inventory/mycluster/hosts.yaml -m ping # SUCCESS 출력 확인
# ansible-playbook -i inventory/mycluster/hosts.yaml --private-key /root/.ssh/id_rsa --become --become-user=root cluster.yml
```
결과(약 11분 소요) :
```
Saturday 31 July 2021  21:14:19 +0000 (0:00:00.061)       0:11:04.442 *********

PLAY RECAP ****************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
node1                      : ok=643  changed=131  unreachable=0    failed=0    skipped=1177 rescued=0    ignored=2
node2                      : ok=383  changed=76   unreachable=0    failed=0    skipped=648  rescued=0    ignored=1
```
kubespray container 종료
```
# exit
```

#### kubernetes 계정 설정 및 클러스터 확인 - master node에서 실행(≒kube_control_plane ROLE을 가진 Node)
kubernetes config 설정
```shell
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

클러스터 정보 확인
```
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:6443
```

클러스터 노드  확인
```
$ kubectl get nodes -o wide
NAME    STATUS   ROLES                  AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
node1   Ready    control-plane,master   9m42s   v1.21.3   192.168.0.200   <none>        Ubuntu 20.04.2 LTS   5.4.0-77-generic   docker://20.10.7
node2   Ready    <none>                 8m39s   v1.21.3   192.168.0.201   <none>        Ubuntu 20.04.2 LTS   5.4.0-77-generic   docker://20.10.7
```

### OS 설정 확인(optional) - 모든 노드
Kubespray에서 알아서 해주지만, Kubernetes 설치에 필수적인 OS 설정 적용 여부 확인 방법

#### swap off 확인
정상 예
```
$ cat /proc/swaps
Filename                                Type            Size    Used    Priority
```
비정상 예
```
$ cat /proc/swaps
Filename                                Type            Size    Used    Priority
/swap.img                               file            2097148 0       -2
```
위 경우, 아래 커맨드 실행 후 재확인
```
swapoff -a
```

#### ip forward 설정 확인
정상 예
```
$ cat /proc/sys/net/ipv4/ip_forward
1
```
비정상 예
```
$ cat /proc/sys/net/ipv4/ip_forward
0
```
위 경우, 아래 커맨드 실행 후 재확인
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```



### 참고링크
- https://github.com/kubernetes-sigs/kubespray
- https://github.com/kubernetes-sigs/kubespray/blob/master/docs/getting-started.md
- https://github.com/kubernetes-sigs/kubespray/blob/master/docs/setting-up-your-first-cluster.md
- https://ssup2.github.io/record/Kubernetes_%EC%84%A4%EC%B9%98_kubespray_Ubuntu_18.04_OpenStack/
- https://lindarex.github.io/kubernetes/ubuntu-kubernetes-installation-with-kubespray/
- https://www.whatwant.com/entry/Kubernetes-Vagrant-VirtualBox-Kubespray
- https://memory-hub.tistory.com/8
