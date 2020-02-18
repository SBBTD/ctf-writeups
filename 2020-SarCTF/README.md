# 2020-SarCTF

ctftime链接：https://ctftime.org/event/975/

比赛结果：![](https://img.shields.io/badge/%E5%89%8D-38%25-orange) 158 th / 418 teams 

## 0x01 Stego: Red King

题目：[m0r1ar7y.png](m0r1ar7y.png) 

描述：Just Moriarty? Really?

wp：Stegsolve，Extract Red / Column / LSB：

```
In 1989,  Moriarty murder ed .    Carl Pow ers, whom he cla imed .    "laugh ed at him". He k ept Powers' .     trainers almost  twenty years un til he .    used  the shoes as pa rt of his plan t o .    meet Sher lock Holmes and  had them planted  .    in the dis used flat at 221 C Baker Street..     From that po int onwards he d eveloped his .     skills at crim inal activities  and .    went on to create a larg e criminal .     organisation whi ch would stretch  .    across the  whole globe. fl ag is FLAG{who_i s_moriarty}
```

```
FLAG{who_is_moriarty}
```

## 0x02 Forensics: Confidential

原文件是抓FTP的一个pcap不放了，很简单可以提取出[KeePass文件](database.kdbx)，文件有密码保护。在这里了解了一个新工具`John The Ripper`，支持各种加密的爆破。[参考wp](https://spotless.tech/sarctf-Confidential.html)

1. 用John的实用工具转换kdbx文件为John支持的格式
2. 然后用字典爆破
3. 查看结果

```bash
$ keepass2john database.kdbx > crack
$ john --wordlist=rockyou.txt crack
$ john --show crack
```

好吧事实证明，John在Windows上实是难用，功能能用但是GPU加速的opencl对较新的显卡支持太差了，调了整整一下午只能做到用集显计算，也就比CPU快了一倍没啥意义。等以后装了Linux的双系统再议吧。

于是换成`hashcat`，对独显GPU兼容贼好，半分钟就完事。

将上面生成的crack文件裁剪一下，去掉前缀只保留这些：

```
$keepass$*2*60000*0*55b1d133aa811878a7573efbfc94107d035be43e56b22194e33a3982a98548b0*951d07f3d867d9edad2dcf60dab6995cac5246a1fa80841d06c7dccd8d7dd7b3*3e52f184d49556fb5a4052b102f6511a*b27cbff65e1a7ffefe76011a12f0b22b2e325fba30cb1980ce147cfc505023f6*ce7dfdb0f2abb3adb8b0dfd775e1e1e1ddf6bc9146e3582f57b930594b174205
```

然后运行：

```
hashcat64.exe -m 13400 -a 0 ..\crack ..\rockyou.txt
```

然后解出的这个密码还藏得不容易注意到：

```
$keepass$*2*60000*0*55b1d133aa811878a7573efbfc94107d035be43e56b22194e33a3982a98548b0*951d07f3d867d9edad2dcf60dab6995cac5246a1fa80841d06c7dccd8d7dd7b3*3e52f184d49556fb5a4052b102f6511a*b27cbff65e1a7ffefe76011a12f0b22b2e325fba30cb1980ce147cfc505023f6*ce7dfdb0f2abb3adb8b0dfd775e1e1e1ddf6bc9146e3582f57b930594b174205:blowme!
```

发现了吗，在最后面！就不能用文件名加编号的方式标注吗，找密码找得好苦。/(ㄒoㄒ)/~~

然后放到KeePass软件里密码`blowme!`一输，从里面存的密码里找一找flag就有了。

```
FLAG{bru73_p455w0rd_4ll_n16h7_l0n6}
```