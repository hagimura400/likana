likana
======

[![Build Status](https://travis-ci.org/maijou2501/likana.svg?branch=master)](https://travis-ci.org/maijou2501/likana)

## 概要

MIZUSHIKI 様が作成された IMEオン忘れ時打ち直しツール「りかなー」とほぼ同仕様の Linux 版りかなー。


## 使い方

※ 本家「りかなー」の使い方から一部変更し引用 (2連打の箇所)

IME(インプットメソッドエンジン)オンを忘れてタイプしてしまったらすかさず「半角/全角」キーを2連打。直前の文字を打ち直しします。


## 動作例

1. IMEオフ時に、キーボード入力「utinaositesuto」
2. "半角/全角キー" を2連打
3. IMEオンになり、「うちなおしてすと」で変換候補が出る

![ likana.gif ](https://github.com/maijou2501/maijou2501.github.io/blob/master/image/likana.gif "likana.gif")

## 動作要件

サービスの管理には "SysVinit", "systemd" を利用しています。

動作確認は、Ubuntu 12.04 (SysVinit)、Ubuntu 16.04 (systemd) で行いました。


## インストール と アンインストール

### PPA に公開されたパッケージからインストール・アンインストール

#### インストール

```sh
sudo apt-add-repository -y ppa:maijou2501/likana
sudo apt update
sudo apt -y install likana
```

#### アンインストール

```sh
sudo apt -y remove likana
```


### make してインストール・アンインストール

#### インストール

`/usr/local/src` など、`make` を実行するディレクトリに移動して作業を行う。

```sh
git clone https://github.com/maijou2501/likana
cd likana/src
mv Makefile.bak Makefile
make && sudo make install
```

※ インストール先に書き込み権限があれば、`sudo` は必要ありません。(これ以降の手順でも同様です。)

#### アンインストール

```sh
sudo make uninstall
```


### ./configure してインストール・アンインストール

#### インストール 

`/usr/local/src` など、`./configure` を実行するディレクトリに移動して作業を行う。

```sh
git clone https://github.com/maijou2501/likana
cd likana
./configure && make && sudo make install
```

デフォルトでは `/usr/bin` に実行ファイルが配置されますが、任意の場所にインストールする場合は `./configure --prefix=/target/dir` を実行してください。


#### アンインストール

```sh
sudo make uninstall
```


## 設定

### OS起動時のサービス実行

#### SysVinit

`/etc/default/likana` で起動時のサービス実行設定を行なった後、`service likana start` を実行してください。(`insserv` が実行されます)


#### systemd

```sh
sudo systemctl enable likana.service
```


### キーボードとマウスの設定

`/etc/default/likana` にて設定を行います。

キーボードとマウスの `/dev/input/eventX` 割り当ては下記コマンドで調べたり、起動毎に割り当てが変わる場合は `udev` のルールで固定化する必要があるかもしれません。

```sh
$ ls -l /dev/input/by-id
lrwxrwxrwx 1 root root   9  1月  4 22:24 usb-Lite-On_Technology_Corp._ThinkPad_USB_Keyboard_with_TrackPoint-event-kbd -> ../event3
lrwxrwxrwx 1 root root   9  1月  4 22:24 usb-Lite-On_Technology_Corp._ThinkPad_USB_Keyboard_with_TrackPoint-if01-event-mouse -> ../event4
```

上記の場合は、`/etc/default/likana` に下記のように設定します。

```
LIKANA_KEYBOARD=/dev/input/event3
LIKANA_MOUSE=/dev/input/event4
```


または `/dev/input/eventX` ではなく、下記指定も可能です。(キーボードとマウスのキャラクタデバイスのチェックを、`lstat()` ではなく `stat()` でチェックしているため。)

```
LIKANA_KEYBOARD=/dev/input/by-id/usb-Lite-On_Technology_Corp._ThinkPad_USB_Keyboard_with_TrackPoint-event-kbd
LIKANA_MOUSE=/dev/input/by-id/usb-Lite-On_Technology_Corp._ThinkPad_USB_Keyboard_with_TrackPoint-if01-event-mouse
```


#### systemd

スマートでは無いですが、キーボードだけを設定する場合は、下記の変数代入のコメントアウト場所を入れ替えてください。

```
# For systemd
# Select below
LIKANA_ARGS="-k /dev/input/event2" 
#LIKANA_ARGS="-k /dev/input/event2 -m /dev/input/event3"
```


### キーボードの接続・取り外しを udev にて検知する(推奨設定)

`/lib/udev/rules.d/60-likana.rules` を `/etc/udev/rules.d` にコピーし、利用中のキーボード情報を登録することでキーボードの取り外しを検知して自動で `likana` サービスを停止・起動することが可能となります。

```sh
$ lsusb
Bus 002 Device 003: ID 17ef:6009 Lenovo ThinkPad Keyboard with TrackPoint
```

上記例では `idVendor=17ef`、`idProduct=6009` をルールに設定します。(コマンドは `service`、`systemctl` を適宜選択ください。)

__この設定を行わない場合__、`likana` サービスはキーボードが取り外されても実行したままとなり、更にキーボードのキャラクタデバイスのファイルオープンが切れるため、キーボードを繋ぎ直しても打ち直しが実行できなくなります。

また、systemd の場合はキーボードを繋ぎ直した際に `likana` サービスを自動起動できるようにルール設定の `ACTION=="add"` を有効にしてください。(SysVinit でこの add 設定を行うとサービスの多重起動が問題となったため、SysVinit の接続検知は見送ることと致しました。)


## プログラムの仕様

* 設定された1つのキーボードの入力をキャプチャします。（マウスも、接続された1つのマウスのみイベントをキャプチャできます）
* "アルファベット" or "-" が入力された場合にキーロギングを行います。
* "アルファベット" or "-" 入力以外と、左クリックが行われた場合は、ロギングしたキー内容をクリアします。
* "半角/全角"キーが2回押された場合には、 "半角/全角"キー入力をソフトウェアで行いIMEオンにし、ロギングしたキー内容を再生します。
* キーロギングできる文字数は60文字で、それを超えるとロギングしたキー内容をクリアします。

[ likana: メインページ ]( http://maijou2501.github.io/likana/index.html )


## 免責

作者は、本ソフトウェアの使用または使用不能から生じるコンピュータの故障、情報の消失、その他あらゆる直接的及び間接的被害に関して一切の責任を負いません。

これに同意できない場合、本ソフトウェアの使用を禁止します。


## ライセンス

GPLv3 で配布しています。ソースコードを変更して再配布など自由にご利用ください。


## 更新履歴

* version 1.5 (2016/10/28 ) systemd 対応を行った
* version 1.4 (2016/09/13 ) キーボードの取り外しを `udev` で検知し、`likana stop` の実行を行えるようにした
* version 1.3 (2016/01/02 ) 引数のチェックを改良。セキュリティに配慮したコンパイル設定に変更した
* version 1.2 (2015/03/03 ) initスクリプト登録を、可能であれば `insserv` を用いるようにした
* version 1.1 (2015/03/01 ) キー再生についてパラメータ調整を行った
* version 1.0 (2015/02/28 ) 公開


## Special thanks

"りかなー" 作成者である MIZUSHIKI 様には、似たソフト名にすることに加え公開についての許可、仕様についてのご助言をいただきました。

"りかなー" を使わせていただいていることと、本ソフト作成の経緯について感謝を申し上げます。
