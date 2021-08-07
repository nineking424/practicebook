vagrant를 사용한 Multi-Node Cluster 가상환경 구축
====

**※ 기본 명령어**
```
vagrant box list
vagrant init
vagrant up
vagrant status
vagrant halt
vagrant destroy
vagrant reload
vagrant provision # vagrant up --provision 과 동일
```

### 1. vagrant 설치
- https://www.vagrantup.com/downloads
- 본 문서 작성일 기준 2.2.18, 64bit
- C:\vagrant> 경로에서 cmd 실행 후 진행
### 2. vagrant init
```
vagrant init
```
### 3. Vagrantfile 
- master 1대, worker 3대
- 기본 admin 계정은 vagrant/vagrant
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

N=3
Vagrant.configure("2") do |config|


  config.vm.define "w-k8s-master" do |cfg|
    cfg.vm.box = "ubuntu/focal64"
    cfg.vm.hostname = "master"
    cfg.vm.network "public_network", ip: "192.168.0.210"
    # manual ip
    cfg.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.cpus = 2
      vb.memory = 2048
    end
    cfg.vm.provision "shell", inline: <<-SHELL
      sed -i 's/kr.archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
      sed -i 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
      apt-get update
      apt-get upgrade -y
      apt-get install net-tools
    SHELL
  end

  (1..N).each do |i|
    config.vm.define "w-k8s-worker#{i}" do |cfg|
      cfg.vm.box = "ubuntu/focal64"
      cfg.vm.hostname = "worker#{i}"
      cfg.vm.network "public_network", ip: "192.168.0.21#{i}"
      cfg.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.cpus = 1
        vb.memory = 1024
      end
      cfg.vm.provision "shell", inline: <<-SHELL
        sed -i 's/kr.archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
        sed -i 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
        apt-get update
        apt-get upgrade -y
        apt-get install net-tools
      SHELL
    end
  end

end
```

### 4. vagrant up - vm 환경구성
```
vagrant up
```

### 5. vagrant status - vm 설치 확인
```
C:\vagrant>vagrant status
Current machine states:

w-k8s-master              running (virtualbox)
w-k8s-worker1             running (virtualbox)
w-k8s-worker2             running (virtualbox)
w-k8s-worker3             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```


#### Trouble shooting - (1)
사용할 수 있는 network interface가 없는 경우 발생,
```
 Bringing virtual machine 'default' up with 'virtual-box' provider..
 [default] clearing any previously set forwarded ports...
 [default] clearing any previously set network interfaces...
 [default] Available bridged network interfaces:
 [default] what interface should the network bridge to?
```
Virtual Box 재설치하여 해결
- Vagrant not showing network interface after upgrade to Windows 10 : https://github.com/hashicorp/vagrant/issues/6076
#### Trouble shooting - (2)
공유폴더 /vagrant 경로를 mount 할 수 없는 경우 발생,
```
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000,_netdev vagrant /vagrant

The error output from the command was:

: Invalid argument
```
vagrant-vbguest 설치하려 해결
- Vagrant 공유 폴더 문제(mount.vboxsf 관련) - vagrant-vbguest 플러그인 :https://javaworld.co.kr/96
> vagrant-vbguest 플러그인은 guest machine과 VirtualBox host의 Guest Additions 버전이 다를 경우에 알맞은 버전을 설치해 주는 플러그인입니다.


**※ 참고링크**
- vagrant : https://judo0179.tistory.com/120
- Vagrant 여러 개의 VM 생성하기 : https://www.whatwant.com/entry/Vagrant-Multi-VM
