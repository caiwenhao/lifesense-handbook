# Linux 命令神器：lsof 入门

**lsof**是系统管理/安全的尤伯工具。我大多数时候用它来从系统获得与网络连接相关的信息，但那只是这个强大而又鲜为人知的应用的第一步。将这个工具称之为lsof真实名副其实，因为它是指“**列出打开文件（lists openfiles）**”。而有一点要切记，在Unix中一切（包括网络套接口）都是文件。

有趣的是，lsof也是有着最多开关的[Linux](http://www.ttlsa.com/linux/)/Unix命令之一。它有那么多的开关，它有许多选项支持使用-和+前缀。

正如你所见，lsof有着实在是令人惊讶的选项数量。你可以使用它来获得你系统上设备的信息，你能通过它了解到指定的用户在指定的地点正在碰什么东西，或者甚至是一个进程正在使用什么文件或网络连接。

对于我，lsof替代了netstat和ps的全部工作。它可以带来那些工具所能带来的一切，而且要比那些工具多得多。那么，让我们来看看它的一些基本能力吧：

### 关键选项

理解一些关于lsof如何工作的关键性东西是很重要的。最重要的是，当你给它传递选项时，**默认行为**是对结果进行“或”运算。因此，如果你正是用-i来拉出一个端口列表，同时又用-p来拉出一个进程列表，那么默认情况下你会获得两者的结果。

下面的一些其它东西需要牢记：

* **默认**
  : 没有选项，lsof列出活跃进程的所有打开文件
* **组合**
  : 可以将选项组合到一起，如-abc，但要当心哪些选项需要参数
* **-a**
  : 结果进行“与”运算（而不是“或”）
* **-l**
  : 在输出显示用户ID而不是用户名
* **-h**
  : 获得帮助
* **-t**
  : 仅获取进程ID
* **-U**
  : 获取UNIX套接口地址
* **-F**
  : 格式化输出结果，用于其它命令。可以通过多种方式格式化，如-F pcfn（用于进程id、命令名、文件描述符、文件名，并以空终止）

### 获取网络信息

正如我所说的，我主要将lsof用于获取关于系统怎么和网络交互的信息。这里提供了关于此信息的一些主题：

**使用-i显示所有连接**  
有些人喜欢用netstat来获取网络连接，但是我更喜欢使用lsof来进行此项工作。结果以对我来说很直观的方式呈现，我仅仅只需改变我的语法，就可以通过同样的命令来获取更多信息。

```bash
# lsof -i
COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME
dhcpcd 6061 root 4u IPv4 4510 UDP *:bootpc
sshd 7703 root 3u IPv6 6499 TCP *:ssh (LISTEN)
sshd 7892 root 3u IPv6 6757 TCP 10.10.1.5:ssh->192.168.1.5:49901 (ESTABLISHED)
```

**使用-i 6仅获取IPv6流量**

```bash
lsof -i 6
```

**仅显示TCP连接（同理可获得UDP连接）**  
你也可以通过在-i后提供对应的协议来仅仅显示TCP或者UDP连接信息。

```bash
# lsof -iTCP
COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME
sshd 7703 root 3u IPv6 6499 TCP *:ssh (LISTEN)
sshd 7892 root 3u IPv6 6757 TCP 10.10.1.5:ssh->192.168.1.5:49901 (ESTABLISHED)
```

**使用-i:port来显示与指定端口相关的网络信息**

或者，你也可以通过端口搜索，这对于要找出什么阻止了另外一个应用绑定到指定端口实在是太棒了。

```bash
# lsof -i :22
COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME
sshd 7703 root 3u IPv6 6499 TCP *:ssh (LISTEN)
sshd 7892 root 3u IPv6 6757 TCP 10.10.1.5:ssh->192.168.1.5:49901 (ESTABLISHED)
```

**使用@host来显示指定到指定主机的连接**  
这对于你在检查是否开放连接到网络中或互联网上某个指定主机的连接时十分有用。

```bash
# lsof -i@172.16.12.5
sshd 7892 root 3u IPv6 6757 TCP 10.10.1.5:ssh->172.16.12.5:49901 (ESTABLISHED)
```

**使用@host:port显示基于主机与端口的连接**

你也可以组合主机与端口的显示信息。

```bash
# lsof -i@172.16.12.5:22
sshd 7892 root 3u IPv6 6757 TCP 10.10.1.5:ssh->172.16.12.5:49901 (ESTABLISHED)
```

**找出监听端口**

找出正等候连接的端口。

```bash
# lsof -i -sTCP:LISTEN
```

你也可以grep “LISTEN”来完成该任务。

```bash
# lsof -i | grep -i LISTEN
iTunes 400 daniel 16u IPv4 0x4575228 0t0 TCP *:daap (LISTEN)
```

**找出已建立的连接**

你也可以显示任何已经连接的连接。

```bash
# lsof -i -sTCP:ESTABLISHED
```

你也可以通过grep搜索“ESTABLISHED”来完成该任务。

```bash
# lsof -i | grep -i ESTABLISHED
firefox-b 169 daniel 49u IPv4 0t0 TCP 1.2.3.3:1863->1.2.3.4:http (ESTABLISHED)
```



