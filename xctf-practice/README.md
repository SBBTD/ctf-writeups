# xctf

这个是国内做的挺不错的网站，经常在上面刷题，选择它很多还是因为简洁、难度分级合理，而且国外那些知名ctfoj访问确实不是很方便。

https://adworld.xctf.org.cn/task

## 0x001.Reverse: Akira_Homework

题目：[WinRev.exe](WinRev.exe)

笔记：这个题涉及面很广，用了TLS、文件流、多线程、“另类”的加载库等，对新手学习帮助很大，值得仔细研磨。

附件：提取出来的dll文件：[WinRevdumped.dll](WinRevdumped.dll)

## 0x002.Stego: 简单流量分析

// 2019工业信息安全技能大赛个人线上赛第二场 [fetus_pcap.pcap](fetus_pcap.pcap)

这个题全是ICMP数据包，每一段里都有数据，而且和一般的ICMP不一样，全是plaintext，于是最先想到的就是导出来看看。使用`tshark`导出：

```bash
$ tshark -r fetus_pcap.pcap -e data.data -T fields -Y "ip.src == 192.168.3.73" > a.txt
```

对数据格式进行处理，结果很像是base64，但是解码后全是乱码。

换一个思路，这个题和正常的ICMP再一对比，发现数据长度有异常，正常应该长度不变，而且长度数值也都小于128，于是导出数据部分的长度，作为ASCII码：

```bash
$ tshark -r fetus_pcap.pcap -e data.len -T fields -Y "ip.src == 192.168.3.73"
```

然后脚本解出来再解base64得到flag。

```
FLAG{xx2b8a_6mm64c_fsociety}
```

## 0x003.Misc: 恶臭的数据包

cap数据包文件[cacosmia.7z](cacosmia.7z)

这个用wireshark一打开全是没有标注的无线数据包，一开始以为是正常的包，搜索后得知这其实是因为没有对加密的无线数据包进行解密，要解密必须获知对应网络的密码。

于是用aircrack-ng查看有一个WPA握手包，破解出来密码是12345678，然后在wireshark的“首选项-协议-IEEE802.11”设置网络的密码，格式为`pwd:SSID`这样数据包就是解密的了。

另外，WLAN中认证协议为EAPOL，可以在wireshark里filter EAPOL来查看是否有共四步的握手包。

分析数据得到一个PNG，隐藏一个zip，有密码，不是伪加密，密码hint是藏在JWT里的，很多时候可能就错过了，要注意检查这些细节。hint里面说密码是刚刚ping过的网站，但是并没有icmp(or v6)数据包，这个是想让找dns包，ping没有抓到icmp就得考虑下dns的可能。

## 0x004.