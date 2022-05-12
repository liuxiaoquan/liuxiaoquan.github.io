# Nethogs命令


```shell
wget https://github.com/raboof/nethogs/archive/v0.8.1.tar.gz
yum install libpcap-devel
tar zxvf v0.8.1.tar.gz
cd nethogs-0.8.1/
make && make install
nethogs eno1
```

![](/image-20210827152800063.png)

```以下是NetHogs的一些很有用的交互控制(键盘快捷键)```

```

    -m: Change the units displayed forthe bandwidth inunits like KB/sec->KB->B->MB.
    -r: Sort by magnitude of respectively traffic.
    -s: Sort by magnitude of sent traffic.
    -q: Hit quit tothe shell prompt.

```


