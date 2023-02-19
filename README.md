---
description: A small misc challenge from CakeCTF 2022
---

# 🎂 README 2022

```
nc misc.2022.cakectf.com 12022
readme2022_80ade97026adcb7e3e8f6203ad1eab06.tar.gz
```

The challenge give us one compressed file and server to connect.

I decided to decompressed it then I found Dockerfile and server.py file.

![Dockerfile](https://user-images.githubusercontent.com/89141562/188305566-df9bfcdb-5123-476f-8c89-cf63b9e3af1a.png)

So when we connect to the remote server, it will run server.py, let's see what inside.

{% code lineNumbers="true" %}
```python
import os

try:
    f = open("/flag.txt", "r")
except:
    print("[-] Flag not found. If this message shows up")
    print("    on the remote server, please report to amdin.")

if __name__ == '__main__':
    filepath = input("filepath: ")
    if filepath.startswith("/"):
        exit("[-] Filepath must not start with '/'")
    elif '..' in filepath:
        exit("[-] Filepath must not contain '..'")

    filepath = os.path.expanduser(filepath)
    try:
        print(open(filepath, "r").read())
    except:
        exit("[-] Could not open file")
```
{% endcode %}

As you can see, we must input the filepath to the remote server in order to get the flag, we can't use <mark style="color:red;">`/`</mark> or <mark style="color:red;">`..`</mark> symbol either. But I found something interesting: <mark style="color:yellow;">`os.path.expanduser(`</mark><mark style="color:green;"><mark style="color:yellow;">`filepath`<mark style="color:yellow;"></mark><mark style="color:yellow;">`)`</mark>

"If you pass something like `~xxx/path/to/file`, \~xxx is expanded to the path of xxx's home directory. If you pass something like `~xxx/path/to/file`, \~xxx is expanded to the path of xxx's home directory." - Thanks to _**ptr-yudai**_. So I decided to look up passwd file by building up docker.

![Docker Environment](https://user-images.githubusercontent.com/89141562/188305576-fe067ec1-fd1f-4294-a8eb-0cd2075502d1.png)

We will notice there's a line `sys:x:3:3:sys:/dev:/usr/sbin/nologin`. So we have the access to all file under /dev. Using `ls -la /dev`, I found this `lrwxrwxrwx 1 root root 13 Sep 4 08:09 fd -> /proc/self/fd`. So we will use file descriptor to get the flag.

There's also a check function in server.py

```python
try:
    f = open("/flag.txt", "r")
except:
    print("[-] Flag not found. If this message shows up")
    print("    on the remote server, please report to amdin.")
```

When I connect to remote server, nothing went wrong. As a result, we can come to the following conclusion: the flag file is opening with a file descriptor. The `/dev/fd` has a symbolic link to `/proc/self/fd`. All we had to do was using `~sys/fd/[x]`. You can try "x" from 3, because "on a Unix-like operating system, the first three file descriptors, by default, are STDIN (standard input), STDOUT (standard output), and STDERR (standard error)" which is "0", "1" and "2". (You can read it from [here](https://www.computerhope.com/jargon/f/file-descriptor.htm))

Here is the result:

![](https://user-images.githubusercontent.com/89141562/188305587-de4f6df2-e9ae-4a44-b606-64ceffbbc894.png)

<mark style="color:green;">**FLAG: CakeCTF{\~USER\_r3f3rs\_2\_h0m3\_d1r3ct0ry\_0f\_USER}**</mark>
