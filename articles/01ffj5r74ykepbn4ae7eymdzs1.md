---
title: "M5Stack Core2 for AWSを買ったので環境構築してみる"
type: "tech"
category: []
description: "M5Stack Core2 for AWSを動かすための環境を作ります"
publish: false
---

## はじめに
業務でATECC608Aを使う機会があり、もうちょっとCレイヤーに入り込めたら実装できたであろう出来事があったため、素振りのため買ってみました。

## 外観
**パッケージ**
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631630905/blog/01ffj5r74ykepbn4ae7eymdzs1/cj27lxejbzck136jm1pe.png)

**本体**
AWSカラーがかっこいい(!)
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631630943/blog/01ffj5r74ykepbn4ae7eymdzs1/mvi9vmfp3rf1aboep6mx.jpg)

**内容物**
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631630592/blog/01ffj5r74ykepbn4ae7eymdzs1/eukuk6karp6gdbjbwrkb.jpg)

## PlatformIOを使ってAWS IoT EduKitを動かしてみる

### はじめに
[AWS IoT EduKit](https://edukit.workshop.aws/jp/getting-started/prerequisites/macos.html)に従って環境構築を進める。

### Silicon Labs USB to UART bridgeのセットアップ
[Silicon Labs USB to UART bridgeのセットアップ](https://edukit.workshop.aws/jp/getting-started/prerequisites/macos.html#silicon-labs-usb-to-uart-bridgeのセットアップ)を参考にし、設定

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631634117/blog/01ffj5r74ykepbn4ae7eymdzs1/fouyzahpki1fiq3ih7cz.png)

インストーラーが起動します
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631634146/blog/01ffj5r74ykepbn4ae7eymdzs1/tegjhuhfxxlumbh0jalr.png)

### VSCodeの設定
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631634297/blog/01ffj5r74ykepbn4ae7eymdzs1/slubzp6bprh7oikhdmo8.png)

### ソースコードの取得
[ソースコードの取得](https://edukit.workshop.aws/jp/getting-started/prerequisites/macos.html#ソースコードの取得) の手順はやらない。
変わりに、[github.com/m5stack/Core2-for-AWS-IoT-EduKit](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git)を`ghq`を利用して取得。

```bash
$ ghq get https://github.com/m5stack/M5Core2.git
```

### スマートフォン向けアプリのダウンロードとインストール
[スマートフォン向けアプリのダウンロードとインストール](https://edukit.workshop.aws/jp/getting-started/prerequisites/windows.html#スマートフォン向けアプリのダウンロードとインストール)の通り実施

### USBポートの確認
[USBポートの確認](https://edukit.workshop.aws/jp/getting-started/prerequisites/macos.html#usbポートの確認)を参考に、`/dev/cu.SLAB_USBtoUART`をコピーします。

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631634924/blog/01ffj5r74ykepbn4ae7eymdzs1/rkyiaiiaohcwxl9h3rdm.png)

### PlatformIO でプロジェクトを開く
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631635129/blog/01ffj5r74ykepbn4ae7eymdzs1/veikahx4pl6ovy9rravc.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631635241/blog/01ffj5r74ykepbn4ae7eymdzs1/crjjj5tyeg8lwxmtdapq.png)

コピーした内容`github.com/m5stack/Core2-for-AWS-IoT-EduKit/Getting-Started/platformio.ini`を修正

```bash
-; upload_port =
+upload_port = /dev/cu.SLAB_USBtoUART
```

### ターミナルを開く
ターミナルを開きます。本手順でTerminalを開かないと`pio`コマンドが利用できない

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631635356/blog/01ffj5r74ykepbn4ae7eymdzs1/fcrjxvhgdakjgvqblyah.png)

### ファームのビルド/転送

ファームウェアのビルド
```bash
pio run --environment core2foraws
```

ビルドしたファームを転送
```bash
pio run --environment core2foraws --target upload
```

### シリアルの出力を監視

自分がM5Stackに対して行ったタッチ操作などイベントを監視可能

```bash
pio run --environment core2foraws --target monitor
```
ESP RainMaker スマートフォンアプリ用のQRコードが出ない場合、上記のコマンドを実行中の状態で、M5Stackを再起動させることで再出力することが可能


### ファームの削除

デバイスのファームウェアを消去
```bash
pio run --environment core2foraws --target erase
```

再起動するとディスプレイには何も映らない。なんかプチプチ変な音がする。。(これは私だけかもしれない..)




# (おまけ) Arduino.appで最初環境構築したメモ
::: details 詳細
### はじめに
[M5STACK公式で提供されているQUICK START](https://docs.m5stack.com/en/arduino/arduino_core2_development)に従って環境構築を進める。タブでCore2シリーズを選択。

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631624513/blog/01ffj5r74ykepbn4ae7eymdzs1/im5tw7gek5zs8kwke7ef.png)

### デバイスドライバーのインストール

Mac OSの場合、インストールする前に[システム環境設定]-> [セキュリティとプライバシー]-> [一般]を確認し、「AppStoreと確認済みの開発元からのアプリケーションを許可」になっていることを確認

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631624773/blog/01ffj5r74ykepbn4ae7eymdzs1/jxhdtslmfhuayogyibtz.png)

MacOSのデバイスドライバーを選択
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631624895/blog/01ffj5r74ykepbn4ae7eymdzs1/ffajbn0hczwowiolok97.png)

選択すると下記のzipファイルがダウンロードされます
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631624985/blog/01ffj5r74ykepbn4ae7eymdzs1/av3xzkzsm6g3gvihawug.png)

解凍するとdmgファイルになります
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625086/blog/01ffj5r74ykepbn4ae7eymdzs1/fw49nds6xcrhkfzdw3da.png)

pkgファイルをダブルクリックします
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625198/blog/01ffj5r74ykepbn4ae7eymdzs1/vywoxiqthondo02mztcc.png)

インストーラーが立ち上がるので、ポチポチしていきます
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625269/blog/01ffj5r74ykepbn4ae7eymdzs1/feb2dwjqqal6rm7taizt.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625335/blog/01ffj5r74ykepbn4ae7eymdzs1/gdpnjjibvenwvhp722wg.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625740/blog/01ffj5r74ykepbn4ae7eymdzs1/nhndt7pjooyfzbnnkrzj.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625744/blog/01ffj5r74ykepbn4ae7eymdzs1/c71z0yjxtj2n1q2hsosf.png)

インストール中にブロックされたので、セキュリティとプライバシーからインストールを許可設定をしました(Macのインストールあるあるですね)
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625749/blog/01ffj5r74ykepbn4ae7eymdzs1/ojhkqlpgol0jqnznxu5b.png)

インストール完了!
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631626429/blog/01ffj5r74ykepbn4ae7eymdzs1/m96hyxhymbscuwzehwhf.png)

### Arduino-IDEのインストール
M5STACK公式に書いてある通り、Arduino-IDEを引き続きインストールしていきます

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625978/blog/01ffj5r74ykepbn4ae7eymdzs1/isnqdgnh4eqzysyolqim.png)

172MBある
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631625988/blog/01ffj5r74ykepbn4ae7eymdzs1/rcsno2eqisnwptwhrchd.png)

解凍して、Arduino.appを立ち上げます
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631626238/blog/01ffj5r74ykepbn4ae7eymdzs1/ylemxmbojlju3zdz4pjy.png)


### M5Stack Boards Managerのインストール
`Preferences`を選択します
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631626525/blog/01ffj5r74ykepbn4ae7eymdzs1/qfdbjpcfxxmdxkzwwkxk.png)

追加のボードマネージャのURLに、`https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/arduino/package_m5stack_index.json`を追加
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631627059/blog/01ffj5r74ykepbn4ae7eymdzs1/lpxwp2semtqz4ahgzgzd.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631627066/blog/01ffj5r74ykepbn4ae7eymdzs1/xr22s1zex9zz19my7pfc.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631627105/blog/01ffj5r74ykepbn4ae7eymdzs1/gbsvw8oiiasoqib9xrru.png)

![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631627284/blog/01ffj5r74ykepbn4ae7eymdzs1/dxcdjmovfdnqcuwdizhw.png)

検索窓に`M5Stack`を入力し、インストールを押下
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631627503/blog/01ffj5r74ykepbn4ae7eymdzs1/vfs2galofvuvhjlfcxxq.png)

インストールが開始されると画像のように、進捗バーが表示される(初回インストールが始まらず、アプリを再起動して再度インストールを実施)
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631627873/blog/01ffj5r74ykepbn4ae7eymdzs1/vwyocosr4pb8nq1amd1z.png)

ボードに`M5Stack-Core2`を選択
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628031/blog/01ffj5r74ykepbn4ae7eymdzs1/fxcewj2qseecviavef2l.png)

ライブラリ管理を選択
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628146/blog/01ffj5r74ykepbn4ae7eymdzs1/grp1pcxlwus6nq1xxuao.png)

検索窓に`M5Core2`を入力し、インストールを押下
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628255/blog/01ffj5r74ykepbn4ae7eymdzs1/kcmu96od3iekegmogkew.png)

`install all`を選択
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628325/blog/01ffj5r74ykepbn4ae7eymdzs1/u2qluokcggknej34zzpu.png)

インストールが開始されます(初回no protcolとエラーメッセージが出力され失敗。アプリを再起動して再度インストールを実施)
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628373/blog/01ffj5r74ykepbn4ae7eymdzs1/uwqfyurtko1b7g1bkuy3.png)

### AWS IoTEduKit用のCore2用の追加ライブラリのインストール
M5STACK公式の手順でこちらがあるのは助かります。。

前の手順(M5Core2インストール)と同様に、ライブラリマネージャを開き、検索窓に`FastLED`と入力。インストールを押下
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628745/blog/01ffj5r74ykepbn4ae7eymdzs1/y21p4dx3hj3tcvbecech.png)

※ M5スタック公式にはない手順です。[M5Core2リポジトリ](https://github.com/m5stack/M5Core2)をcloneします。
```bash
$ ghq get https://github.com/m5stack/M5Core2.git
```

zipファイルを追加します
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631628961/blog/01ffj5r74ykepbn4ae7eymdzs1/l6biozmiyzaslwgganls.png)

手順通り、cloneしたリポジトリ[M5Core2/examples/core2_for_aws/](https://github.com/m5stack/M5Core2/tree/master/examples/core2_for_aws)配下に、`ArduinoECCX08.zip`があるので選択します
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631629364/blog/01ffj5r74ykepbn4ae7eymdzs1/hwahzaua3hv6tubmdrqo.png)

インストールしたライブラリを確認します
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631629589/blog/01ffj5r74ykepbn4ae7eymdzs1/kiypeduyiojb7vmwggqq.png)

インストール済みに絞ります
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631629649/blog/01ffj5r74ykepbn4ae7eymdzs1/ifiavagnpyxsuysyhx3q.png)

`ArduinoECCX08.zip`を追加したことで下記のライブラリが追加されたと推測(ECC608関連のライブラリはインストール済みのライブラリに対して検索すると、こちら以外にもあるため違う可能性あり)
![img](https://res.cloudinary.com/dkerzyk09/image/upload/v1631629706/blog/01ffj5r74ykepbn4ae7eymdzs1/efz2wvcal8z5rh5xlaf1.png)

:::
