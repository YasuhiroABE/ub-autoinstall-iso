
目的
====
これまで、TX120s7やThinkpad x220/x230をkittingするためのISOイメージを作成するための作業スペースを公開用に再編集しました。
元々はpreseedを利用していましたが、Ubuntu 20.04以降のAutoInstallのみに対応しています。

参考資料
========

22.04ではEFIブートイメージ(boot/grub/efi.img)がISOイメージに含まれなくなりました。
詳細と対応策については下記の文書を参照してください。

* https://askubuntu.com/questions/1403546/ubuntu-22-04-build-iso-both-mbr-and-efi

利用手順
=======

ISOイメージをダウンロードと初期ファイルの配置のため、次の作業は1回だけ実行します。

    $ make download
	$ make init

以下の作業は、ISOファイルを生成する度に実行する必要があります。
	
	$ make setup
	$ make geniso

新規にファイルを配置した場合には、適宜Makefileを編集してください。

user-dataファイル
----------------

導入作業のカスタマイズは、config/user-data を中心に行ないます。
主な追加設定の方法についてまとめます。

デフォルトユーザー・パスワード
----------------------------

user-data の username:、 password:行をそれぞれ希望に合わせて変更してください。

* ID: ubuntu
* Password: secret

password: 行に指定するハッシュ値は、``$ openssl passwd -6 -salt "$(openssl rand -hex 8)" secret`` コマンドで生成しています。

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

なお、APU/APU2はEFIブートに対応していないため、MBRブートをさせるため、config/user-data.mbr を config/user-data にコピーしてください。

この後に、``make setup && make geniso`` でISOイメージを作成してください。

以上
