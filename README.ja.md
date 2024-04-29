
目的
====
これまで、TX120s7やThinkpad x220/x230をkittingするためのISOイメージを作成するための作業スペースを公開用に再編集しました。

元々はpreseedを利用していましたが、Ubuntu 22.04以降のAutoInstallのみに対応しています。

履歴
====

* 2024/04/26
  * Ubuntu 24.04 Desktop版とServer版をサポートしました。
  * リポジトリ名を  ub-autoinstall-iso に変更しました。

参考資料
========

22.04ではEFIブートイメージ(boot/grub/efi.img)がISOイメージに含まれなくなりました。
詳細と対応策については下記の文書を参照してください。

* https://askubuntu.com/questions/1403546/ubuntu-22-04-build-iso-both-mbr-and-efi

利用手順
=======

必要なパッケージをインストールします。

    $ sudo apt update
    $ sudo apt install git make sudo

導入したいバージョンのタグをチェックアウトします。最新版を利用する場合は不要です。

    $ sudo git checkout refs/tags/22.04.4 -b my_22.04.4

ISOイメージをダウンロードと初期ファイルの配置のため、次の作業は1回だけ実行します。

    $ make download
    $ make init

以下の作業は、ISOファイルを生成する度に実行する必要があります。

    $ make setup
    $ make geniso

fdiskコマンドの出力がlocaleによって変化するため、LANG=Cの指定が安全です。
もし"C"以外のLANGを指定したい場合には、MakefileのGENISO_LANG値を変更してください。

config/user-dataファイル
----------------

導入作業のカスタマイズは、config/user-data を中心に行ないます。

* config/user-data.efi - UEFIで起動するためのESP領域を作成するための設定
* config/user-data.mbr - MBR(BIOS)起動をするための設定

必要なファイルを config/user-data に配置してください。

デフォルトでは、config/user-data.efi が config/user-data にリンクされています。
EFIをサポートしないシステムを使用されている場合には、config/user-data.mbr を使用してください。

主な追加設定の方法についてまとめます。

config/boot/grub/gurb.cfg file
------------------------------

シリアルのみでビデオ出力のないサーバーを利用する場合には、config/boot/grub/grub.cfgファイルで次の行を有効にしてください。

    linux	/casper/vmlinuz autoinstall "ds=nocloud-net;s=file:///cdrom/" quiet  --- console=ttyS0,115200n8

既存の行は削除するか以下のようにコメントアウトしてください。

    ## linux	/casper/vmlinuz autoinstall "ds=nocloud-net;s=file:///cdrom/" quiet ---

x230のようにテストしたシステムでは"console=ttyS0,115200n8"が有効でも正常に動作します。
しかしx270などの他のシステムでは正常に動作しない様子が確認されていますので、この設定には注意してください。

デフォルトユーザー・パスワード
----------------------------

user-data の username:、 password:行をそれぞれ希望に合わせて変更してください。

* ID: ubuntu
* Password: secret

password: 行に指定するハッシュ値は、以下のコマンドで生成しています。

    $ openssl passwd -6 -salt "$(openssl rand -hex 8)" secret

この他のカスタマイズ
===================

ssh鍵情報の登録
---------------

user-data の authorized-keys の空リストを削除し、.pubファイルの内容を列挙します。

    ssh:
      authorized-keys:
        - "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH8mvfUPhRddvGXBxGcvwo5m3CRVOf8RbFXwaUa9mhLX comment"
        - "..."

APU/APU2への対応
----------------

シリアル端末しかないデバイスにOSをインストールするため、次のような変更を加えています。

* config/isolinux/txt.cfg のappend行の最後に ``console=ttyS0,115200n8`` を加える
* config/boot/grub/grub.cfg のlinux行の最後に ``console=ttyS0,115200n8`` を加える
* isolinuxを利用したISOイメージファイルを作成するため、Makefileへのタスクの追加

APU/APU2にUSBメモリからインストールするためのISOイメージを作成するには以下のように isolinux, syslinux-common パッケージに含まれるファイルを利用してください。

以下の手順は最初の1回だけ行えば十分です。
繰り返し実行しても問題はありません。

    $ make download
    $ make init
    $ make setup-isolinux
    $ ln -fs user-data.mbr config/user-data
    $ sed -i -e 's/---$/--- console=ttyS0,115200n8/' config/boot/grub/grub.cfg

user-dataファイルを編集してから、ISOイメージを作成するには以下の手順を繰り返してください。

    $ make setup
    $ make geniso-isolinux

ライセンス
----------

    Copyright 2023,2024 Yasuhiro ABE, <yasu@yasundial.org, yasu-abe@u-aizu.ac.jp>

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

以上
