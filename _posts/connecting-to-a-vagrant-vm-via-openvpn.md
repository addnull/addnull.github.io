---
layout: post
title: Connecting to a Vagrant VM via OpenVPN
modified: 2015-12-13
tags: [vagrant, vm, vpn, network]
---
/* Last modified at {{ page.modified | date: '%B %d, %Y' }} */

# 환경

이 글의 내용은 아래와 같은 환경에서 확인된 내용이다.

{% highlight bash %}
OS (HOST) : Mac OS X (10.11.1)
OS (VM)   : Ubuntu 14.04.3
           (http://cloud-images.ubuntu.com/vagrant/trusty/ 에서 20151208.1.0)
{% endhighlight %}

# 시작

Server, client 기반의 서비스 개발에 있어서, 개발자 1인이 온전하게 "혼자 사용"하는, "격리"된 개발 환경을 가지는게 좋다고 본다.

즉, 개발자마다 server와 client 전체가 포함된 "자신만의 full set"가 있으면, 안의 내용을 마음껏 수정, 테스트하고, 심지어 전체 재설치(reprovisioning)를 해볼 수 있기 때문이다.

이런 full set를 구축하기 위한 방법에 여러가지가 있지만, 가장 흔한 방법 중에 하나가 local(laptop, desktop)에서 Vagrant VM 기반으로 구축하는 것이다.

간단하게 Vagrant VM 사용의 장점으론,

1. local 환경이므로, 초기 비용(laptop, desktop 구매) 이후에는 비용 지출이 없다.
1. 전체 설치(provisioning) 반복이 쉬워진다. 즉, 뭔가 잘못되었다 싶으면, 모든 VM을 폐기(destroy)해버리고, reprovisioning 할 수 있다.
1. 모든 개발자가 동일한 개발 환경(앞서 말한 full set)을 보장한다.

Vagrant에 대한 자세한 내용은 여기서 다루지 않는다. ([official site](https://www.vagrantup.com/) 참고)

# 문제

만약 client(frontend)가 web client라면 일반적으로 아래와 같이 두 가지 형태로 구축할 것이다.

1. 하나의 VM에 server, client를 모두 설치하되, server, client 간에 port를 서로 다르게 설정한다.
1. 두 개의 VM에 하나는 server를, 다른 하나는 client을 설치한다.

"하나의 VM" 방법은 client에서 server IP 주소를 "127.0.0.1"로 설정하면 문제 없을 것이다.

그리고 "두 개의 VM" 방법은 client에서 "127.0.0.1" 대신에 server VM의 "private IP address"를 사용하면 문제가 없다.

하지만, 만약 client가 iOS/Android app이라서 local(laptop, desktop) 외부의 환경이라면, 이럴 경우의 server, client 간의 통신(network)은 아래 그림과 같다.

<a href="/images/connecting-to-a-vagrant-vm.png">
    <img src="/images/connecting-to-a-vagrant-vm.png" alt="">
</a>

문제는 local 외부에서는 local 내부의 Vagrant VM에 직접 접근이 불가능하다는 점이다.

즉, Vagrant VM에 할당된 "private IP address"는 일반적으로 local 외부에서는 접근이 불가능하다.

# 해결

해결 방법으론 local 외부에 있는 iOS/Android 기기를 local 내부의 network의 구성에 포함되게 만들어야한다.

구체적으로 OpenVPN server용 Vagrant VM을 추가로 생성해서, iOS/Android 기기가 VPN으로 연결한다.

전체 구조는 아래 그림과 같다.

<a href="/images/connecting-to-a-vagrant-vm-via-openvpn.png">
    <img src="/images/connecting-to-a-vagrant-vm-via-openvpn.png" alt="">
</a>

Local device와 client의 device를 동일한 Wi-Fi network 안에 둔 상태에서, 아래와 같이 진행한다.

### STEP 1

Local device에 두 개의 Vagrant VM을 생성한다. 하나는 "OpenVPN server"용, 다른 하나는 현재 "개발 중인 서비스의 server"용이다.

"개발 중인 서비스의 server"에 관련된 내용은, 각자 상황에 따라 다를 것이 때문에, 여기에 포함하지 않았다.

"OpenVPN server"의 Vagrantfile과 "provisioning.sh"은 다음과 같다. (중간에 빠진 설정 파일을 포함한 모든 파일은 내 [GitHub](https://github.com/addnull/miscellaneous/blob/master/conneting_to_a_vagrant_vm_via_openvpn/)에 올려두었다.)

**Vagrantfile**
{% highlight ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provider "virtualbox" do |v|
	v.cpus = 1
    v.memory = 128
  end

  config.vm.provision "shell", path: "provisioning.sh"
  config.vm.network "private_network", ip: "192.168.100.99"
  config.vm.network "forwarded_port", guest: 1194, host: 1194, protocol: "udp"
end
{% endhighlight %}

**provisioning.sh**
{% highlight bash %}
#!/usr/bin/env bash
set -e

cp -R /vagrant/configuration ~/

export DEBIAN_FRONTEND=noninteractive

apt-get -y install gdebi-core
apt-get -y install openvpn
apt-get -y install easy-rsa

PP=~/configuration/etc
cp -R ${PP}/openvpn/*          /etc/openvpn/
cp    ${PP}/sysctl.conf        /etc/sysctl.conf
cp    ${PP}/default/ufw        /etc/default/ufw
cp    ${PP}/ufw/before.rules   /etc/ufw/before.rules

ufw allow ssh
ufw allow 1194/udp

ufw --force enable

service openvpn start

reboot
{% endhighlight %}

### STEP 2

이제 client device에 OpenVPN client app을 설치한다. (여기서는 iOS 기준으로 설명한다.)

'App Store'에서 '[OpenVPN](https://itunes.apple.com/us/app/openvpn-connect/id590379981)'을 설치한다.

처음에 설치해서 실행하면 아래와 같은 화면이 보인다.

<a href="/images/openvpn-main-screen.png">
    <img src="/images/openvpn-main-screen.png" alt="" width="400">
</a>

### STEP 3

Local device에 terminal에서 아래처럼 명령어를 실행해서 IP 주소를 찾아낸다.

{% highlight bash %}
➜  ifconfig en0
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
    ether 11:22:33:44:55:66
    inet6 1234::1234:1234:1234:1234%en0 prefixlen 64 scopeid 0x4
    inet 192.168.10.2 netmask 0xffffff00 broadcast 192.168.10.255
    nd6 options=1<PERFORMNUD>
    media: autoselect
    status: active
  
  
  
➜  ifconfig en1
en1: flags=963<UP,BROADCAST,SMART,RUNNING,PROMISC,SIMPLEX> mtu 1500
    options=60<TSO4,TSO6>
    ether 22:33:44:55:66:77
    media: autoselect <full-duplex>
    status: inactive
  
  
  
➜  ifconfig en2
en2: flags=963<UP,BROADCAST,SMART,RUNNING,PROMISC,SIMPLEX> mtu 1500
    options=60<TSO4,TSO6>
    ether 33:44:55:66:77:88
    media: autoselect <full-duplex>
    status: inactive
  
  
  
➜  ifconfig en3
ifconfig: interface en3 does not exist
  
  
  
➜  ifconfig en4
ifconfig: interface en4 does not exist
{% endhighlight %}

위의 예시에서 'en0'를 제외하면 모두 유효한 IP 주소('inet' 항목 값, '192.168.10.2')가 없지만, 만약 여러개의 유효한 IP 주소가 뜬다면, iOS 기기와 같은 network에서 할당한 IP 주소인지 직접 판단해야한다.

### STEP 4

미리 git에 만들어둔 OpenVPN configuration file(아래 URL)을 다운 받은 후, 'remote [YOUR_IP_ADDRESS] 1194'라는 항목을 'remote 192.168.10.2 1194'라고 수정한다.

https://github.com/addnull/miscellaneous/blob/master/conneting_to_a_vagrant_vm_via_openvpn/client.ovpn

### STEP 5

iOS 기기를 Host(Mac OS X) 기기에 USB로 연결한 후에 'iTunes'를 실행해서, OpenVPN configuration file을 추가한다. (아래 'screenshot' 참고.)

<a href="/images/copy-openvpn-configuration-via-itunes.png">
    <img src="/images/copy-openvpn-configuration-via-itunes.png" alt="" width="700">
</a>

### STEP 6

다시 client device로 돌아가서 'OpenVPN' App을 실행하면, 화면이 아래와 같이 달라져 있다.

<a href="/images/openvpn-main-screen-with-configuration-0.png">
    <img src="/images/openvpn-main-screen-with-configuration-0.png" alt="" width="400">
</a>

### STEP 7

'+'를 눌러서 configuration을 등록한다.

<a href="/images/openvpn-main-screen-with-configuration-1.png">
    <img src="/images/openvpn-main-screen-with-configuration-1.png" alt="" width="400">
</a>

### STEP 8

'toggle button'을 오른쪽으로 옮겨서 OpenVPN server(Vagrant VM)에 접속한다.

<a href="/images/openvpn-main-screen-with-configuration-2.png">
    <img src="/images/openvpn-main-screen-with-configuration-2.png" alt="" width="400">
</a>

이제 local 외부에 있는 iOS/Android 기기가 local 내부의 network의 구성에 포함되었다.

즉, iOS/Android app(client)이 Vagrant VM로 구축된 server에 접속 가능해졌다.

아래 그림과 같이 통신이 이뤄지며, 참고로 iOS의 경우 OpenVPN 연결한 이후부터 모든 network traffic이 OpenVPN server를 통해서 흘러간다.

<a href="/images/connecting-to-a-vagrant-vm-via-openvpn.png">
    <img src="/images/connecting-to-a-vagrant-vm-via-openvpn.png" alt="">
</a>

# 남은 이야기

Local device가 접속한 network(예: 커피샵 WiFi)에 따라서 IP 주소가 변경되면, 기존에 등록한 OpenVPN configuration을 삭제하고, 다시 등록해야한다.

기존에 등록한 OpenVPN configuration을 삭제하기 위해선 OpenVPN App을 실행 후, 아래 'screenshot'처럼 진행한다.

<a href="/images/openvpn-main-screen-delete-configuration-0.png">
    <img src="/images/openvpn-main-screen-delete-configuration-0.png" alt="" width="400">
</a>

<a href="/images/openvpn-main-screen-delete-configuration-1.png">
    <img src="/images/openvpn-main-screen-delete-configuration-1.png" alt="" width="400">
</a>

<a href="/images/openvpn-main-screen-delete-configuration-2.png">
    <img src="/images/openvpn-main-screen-delete-configuration-2.png" alt="" width="400">
</a>

위의 과정을 거치면 최초 App을 설치했던 화면(아래)이 된다.

<a href="/images/openvpn-main-screen.png">
    <img src="/images/openvpn-main-screen.png" alt="" width="400">
</a>

이제, OpenVPN configuration에서 "remote [IP_ADDRESS] 1194" 항목을 수정해서 다시 등록하면된다. (위에서 "STEP 4~8" 과정)
