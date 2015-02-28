likana
======

## 概要

MIZUSHIKI 様が作成された IMEオン忘れ時打ち直しツール「りかなー」と  
ほぼ同仕様の Linux 版りかなー


## 使い方

※ 本家「りかなー」の使い方から一部変更し引用 (2連打の箇所 )

IMEオンを忘れてタイプしてしまったらすかさず「半角/全角」キーを2連打。  
直前の文字を打ち直しします。  


## 動作例

1. IMEオフ時に、キーボード入力「utinaositesuto」
2. "半角/全角キー" を2連打
3. IMEオンになり、「うちなおしてすと」で変換候補が出る


## 動作要件

サービスの管理には "system V" の init を利用しています。  
また本デーモンの起動は root 権限で実行されます。

※ 動作確認は ubuntu 12.04 で行いました。
※ 今後 ubuntu の LTS のバージョンで systemd が採用された場合には、systemd 対応を行う予定です。


## インストール と アンインストール

### PPA に公開されたパッケージからインストール・アンインストール

インストール

```shell
sudo apt-add-repository ppa:maijou2501/likana
sudo apt-get update
sudo apt-get install likana
```

アンインストール

```shell
sudo apt-get purge likana
```


### make してインストール・アンインストール

インストール ( /usr/src など、make を実行するディレクトリに移動してから )

```shell
git clone https://github.com/maijou2501/likana
cd likana
make
sudo make install
```

アンインストール

```shell
sudo make uninstall
```


## 設定

/etc/default/likana で起動時のサービス実行の有無、  
キーボードとマウスの設定等を行なってください。

キーボードとマウスの /dev/input/event* 割り当ては  
下記コマンドで調べたり、起動毎に割り当てが変わる場合は  
udev ルールで固定化する必要があるかもしれません。

```shell
ll /dev/input/by-id
ll /dev/input/by-path
```


## 免責

作者は、本ソフトウェアの使用または使用不能から生じるコンピュータの故障、情報の  
消失、その他あらゆる直接的及び間接的被害に関して一切の責任を負いません。

これに同意できない場合、本ソフトウェアの使用を禁止します。


## ライセンス

GPLv3 で配布しています。  
ソースコードを変更して再配布など自由にご利用ください。


## 更新履歴
￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣￣
* version 1.0 公開 (2015/02/28 )


## Special thanks

"りかなー" 作成者である MIZUSHIKI 様には、似たソフト名にすることに加え  
公開についての許可、仕様についてのご助言をいただきました。  
"りかなー" を使わせていただいていることと、本ソフト作成の経緯について  
感謝を申し上げます。