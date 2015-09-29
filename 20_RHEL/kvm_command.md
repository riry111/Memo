##KVM 使い方メモ

* IPアドレスの確認

~~~
1) 定義している仮想マシンの確認
	virsh list --all
	
2) MACアドレス確認
	virsh domiflist <ホスト名>

3) ARP より IPアドレス確認 (2)の MACアドレスを活用
	arp -e
~~~

* 仮想マシンの起動・停止

~~~
1) 定義している仮想マシンの確認
	virsh list --all

2) 仮想マシンの起動
	virsh start  <ホスト名>

3) 仮想マシンの停止
	virsh shutdown <ホスト名>

4) 仮想マシンの強制終了
	virsh destroy <ホスト名>
~~~

* 仮想マシンの 削除 / Clone 作成

~~~
1) 既存仮想マシンの削除
	virsh undefine <ホスト名>

2) 仮想マシンの Clone
	virt-clone --original rhel7-min --name cicd01 \
	--file /var/lib/libvirt/images/cicd01.qcow2
~~~

### 仮想マシンに固定IPを払い出す
* default ネットワークの設定を変更

~~~
# virsh net-edit default
~~~

* <dhcp> 以下に固定アドレスを登録する

~~~
<network>
  <name>default</name>
  <uuid>635f418b-5c9b-40ed-93bc-f7b598352eb6</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:77:71:1e'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.99'/>
      <host mac='52:54:00:06:0e:9d' name='docker-reg' ip='192.168.122.201'/>
      <host mac='52:54:00:a4:42:aa' name='docker' ip='192.168.122.202'/>
      <host mac='52:54:00:90:9e:ad' name='repo02' ip='192.168.122.202'/>
      <host mac='52:54:00:11:77:04' name='cicd01' ip='192.168.122.204'/>
      <host mac='52:54:00:ab:8d:2a' name='cicd02' ip='192.168.122.205'/>
    </dhcp>
  </ip>
</network>
~~~

*ネットワークを再起動する

~~~
# virsh net-destroy default
# virsh net-start default
~~~

* 固定アドレスが登録されている事を確認する

~~~
# cat /var/lib/libvirt/dnsmasq/default.hostsfile
52:54:00:06:0e:9d,192.168.122.201,docker-reg
52:54:00:a4:42:aa,192.168.122.202,docker
52:54:00:90:9e:ad,192.168.122.202,repo02
52:54:00:11:77:04,192.168.122.204,cicd01
52:54:00:ab:8d:2a,192.168.122.205,cicd02
~~~

* VMを停止・起動して指定したアドレスが設定される事を確認する

