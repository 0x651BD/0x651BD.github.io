---
layout: post
title:  "SECCON Beginners CTF 2018 Write-up"
date:   2018-06-14 07:30:00 +0900
categories: CTF
---
やや今更感がありますが，[SECCON Beginners CTF 2018](https://2017.seccon.jp/news/seccon-beginners-ctf-2018.html)に，チームとして参加しました．
結果は46位でした．

既にwrite-upが複数公開されていますが，その中の「Find the messages」について，私が解いた方法がまだ挙がっていないようなので，ここに記しておきます．

Find the messagesにて配布される7zファイルを解凍すると，disk.imgが展開されます．

```
$ file disk.img
file disk.img                                                                                                                                                                                                                                                  [15:03:07]
disk.img: DOS/MBR boot sector; partition 1 : ID=0x83, start-CHS (0x0,32,33), end-CHS (0x8,40,32), startsector 2048, 129024 sectors
```

この結果より，次のことが分かります．
* disk.imgがディスクイメージであること
* ブートセクタがついていること
* パーティション1の開始セクタが2048セクタであること

ここで1セクタは512バイトと仮定しても良いのですが，一応1セクタが何バイトかを調べてみます．

```
$ fdisk -l disk.img                                                                                                                                                                                                                                             [15:14:52]
Disk disk.img: 64 MiB, 67108864 bytes, 131072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xad4d4cf0

Device     Boot Start    End Sectors Size Id Type
disk.img1        2048 131071  129024  63M 83 Linux
```

1セクタが512バイトである事が分かりました．つまり，1つ目のパーティションは，`2048 * 512 = 1048576`バイトから始まるので，

```
sudo mount -o loop,offset=1048576 ./disk.img /mnt
```

でマウントすることができます．あとは，

```
$ cat /mnt/message1/message_1_of_3.txt| base64 -d
ctf4b{y0u_t0uched
```

`message2/message_2_of_3.png`は他のPNGヘッダと`XXXXX`で隠された部分とを比較してコピペしました．

`message3/message_3_0f_3.pdf`はファイルが削除されたのか見えません．
調べてみたところ，PDFファイルは先頭と最後が決まっているようなので，`file.img`からそれで囲まれる部分を取り出したらいけました．
