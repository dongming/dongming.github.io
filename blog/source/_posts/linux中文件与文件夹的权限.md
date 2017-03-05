---
title: linux中文件与目录的权限
date: 2017-03-05 21:38:29
tags:
---
## linux中文件的权限

最近看了《鸟哥的linxu私房菜》其中涉及到linux中文件的权限问题，对于一个已经有过一定linux使用经验的人来说，也许知道了rwx三个分别代表了什么样的权限。在平时的使用中如果没有出问题，也许就不会去特地系统性的学习一下linux的文件和目录权限。很多时候都是google或者百度就是为了解决眼前的问题，不看书总结就不会有系统的知识。

### linux文件的权限
我的linux机器(ubuntu server)上的/etc目录下执行ll命令，显示如下的文件与目录

```
drwxr-xr-x  9 root root    4096 Feb 17 18:01 apparmor.d/
drwxr-xr-x  3 root root    4096 Feb 17 18:00 apport/
drwxr-xr-x  6 root root    4096 Feb 17 18:05 apt/
-rw-r-----  1 root daemon   144 Jan 14  2016 at.deny
-rw-r--r--  1 root root    2188 Aug 31  2015 bash.bashrc
-rw-r--r--  1 root root      45 Aug 12  2015 bash_completion
```

其中第一个字符为"d"的代表这是一个目录，"-"的代表是一个文件，这个一般使用过linux的都了解。第一个字符后跟着三组权限分别对应着user，group，others用户的权限。我们拿bash.bashrc文件为例，其权限为：

````
-rw-r--r--  1 root root    2188 Aug 31  2015 bash.bashrc
````

这个文件是属于root的，对应的用户组也是root。其对应权限"rw- r-- r--",说明root本身对其有rw的权限，而其同组的其他用户和其他非同组的用户对其只有可取权限。这个都是非常好理解的，其实文件的权限还是非常好理解的。

权限还可以用数字来表示，经常会听说600,755,777权限等，这个主要是指在权限表中r权限的值为4，w权限值为2，x权限值为1，而平常所说的755三个数字分别表示user，group，others的权限。那么7就是代表的这个用户对这个文件有rwx的权限，同组的其他用户对这个文件是rx的权限，其他用户对这个用户也是rx的权限。

当我作为一个others用户去修改一个我没有w权限的文件时会发生什么情况呢？我用mars用户去修改root用户的bash.bashrc文件

```
echo "hello world"
"bash.bashrc"
"bash.bashrc" E212: Can't open file for writing
Press ENTER or type command to continue

```
我在这个文件的最后增加了echo "hello world"这句话，当我执行保存时，会出现Can't open file for writing的错误，说明我们队这个文件是没有写的权限的。

对于文件的权限还是比较好理解的

### linux中目录的权限

上面的例子显示目录也是有对应的rwx的权限的，今天看书的时候才对目录的权限了解的这么细致。我就做了一下实验，来看看目录的权限分别表示什么

我使用root账户在/tmp目录下建立了一个目录act，并且在里面建立了一个properties的文件server.properties。并且设置这个目录对于others的用户只有r权限:

```
drwxr--r--  2 root root 4096 Mar  5 09:35 act/
```

root对这个目录有rwx的权限，而我一个others的mars用户对这个目录只有r的权限，我去访问以下这个目录试试

```
mars@ubuntu:/tmp$ ls act/
ls: cannot access 'act/server.properties': Permission denied
server.properties
mars@ubuntu:/tmp$ cd act/
-su: cd: act/: Permission denied
```

 对于只有r权限的目录，我只能看到这个目录下有哪些文件，对于cd操作是没有权限的，也就是我无法进入这个目录

 想要进入这个目录怎么办呢？加入x的权限，这里x的权限不是文件的执行权限，而是用户能够进入这个工作目录的(access)权限。

 ```
 drwxr-xr-x  2 root root 4096 Mar  5 09:35 act/
 ```
 现在目录的权限已经变更为755了，我的mars用户对这个目录具有的access的权限。看下面的操作

 ```
mars@ubuntu:/tmp$ cd act/
mars@ubuntu:/tmp/act$ ll
total 8
drwxr-xr-x 2 root root 4096 Mar  5 09:35 ./
drwxrwxrwt 9 root root 4096 Mar  5 09:50 ../
-rw-r--r-- 1 root root    0 Mar  5 09:35 server.properties
 ```

现在mars用户可以执行cd命令，将act的目录作为当前的工作目录了。

现在删除一下这个server.properties试试

```
mars@ubuntu:/tmp/act$ rm server.properties 
rm: remove write-protected regular empty file 'server.properties'? y
rm: cannot remove 'server.properties': Permission denied
```
我们能够读取act目录中内容，能够进入act目录作为当前的工作目录，但是我们无法删除该目录下的文件是因为我们对于这个目录没有w权限
w权限主要是修改这个目录结构的权限

* 在目录下新建文件或者目录
* 删除目录下的文件
* 对目录下的文件重命名
* 移动目录下的文件

所以这个权限对于目录来说最重要，默认的目录建立的时候是755，也就是说others用户可以访问和读取文件夹的信息，但是不能修改文件夹的结构。对于一些重要的目录这些权限保证了和这个目录的安全，是非常实用的配置。

### 总结
linux中对于文件和目录的权限是有区别的
rwx对于文件代表着可读，可写和可执行，但是对于目录而言则分别表示可以读取目录的结构信息，可能更改目录的结构和能将这个目录作为当前工作目录。