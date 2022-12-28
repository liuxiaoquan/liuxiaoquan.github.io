# 免密登陆


``ssh-keygen``一路回车,主要是用来免密通信的

``ssh-copy-id ``172.16.24.220 需要输入对应主节的root密码

```shell
[root@chm log]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
7f:69:87:cf:28:fe:8b:19:55:a7:d0:c9:aa:6d:05:0c root@chm
The key's randomart image is:
+--[ RSA 2048]----+
|          E      |
|           o o . |
|            + = .|
|             = o |
|        S   o o  |
|         . + +   |
|          + B .  |
|          .B =   |
|         .+o+.o  |
+-----------------+
[root@chm log]# 
[root@chm log]# ssh-copy-id 172.16.24.220
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@172.16.24.220's password: 
 
Number of key(s) added: 1
 
Now try logging into the machine, with:   "ssh '172.16.24.220'"
and check to make sure that only the key(s) you wanted were added.
```


