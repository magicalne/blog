title: tty、pty、ptmx、pts与container
layout: post
date: 2020-12-27
tag:
- ptmx
---

当我在实现linux container的时候，遇到的最大的问题（至今为止），就是如何让host与container通过命令行通信。所以深入学习了linux的tty。

所谓tty就是最早的tele-type输入设备，后来也叫pty。

现在的问题来了。我在linux上创建了一个container，这个container有自己的namespace，且会通过fork创建另外一个进程。这里把host进程称为pm，container进程称为ps。那么pm和ps之间如何在terminal中通信呢？pm是我们在terminal里启动的进程，可以通过stdio`看到`pm现在发生着什么。问题是ps是另外一个进程阿，ps是死是活，我怎么通过当前的terminal感知到呢？

所以需要重定向stdio，把ps的stderr、stdout都发送到pm的stdin，并把pm的stdio发送到ps的stdin。

以host的视角看container的目录结构：

```sh
# ls my_container/rootfs       
bin  dev  etc  home  notify  proc  root  sys  tmp  usr  var
```

但是以container的视角看自己当然是这样的：

```sh
# ls /       
bin  dev  etc  home  notify  proc  root  sys  tmp  usr  var
```

# [devpts](https://www.kernel.org/doc/html/latest/filesystems/devpts.html)

devpts历史众多，这里记录了我关注的部分。

在比较new的早期，devpts在linux下是一个单例，如果需要创建一个`新的`terminal，可以通过创建一个pts来实现，而devpts这个设备仍然是独一份。但是时间来到2008年左右，由于linux加入了container的概念，事情就很不一样了。如果devpts还是单例，那么会有安全问题：container1创建了一个pts1，container2可以通过devpts`看到`pts1。为了解决这个问题，devpts增加了`newinstance`这个参数，至此，container中的devpts才是单例。为了可以向后兼容（这个兼容我没看出有啥用），还需要:
```sh
 mount -o bind /dev/pts/ptmx /dev/ptmx
 ```

[patch详见这里](https://lwn.net/Articles/689539/)。

重点在这里：

>4. If multi-instance mode mount is needed for containers, but the system
   startup scripts have not yet been updated, container-startup scripts
   should bind mount /dev/ptmx to /dev/pts/ptmx to avoid breaking single-
   instance mounts.
\
   Or, in general, container-startup scripts should use:\
\
	mount -t devpts -o newinstance -o ptmxmode=0666 devpts /dev/pts\
	if [ ! -L /dev/ptmx ]; then\
		mount -o bind /dev/pts/ptmx /dev/ptmx\
	fi\
\
   When all devpts mounts are multi-instance, /dev/ptmx can permanently be
   a symlink to pts/ptmx and the bind mount can be ignored.

ptmx、pts是成对出现的 - [pseudoterminal master and slave](https://linux.die.net/man/4/ptmx)。

TODO
