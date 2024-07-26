---
title: "自作基盤メモ"
type: "note"
category: []
description: ""
thumbnail: ""
publish: false
---

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714533363-640x336.png" alt="" width="640" height="336" class="alignnone size-medium wp-image-1262128" />

## はじめに
世の中ではGrafana Weekということで、Raspberry Pi 5複数台をクラスタリングしてKubernetesを作成し、Grafanaを載せてみたいと思います。

というのは冗談ですが、最近趣味で安価に常駐プロセスをデプロイできるホスティング環境に悩んでいました。常駐しないなら最近はゼロコールドスタートなV8 Isolateを使ったCloudflare WorkersやDeno Deployが無料枠が大きくいい感じです。
一方常駐プロセスはHerokuの無料プランがなくなりました。AWS AppRunnerは起動時間を人間が稼働している時間のみに絞っても10$はかかります。fly.ioは、Legacy hobby planでCPU-1x 256mb VM 3つと3 GB 永続ボリュームストレージは無料で扱えます。fly.ioはCLIもよくできているので、軽い検証の場合こちらで良さそうです。

より永続化領域とメモリが潤沢な環境を求めて、自前でコンテナオーケストレーション基盤を作ることにしました。

Kubernetesクラスタ自体は4年前くらいにRPi3で作ったことがあります。エコシステム的な難しさもあり、早々に飽きてしまったので今回はそこそこ真面目に構築しました。実務的で使ったことはないので、経験者からすると拙いのとN番煎じですが、ご参考にして頂けたら幸いです。

以下の点ご理解お願いします。

* クラウドでサーバレス開発をしているので、電気やハード的なところはあまり参考にならないと思います
* 本構成は動作を保証しません。特に電気的な部分は、他の記事と合わせて確認するのが良いと思います

以後Raspberry Pi 5は、RPi5と表記します。


## 構成

### ゴール

Mac(Kubernetesクラスタ外)から`kubectl port-forward`なしで、Grafanaが閲覧できるようになることを目指します。こんな形です。


![img](https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714537552-2.gif)

### 注意点

本項以降の各手順は、最小2台で進めることを推奨します。理由は、一気に4台でやるよりは、2台でgrafana起動まで確認し、その後2ノード追加する方が何か手戻りがあった際にやり直しがしやすいためです。

### 前提

* PoEは利用しない(RPi5ではまだPoE HATがないため)
* RPi5のストレージには NVMe M.2 SSDを利用
  * microSDで代用可能です、RPi5側で対応しているのと家に余っていたので活用しました
* RPi5のイメージには、Ubuntu Server 24.04 LTS(64bit)を利用
* 電源はスイッチング電源2台で、各スイッチング電源に2つのRPi5を接続


|項|名称|個数|補足|リンク|
|---|---|---|---|---|
|1|Raspberry Pi 5 8GB|4|RPi5本体|[リンク](https://www.switch-science.com/products/9250)|
|2|Raspberry Pi 5用アクティブクーラー|4|RPi5のクーラー|[リンク](https://www.switch-science.com/products/9253)|
|3|Geekworm X1001 V1.1 PCIe NVMe M.2 拡張ボード|4|RPi5とM.2.を接続するための拡張ボード|[リンク](https://amzn.asia/d/0CfLay6)|
|4|KIOXIA EXCERIA G2 SSD-CK1.0N3G2/N 1TB|4|M.2. M.2 NVMe SSD。正確には家に余っている別メーカーのものもありますが大きな差はないので割愛。|[リンク](https://amzn.asia/d/4OSW8eP)|
|5|ORICO M.2 SSD 外付けケース|1|NVMe SSDへイメージを書き込むための外付けケース|[リンク](https://amzn.asia/d/eHkpa4V)|
|6|スイッチング電源 DC5V10A 出力50Wタイプ RWS50B-5|2|RPi5は5V/5Aなので、1つあたりRPi52台接続想定。GPIO 5Vピン2つから電源供給|[リンク](https://akizukidenshi.com/catalog/g/g109086/)
|7|Qiコネクタ|1|スイッチング電源からGPIO経由で電源供給するため利用|[リンク](https://akizukidenshi.com/catalog/g/g109733/)
|8|コネクタ用ハウジング|4|上記のQiコネクトを半分に切って、ケーブル4本を1つのハウジングに納めて計4本作ります|[リンク](https://akizukidenshi.com/catalog/g/g112437/)
|9|TP-Link スイッチングハブ 5ポート PoE+|1|PoE+は利用しません|[リンク](https://amzn.asia/d/2owPNQR)|
|10|Type C ケーブル 0.5M 2本組|2|2本組を2つ購入。計4本|[リンク](https://amzn.asia/d/491DZzx)|
|11|LANケーブル 0.3m 5本|4|5本組を1つ購入。計5本|[リンク](https://amzn.asia/d/8mHTViH)|
|12|microSD 32GB|1|ブート設定を変更するためだけに利用します。恒常的に利用しません。|なし|
|13|microSD読めるカードリーダー|1||なし|
|14|キッチンラック|1|ラズパイを積む用のラック。専用のものは高かったので、キッチンラックで代用しました...RPi5の裏面の導通ある部分がメタルラックに接触するのがリスクなので紙の上にRPi5を載せています|[リンク](https://amzn.asia/d/eGq23dw)|
|15|片プラグ付耐熱コード|2|丸端子圧着済みなので、スイッチング電源に接続しやすいです|[リンク](https://amzn.asia/d/av3Cy42)|



## ブート設定

### はじめに

### SDカードにRasberry Pi OSを書き込む

書き込みには、Rasberry Pi Imagerを利用します。ダウンロードは[こちら](https://www.raspberrypi.com/software/)から可能です。私はWindows版を利用しました。

一時的にしか使わないので、OSはRASBERY PI OS LITE(64-BIT)を利用します

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713571189-640x403.jpg" alt="" width="640" height="403" class="alignnone size-medium wp-image-1262132" />

設定の編集をします

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713571296-640x403.jpg" alt="" width="640" height="403" class="alignnone size-medium wp-image-1262134" />

ユーザーとWi-Fi設定を入れたら、サービスタブに移動します
<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713571468-640x653.jpg" alt="" width="640" height="653" class="alignnone size-medium wp-image-1262136" />


SSHを有効化します。書き込みの段階で指定できるの便利ですね！

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713571545-640x653.jpg" alt="" width="640" height="653" class="alignnone size-medium wp-image-1262138" />

書き込み開始します

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713572144-1-640x403.jpg" alt="" width="640" height="403" class="alignnone size-medium wp-image-1262168" />

書き込み完了まで待ちます。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713572144-640x403.jpg" alt="" width="640" height="403" class="alignnone size-medium wp-image-1262139" />


### M.2 SSDにUbuntu Serverを書き込む

先に恒常的に利用するM.2にもOSを書き込みます。6の外付けケースを利用して、PCに画像のような形で接続します。
<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713572773-640x686.jpg" alt="" width="640" height="686" class="alignnone size-medium wp-image-1262141" />

OSは`Ubuntu Server 24.04 LTS`を利用します
<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714522805-640x444.jpg" alt="" width="640" height="444" class="alignnone size-medium wp-image-1262143" />

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713572906-640x654.jpg" alt="" width="640" height="654" class="alignnone size-medium wp-image-1262144" />

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713572937-640x654.jpg" alt="" width="640" height="654" class="alignnone size-medium wp-image-1262145" />


初回はイメージダウンロードの時間がかかるため、通信環境によっては時間がかかります。一度内部でイメージをダウンロードすれば、2回目以降は数分もかからず終わります。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714522805-1-640x444.jpg" alt="" width="640" height="444" class="alignnone size-medium wp-image-1262146" />

書き込みが終わったら、再度M.2をPCにマウントします。すると画像のようにブートパーティションを認識します。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713584023-640x482.jpg" alt="" width="640" height="482" class="alignnone size-medium wp-image-1262147" />

ブートパーティションを開いたら、config.txtを見つけます。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713584235-640x426.jpg" alt="" width="640" height="426" class="alignnone size-medium wp-image-1262148" />


config.txtに2行追記します
[diff]
[all]
+dtparam=nvme
+dtparam=pciex1
[/diff]

### RPi5にクーラーを取り付け

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713573425-640x480.jpg" alt="" width="640" height="480" class="alignnone size-medium wp-image-1262151" />

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713758751-640x395.jpg" alt="" width="640" height="395" class="alignnone size-medium wp-image-1262152" />


### RPi5 M.2拡張
PCIe NVME M.2 拡張ボードを取り付けます

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713758843-640x471.jpg" alt="" width="640" height="471" class="alignnone size-medium wp-image-1262154" />

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714534398-640x457.jpg" alt="" width="640" height="457" class="alignnone size-medium wp-image-1262155" />

裏は家にあったネジで高さを出すようにしました。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1713758580-640x410.jpg" alt="" width="640" height="410" class="alignnone size-medium wp-image-1262156" />


### 電源の設定

電源はRPi5の2,4ピンに+5V、6,9ピンにGNDに繋ぎます。スイッチング電源1つに2つのRPi5を接続します。以下の図が分かりやすいです。下の直方体はコネクタのハウジングを示してます。


<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714538385.png" alt="" width="475" height="420" class="alignnone size-full wp-image-1262157" />
<br />
※ 弊社のこの領域に強い方に教えて頂きました。ありがとうございます;;

実際はこんな形です。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714531320-640x853.jpg" alt="" width="640" height="853" class="alignnone size-medium wp-image-1262158" />

### ブート設定(EEPROM書き込み)

microSDを入れて、RPi5の電源を入れます。ホスト名は`raspberrypi.local`です。

[bash]
$ ssh shuntaka@raspberrypi.local
(中略)
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
(中略)
Linux raspberrypi 6.6.20+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.6.20-1+rpt1 (2024-03-07) aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
shuntaka@raspberrypi:~ $
[/bash]

ブートローダーを更新します。

[bash]
shuntaka@raspberrypi:~ $ sudo rpi-eeprom-update
*** UPDATE AVAILABLE ***
BOOTLOADER: update available
   CURRENT: Wed Dec  6 18:29:25 UTC 2023 (1701887365)
    LATEST: Fri Feb 16 15:28:41 UTC 2024 (1708097321)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader-2712/default)
            Use raspi-config to change the release.
[/bash]

利用可能な更新がありそうなので、更新を実行します。

[bash]
shuntaka@raspberrypi:~ $ sudo rpi-eeprom-update -d -a
*** PREPARING EEPROM UPDATES ***

BOOTLOADER: update available
   CURRENT: Wed Dec  6 18:29:25 UTC 2023 (1701887365)
    LATEST: Fri Feb 16 15:28:41 UTC 2024 (1708097321)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader-2712/default)
            Use raspi-config to change the release.
   CURRENT: Wed Dec  6 18:29:25 UTC 2023 (1701887365)
    UPDATE: Fri Feb 16 15:28:41 UTC 2024 (1708097321)
    BOOTFS: /boot/firmware
'/tmp/tmp.ov5cNFr05e' -> '/boot/firmware/pieeprom.upd'

UPDATING bootloader.

*** WARNING: Do not disconnect the power until the update is complete ***
If a problem occurs then the Raspberry Pi Imager may be used to create
a bootloader rescue SD card image which restores the default bootloader image.

flashrom -p linux_spi:dev=/dev/spidev10.0,spispeed=16000 -w /boot/firmware/pieeprom.upd
UPDATE SUCCESSFUL
[/bash]


変わっていないので、再起動する必要がありそう
[bash]
shuntaka@raspberrypi:~ $ sudo rpi-eeprom-update
*** UPDATE AVAILABLE ***
BOOTLOADER: update available
   CURRENT: Wed Dec  6 18:29:25 UTC 2023 (1701887365)
    LATEST: Fri Feb 16 15:28:41 UTC 2024 (1708097321)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader-2712/default)
            Use raspi-config to change the release.
[/bash]

`sudo reboot` 後再度実行したところ更新完了していることを確認
[bash]
shuntaka@raspberrypi:~ $ sudo rpi-eeprom-update
BOOTLOADER: up to date
   CURRENT: Fri Feb 16 15:28:41 UTC 2024 (1708097321)
    LATEST: Fri Feb 16 15:28:41 UTC 2024 (1708097321)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader-2712/default)
            Use raspi-config to change the release.
[/bash]



ブート設定を変更します。極度のnanoアレルギーなのでviを設定します。

[bash]
env EDITOR=vi sudo -E rpi-eeprom-config --edit
[/bash]

[bash title="変更前"]
[all]
BOOT_UART=1
POWER_OFF_ON_HALT=0
BOOT_ORDER=0xf461
[/bash]


以下のように設定変更します
[diff title="変更後(diff)"]
[all]
BOOT_UART=1
POWER_OFF_ON_HALT=0
-BOOT_ORDER=0xf461
+BOOT_ORDER=0xf416
+PCIE_PROBE=1
[/diff]

[bash title="実行結果"]
shuntaka@raspberrypi:~ $ env EDITOR=vi sudo -E rpi-eeprom-config --edit
Updating bootloader EEPROM
 image: /lib/firmware/raspberrypi/bootloader-2712/default/pieeprom-2024-02-16.bin
config_src: blconfig device
config: /tmp/tmpp6gugkfa/boot.conf
################################################################################
[all]
BOOT_UART=1
POWER_OFF_ON_HALT=0
BOOT_ORDER=0xf416
PCIE_PROBE=1

################################################################################

*** To cancel this update run 'sudo rpi-eeprom-update -r' ***

*** CREATED UPDATE /tmp/tmpp6gugkfa/pieeprom.upd  ***

   CURRENT: Fri 16 Feb 15:28:41 UTC 2024 (1708097321)
    UPDATE: Fri 16 Feb 15:28:41 UTC 2024 (1708097321)
    BOOTFS: /boot/firmware
'/tmp/tmp.53DnGVNp9Z' -> '/boot/firmware/pieeprom.upd'

UPDATING bootloader.

*** WARNING: Do not disconnect the power until the update is complete ***
If a problem occurs then the Raspberry Pi Imager may be used to create
a bootloader rescue SD card image which restores the default bootloader image.

flashrom -p linux_spi:dev=/dev/spidev10.0,spispeed=16000 -w /boot/firmware/pieeprom.upd
UPDATE SUCCESSFUL
[/bash]

### M.2からブート

一度RPi5の電源をきり、**microSDを抜いて**、M.2を挿します。Ubuntu 24.04 LTSの文字があるので、M.2から起動していることが確認できました。

[bash]
$ ssh shuntaka@pi1.local
shuntaka@pi1.local's password:
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-1004-raspi aarch64)
[/bash]

[bash]
shuntaka@pi4:~$ sudo df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           795M  3.2M  792M   1% /run
/dev/nvme0n1p2  917G  1.9G  878G   1% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p1  505M   85M  420M  17% /boot/firmware
tmpfs           795M   12K  795M   1% /run/user/1000
[/bash]

### カーネルの更新

`Ubuntu Server 24.04 LTS` では更新はありません。

[bash]
$ uname -r
6.8.0-1004-raspi
[/bash]

[bash]
sudo apt update
sudo apt -y upgrade
[/bash]

[bash]
$ uname -r
6.8.0-1004-raspi
[/bash]

`Ubuntu Server 23.10` の場合、クーラーが回りっぱなしで音が大きいので確実にやるのが良いです。
[参考](https://askubuntu.com/questions/1490462/fan-speed-control-on-raspberry-pi-5-not-working-in-ubuntu-23-10)


## ネットワーク設定

### はじめに

以下のような形でアドレスを固定します。

|ホスト名|eth0 IP|
|---|---|
|pi1|192.168.1.1<br>192.168.86.10
|pi2|192.168.1.2|
|pi3|192.168.1.3|
|pi4|192.168.1.4|


### p1のnetplanの設定

[bash]
cat <<EOF | sudo tee /etc/netplan/01-netcfg.yaml > /dev/null
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.1/24
        - 192.168.86.10/24
      nameservers:
        addresses: [8.8.8.8]
EOF
[/bash]

[bash]
sudo netplan apply
[/bash]

### pi2以降のnetplanの設定

pi2~pi4のnetplanの設定をします

[bash]
cat <<EOF | sudo tee /etc/netplan/01-netcfg.yaml > /dev/null
network:
    version: 2
    ethernets:
      eth0:
        dhcp4: false
        addresses:
          - 192.168.1.2/24
          # pi3の場合
          # - 192.168.1.3/24
          # pi4の場合
          # - 192.168.1.4/24
        nameservers:
          addresses:
            - 8.8.8.8
EOF
[/bash]

[bash]
sudo netplan apply
[/bash]

`192.168.1.2`が割り当てられていることを確認
[bash]
shuntaka@pi2:~$ ip a
(中略)
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
(中略)
    inet 192.168.1.2/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever
(中略)
[/bash]

## Kubernetesクラスタの構築

### kubelet kubeadm kubectlのインストール

モジュールのロード設定を実施します

[bash title="全てのノード"]
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
[/bash]

読み込まれているか確認
[bash title="全てのノード"]
lsmod | grep br_netfilter
lsmod | grep overlay
[/bash]

kubelet kubeadm kubectlをインストールします

[bash title="全てのノード"]
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl # <--最初は apt-getでやっていたの確認

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
[/bash]

### cri-oのインストール

[download.opensuse.org](http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/)から要件にあったものを選択

[bash title="全てのノード"]
export OS=xUbuntu_22.04

sudo echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" | sudo tee -a /etc/apt/sources.list.d/cri-o-stable.list >/dev/null
sudo echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.28/$OS/ /" | sudo tee -a /etc/apt/sources.list.d/cri-o.list >/dev/null
 
sudo curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/libcontainers-crio-archive-keyring.gpg
sudo curl -L http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.28/$OS/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/cri-o-apt-keyring.gpg
 
sudo apt update
sudo apt install cri-o cri-o-runc -y

sudo rm -rf /etc/cni/net.d/*

sudo systemctl daemon-reload
sudo systemctl enable crio
sudo systemctl start crio
[/bash]


cri-oの動作確認
[bash title="全てのノード"]
shuntaka@pi1:~$ sudo crictl info
{
  "status": {
    "conditions": [
      {
        "type": "RuntimeReady",
        "status": true,
        "reason": "",
        "message": ""
      },
      {
        "type": "NetworkReady",
        "status": false,
        "reason": "NetworkPluginNotReady",
        "message": "Network plugin returns error: No CNI configuration file in /etc/cni/net.d/. Has your network provider started?"
      }
    ]
  },
  "config": {
    "sandboxImage": "registry.k8s.io/pause:3.9"
  }
[/bash]

Kubernetesクラスタの初期化を実行します。

[bash title="マスターノード"]
$ sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --control-plane-endpoint=192.168.1.1 \
  --apiserver-advertise-address=192.168.1.1 \
  --apiserver-cert-extra-sans=192.168.1.1
[/bash]

ワーカーノード側で実行する必要があるコマンドをメモします。
[bash title="メモする出力結果"]
sudo kubeadm join 192.168.1.1:6443 --token rwmoll.7vfynzwt0wbparok \
        --discovery-token-ca-cert-hash sha256:bf28ffd7b92f505b93409da63dacc610d05e4d3566e50b629738a6a4cf4b259f
[/bash]

`kubectl` が利用できるように、設定を行います
[bash title="マスターノード"]
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
[/bash]

Kubernetesクラスタに参加するコマンドを実行します
[bash title="ワーカーノード"]
sudo kubeadm join 192.168.1.1:6443 --token rwmoll.7vfynzwt0wbparok \
        --discovery-token-ca-cert-hash sha256:bf28ffd7b92f505b93409da63dacc610d05e4d3566e50b629738a6a4cf4b259f
[/bash]

ノードを確認します。この段階ではNotReadyですが、CNIをインストールするとReady状態になります
[bash title="マスターノード"]
$ kubectl get node
NAME   STATUS     ROLES           AGE     VERSION
pi1    NotReady   control-plane   9m21s   v1.28.9
pi2    NotReady   <none>          9s      v1.28.9
[/bash]

### CNI(calico)のインストール

手順と異なり、Ubuntu Server 23.10の場合、以下の設定が必要です
[bash title="全てのノード"]
# Ubuntu Server 23.10利用の場合のみ
sudo apt install linux-modules-extra-raspi -y
[/bash]

CNIをインストールします

[bash title="マスターノード"]
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/custom-resources.yaml -O

# custom-resources.yamlのcidr: 10.244.0.0/16 に修正
kubectl create -f custom-resources.yaml
kubectl get pods -n calico-system
[/bash]

CNIがインストールされ、全てReadyになりました
[bash]
$ kubectl get nodes
NAME   STATUS   ROLES           AGE     VERSION
pi1    Ready    control-plane   17m     v1.28.9
pi2    Ready    <none>          8m16s   v1.28.9
[/bash]


### エイリアス設定

使いやすいように、エイリアスと補完設定を入れます。この設定で省略された`k`コマンドと`tab`での補完が効くようになります。

[bash]
cat <<EOF >> ~/.bashrc
# k8s
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k
EOF
[/bash]


## Helmの設定

### はじめに

今回KubernetesリソースはHelm経由でデプロイすることにします。

### Helmのインストール

[bash title="マスターノード"]
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt update
sudo apt install helm
[/bash]

### Helmfileのインストール

[bash title="マスターノード"]
export VERSION="0.163.1"
wget -O "/tmp/helmfile_${VERSION}_linux_arm64.tar.gz" "https://github.com/helmfile/helmfile/releases/download/v${VERSION}/helmfile_${VERSION}_linux_arm64.tar.gz"
tar -xzvf "/tmp/helmfile_${VERSION}_linux_arm64.tar.gz" -C /tmp

sudo mv /tmp/helmfile /usr/bin/helmfile
sudo chmod 755 /usr/bin/helmfile
[/bash]


## MetalLBの設定

### はじめに

MetalLBは、ベアメタルKubernetes環境向けのロードバランサーでKubernetesクラスタ外からアクセス出来るようにIPを割り当てます。

### IPアドレスプールの定義

割り当てるIPプールを定義します。IPレンジはKubernetesの内部IPではなく、ルータで払い出されているものを利用してください。

[yaml title="ip-address-pool.yaml"]
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb
spec:
  addresses:
    - 192.168.86.200-192.168.86.250
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb
spec:
  ipAddressPools:
  - default
[/yaml]

適用します
[bash]
kubectl apply -f ip-address-pool.yaml
[/bash]

### MetalLBのホスティング

[こちら](https://metallb.universe.tf/installation/) を参考に、以下のコマンドで、`strictARP`をtrueにします

[bash]
kubectl edit configmap -n kube-system kube-proxy
[/bash]

[yml title="helmfile.yaml"]
repositories:
  - name: metallb
    url: https://metallb.github.io/metallb

releases:
  - name: metallb
    namespace: metallb
    chart: metallb/metallb
    version: 0.14.5
    values:
      - values.yaml
[/yml]

実行自体はこけますが、repositoriesは追加されます。

[bash]
helmfile sync
[/bash]

以下のコマンドで変更可能箇所を出力します。出力しますが変更箇所はありません。
[bash]
helm show values metallb/metallb>values.yaml
[/bash]

適用します。
[bash]
helmfile sync
[/bash]

## PromethuesとGrafanaの設定

### はじめに

* PromethuesとGrafana間で通信させるために同じnamespaceに所属
* MetalLBを使ってGrafanaをクラスタ外へ公開

### 永続ストレージ設定

PersistentVolumeとPersistentVolumeClaimの設定をします。`prometheus-alertmanager`は、ストレージは作っていますが、後述のprometheusの設定でOFFにするのでなくても大丈夫です。

| ストレージタイプ      | name                       | path                                          |
|---------------------  |----------------------------|-----------------------------------------------|
| PersistentVolume      | prometheus-server          | /mnt/k8s/pv/prometheus-server                 |
| PersistentVolume      | prometheus-alertmanager    | /mnt/k8s/pv/prometheus-alertmanager           |
| PersistentVolume      | grafana                    | /mnt/k8s/pv/grafana                           |

| ストレージタイプ      | namespace     | name                       |
|---------------------  |---------------|----------------------------|
| PersistentVolumeClaim | monitoring    | prometheus-server          |
| PersistentVolumeClaim | monitoring    | prometheus-alertmanager    |
| PersistentVolumeClaim | monitoring    | grafana                    |

[bash title="monitoringネームスペース作成"]
kubectl create namespace monitoring
[/bash]

pvとpvcはcontrol-planeにデフォルトでは設定できないため、pi2側で設定します

[bash title="pi2で実行!!!"]
sudo mkdir -p /mnt/k8s/pv/grafana
sudo mkdir -p /mnt/k8s/pv/prometheus-server
sudo mkdir -p /mnt/k8s/pv/prometheus-alertmanager
[/bash]

pvとpvcを定義します
[yml title="volume.yaml"]
# grafana
kind: PersistentVolume
apiVersion: v1
metadata:
  name: grafana
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: grafana
  local:
    path: /mnt/k8s/pv/grafana
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - pi2
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: grafana
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: grafana
---

# prometheus-server
kind: PersistentVolume
apiVersion: v1
metadata:
  name: prometheus-server
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: prometheus-server
  local:
    path: /mnt/k8s/pv/prometheus-server
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - pi2
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: prometheus-server
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: prometheus-server
---

# prometheus-alertmanager
kind: PersistentVolume
apiVersion: v1
metadata:
  name: prometheus-alertmanager
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: prometheus-alertmanager
  local:
    path: /mnt/k8s/pv/prometheus-alertmanager
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - pi2
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: prometheus-alertmanager
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: prometheus-alertmanager
[/yml]

volumeを適用します
[bash]
kubectl apply -f volume.yaml
[/bash]

表通り作成されていることを確認できたらOKです
[bash]
kubectl get pv
kubectl get pvc -A
[/bash]

### Promethuesの設定

helmfileを書きます

[yml title="helmfile.yaml"]
repositories:
  - name: prometheus-community
    url: https://prometheus-community.github.io/helm-charts

releases:
  - name: prometheus
    namespace: monitoring
    chart: prometheus-community/prometheus
    version: 25.9.0
    values:
      - values.yaml
[/yml]

実行自体はこけますが、repositoriesは追加されます

[bash]
helmfile sync
[/bash]

以下のコマンドで変更可能箇所を出力します。出力しますが変更箇所はありません。
[bash]
helm show values prometheus-community/prometheus > values.yaml
[/bash]

以下のように修正します
[yml title="values.yaml"]
server:
  ## Prometheus server container name
  ##
  (中略)
  persistentVolume:
  (中略)
    existingClaim: "prometheus-server" # <- 先ほど作成したPVCを指定
  (中略)
    size: 10Gi # <- 8Giを10Giに変更
  (中略)
  service:
    (中略)
    type: NodePort # <- ClusterIPをNodePortへ変更
(中略)
alertmanager:
  ## If false, alertmanager will not be installed
  ##
  enabled: false # <- 今回は利用しないためtrueからfalseへ変更
[/yml]

適用します
[bash]
helmfile sync
[/bash]

### Grafanaの設定

同様にhelmfileを書きます

[yml title="helmfile.yaml"]
repositories:
  - name: grafana
    url: https://grafana.github.io/helm-charts

releases:
  - name: grafana
    namespace: monitoring
    chart: grafana/grafana
    version: 7.2.3
    values:
      - values.yaml
[/yml]

こちらも同様です。
[bash]
helmfile sync
[/bash]

以下のコマンドで変更可能箇所を出力します。
[bash]
helm show values grafana/grafana > values.yaml
[/bash]

以下のように修正します
[yaml title="values.yaml"]
service:
  enabled: true
  type: NodePort # <- ClusterIPをNodePortへ変更
(中略)
persistence:
  type: pvc
  enabled: true # <- ストレージ永続化ため、falseからtrueへ変更
  (中略)
  existingClaim: "grafana" # <- 先ほど作成したPVCを指定
(中略)
# データソースを追加
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.monitoring.svc.cluster.local/
      isDefault: true
[/yaml]

適用します
[bash]
helmfile sync
[/bash]

以下のコマンド実行して、マスターノードのクラスタ外から見れるプライベートIP:3000で、PCからアクセスしてGrafanaのログイン画面が見れれば成功です。
[bash title="マスターノード"]
kubectl port-forward --address 0.0.0.0 svc/grafana 3000:80 -n monitoring
[/bash]

ログインIDとパスワードは以下のコマンド出力できます

[bash]
kubectl get secret grafana --namespace monitoring  -o jsonpath="{.data.admin-user}" | base64 --decode ; echo
kubectl get secret grafana --namespace monitoring -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
[/bash]

### Grafanaをクラスタ外から閲覧する

MetalLBを利用して外部アクセス可能なLoadBalancerタイプのサービスを定義します。宛先にgrafanaを指定します。

[yaml title="grafana-loadbalancer.yaml"]
apiVersion: v1
kind: Service
metadata:
  name: grafana-loadbalancer
  namespace: monitoring
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 3000
      protocol: TCP
  selector:
    app.kubernetes.io/instance: grafana
    app.kubernetes.io/name: grafana
[/yaml]

適用します

[bash]
kubectl apply -f grafana-loadbalancer.yaml
[/bash]

`grafana-loadbalancer`に`192.168.86.200`が振られていることが確認できます。

[bash]
$ k get svc -A
NAMESPACE          NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                  AGE
calico-apiserver   calico-api                            ClusterIP      10.109.235.205   <none>           443/TCP                  5h2m
calico-system      calico-kube-controllers-metrics       ClusterIP      None             <none>           9094/TCP                 5h2m
calico-system      calico-typha                          ClusterIP      10.96.11.42      <none>           5473/TCP                 5h3m
default            kubernetes                            ClusterIP      10.96.0.1        <none>           443/TCP                  5h8m
kube-system        kube-dns                              ClusterIP      10.96.0.10       <none>           53/UDP,53/TCP,9153/TCP   5h8m
metallb            metallb-webhook-service               ClusterIP      10.98.53.232     <none>           443/TCP                  4h31m
monitoring         grafana                               NodePort       10.100.215.182   <none>           80:31851/TCP             4h35m
monitoring         grafana-loadbalancer                  LoadBalancer   10.98.112.253    192.168.86.200   80:30699/TCP             4h17m
monitoring         prometheus-kube-state-metrics         ClusterIP      10.110.198.118   <none>           8080/TCP                 4h34m
monitoring         prometheus-prometheus-node-exporter   ClusterIP      10.108.144.213   <none>           9100/TCP                 4h34m
monitoring         prometheus-prometheus-pushgateway     ClusterIP      10.107.27.149    <none>           9091/TCP                 4h34m
monitoring         prometheus-server                     NodePort       10.111.109.182   <none>           80:31648/TCP             4h34m
[/bash]

http://192.168.86.200にアクセスすると、http://192.168.86.200/loginにリダイレクトされて、grafanaのダッシュボードが閲覧できました


<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536800-640x352.png" alt="" width="640" height="352" class="alignnone size-medium wp-image-1262160" />


Data sourcesを開きます

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536797-640x322.png" alt="" width="640" height="322" class="alignnone size-medium wp-image-1262161" />


DatasourceはHelmのvalue.yamlで定義しているため、Prometheusがすでにある状態です。

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536786-640x327.png" alt="" width="640" height="327" class="alignnone size-medium wp-image-1262162" />

<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536792-640x324.png" alt="" width="640" height="324" class="alignnone size-medium wp-image-1262163" />


今回は、[こちら](https://grafana.com/grafana/dashboards/11074-node-exporter-for-prometheus-dashboard-en-v20201010/?tab=revisions)のダッシュボードを利用しますので、11074を入力してLoadします。
<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536789-640x493.png" alt="" width="640" height="493" class="alignnone size-medium wp-image-1262164" />


Loadされたら、importします
<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536785-640x400.png" alt="" width="640" height="400" class="alignnone size-medium wp-image-1262165" />


無事importできました。このときはまだ3つしかノードがないので、正常に動作しています!
<img src="https://devio2023-media.developers.io/wp-content/uploads/2024/05/1714536794-640x303.png" alt="" width="640" height="303" class="alignnone size-medium wp-image-1262166" />


## 最後に

先人が沢山いたのですが、かなり大変でした。環境を完全に消すのも大変だったので、都度M.2にイメージを焼き直してました。気づいたらUbuntu Server 24.04 LTSがリリースされていて手順を差し替えるか迷いました。23.10だと、クーラー周りの問題で音がかなり厳しいのでカーネルアップデートが必須でしたが、24.04は問題なかったので、結果待ち時間が少ない24.04の手順にしました。

大変だった点は主にネットーワーク周りになります。最初はp1をNAT化し、pi2以降はpi1をデフォルトゲートウェイとしてネットーワークに出ていたのですが、MetalLBを入れてから通信がループして設定を削除したりなど。。ここら辺もう少し再現確認してブログにできたらと思います。

各ワーカーノードのストレージとしてM.2を使いましたが、容量の大きいNFSサーバーを作ってそこに永続化リソースをまとめるの方が良いのかなとも思いました！

また弊社のハードに強い方にいくつかアドバイス頂き感謝です！

今後経過についてもブログ記事化できたらと思います！


## 付録

### トラブルシューティング
#### MetalLBで割り当てられたIPに接続できない

curlやブラウザ経由でMetalLBで割り当てられたIPでgrafanaの画面が閲覧できな場合です。以下のコマンドでARPリクエスト送信し、tcpdumpでARP応答を返していることを確認できれば、MetalLBとしては動作しています。FW設定などに問題あるかどうかを切り分けできます。

[bash]
# ARPリクエストを送信
arping -I eth0 192.168.86.100
# 応答していることを確認
sudo tcpdump -n -i eth0 arp src host 192.168.86.100
[/bash]

#### Grafanaダッシュボードからテストリクエストが通らない

corednsを再起動したら動くケースがありました。

[bash]
kubectl rollout restart deployment coredns -n kube-system
[/bash]

#### PrometheusやGrafanaのホスティングをやり直したい

永続化ストレージの消し忘れに注意

[bash]
helm uninstall prometheus -n monitoring
helm uninstall grafana -n monitoring

sudo rm -rf /mnt/k8s/pv/grafana
sudo rm -rf /mnt/k8s/pv/prometheus-server
sudo rm -rf /mnt/k8s/pv/prometheus-alertmanager
[/bash]

## 参考資料

* [Raspberry Pi 5でNVMeストレージ起動を試す](https://zenn.dev/usagi1975/articles/2023-12-29-000_raspi5_boot_nvme)
* [Raspberry Pi 5をM.2 NVMe SSDからブートしてベンチマークしてみた](https://zenn.dev/yuyakato/articles/82383b8c8869d3)
* [オンプレKubernetes環境にMetallbをインストールしてLoadbalanserServiceを利用できるようにする](https://zenn.dev/vampire_yuta/articles/ccbc57be8e092a)
* [minikubeでHelmを勉強してみた](https://zenn.dev/ryoyoshii/articles/f6409ede9ef84b)
* [ローカルKubernetesクラスター上でPrometheusのメトリクスをGrafanaダッシュボードで可視化する](https://zenn.dev/ring_belle/articles/prometheus-grafana-metrics)
* [我流お家クラウドを構築する【1】~kubernetesをセットアップする~](https://zenn.dev/murasame29/articles/ee5868123c1e33)
* [4台のラズパイでk8sクラスタを組んだ](https://skanehira.github.io/this-week-in-gorilla/articles/raspberry-pi-cluster.html)
* [TROUBLESHOOTING METALLB](https://metallb.universe.tf/troubleshooting/)

