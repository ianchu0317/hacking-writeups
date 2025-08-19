---
layout: default
---


- room: https://tryhackme.com/room/lookup
- Nivel: Easy

Lo primero agregar `/etc/hosts` `10.201.87.192 lookup.thm`

## Enumeracion

Lo primero es correr para ver que puertos estan abiertos y versión

```bash
❯ nmap -A lookup.thm
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-14 16:06 -03
Nmap scan report for lookup.thm (10.201.87.192)
Host is up (0.35s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b1:45:15:99:4f:fc:ed:8d:29:d2:ac:e3:03:eb:23:b7 (RSA)
|   256 f9:e1:f7:71:b4:8d:3a:48:da:85:c8:6b:a5:f0:4b:18 (ECDSA)
|_  256 f2:46:8a:fd:11:d2:09:66:5e:08:d7:c4:f5:c3:46:a3 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Login Page
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.91 seconds
```

En gobuster no vemos nada interesante
```
❯ gobuster dir -u http://lookup.thm/ -w /usr/share/wordlists/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://lookup.thm/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 54154 / 220561 (24.55%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 54179 / 220561 (24.56%)
===============================================================
Finished
===============================================================
```

De hecho en la página web redirige a login.php como front. nano  No se si estará utilizando base de datos

Volviendo a la consigna, mencionaba hidden services y subdomains así que intento enumerar los subdominios:
```
❯ gobuster vhost -u http://lookup.thm -w /usr/share/wordlists/n0kovo_subdomains_medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://lookup.thm
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/wordlists/n0kovo_subdomains_medium.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: www.2 Status: 400 [Size: 321]
Found: 1 Status: 400 [Size: 321]
Found: 1111 Status: 400 [Size: 321]
Found: 2 Status: 400 [Size: 321]
Found: 2020 Status: 400 [Size: 321]
Found: 2016 Status: 400 [Size: 321]
Found: 3 Status: 400 [Size: 321]
Found: 2018 Status: 400 [Size: 321]
Found: 2022 Status: 400 [Size: 321]
Found: 360 Status: 400 [Size: 321]
Found: 99 Status: 400 [Size: 321]
Found: 2021 Status: 400 [Size: 321]
Found: 87399 Status: 400 [Size: 321]
Found: 8 Status: 400 [Size: 321]
Progress: 4406 / 500001 (0.88%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 4408 / 500001 (0.88%)
===============================================================
Finished
===============================================================
```

---

##Cosas que aprendi

- Ver lo que dice el servidor. LEER ES IMPORTANTE

- Enumeración de subdominios con vhosts gobuster `gobuster vhost -u http://dom.com -w <wordlist>`

- Utilización y configuración de BurpSuite con FoxyProxy

- Utilización de ffuf para fuerza bruta

- Utilizacion de hydra para fuerza bruta
