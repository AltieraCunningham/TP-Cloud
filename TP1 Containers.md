## 1. Install

### Setup docker apt repo
``` bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### Install the lastest version
```console
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Starting and checking docker service
```console
alt@TP1-YMCA:~$ sudo systemctl start docker  
alt@TP1-YMCA:~$ sudo systemctl status docker  
● docker.service - Docker Application Container Engine  
    Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)  
    Active: active (running) since Mon 2025-03-31 08:48:05 UTC; 41s ago
```

### Add user to docker group

```console
alt@TP1-YMCA:~$ sudo usermod -aG docker $(whoami)
alt@TP1-YMCA:~$ id  
uid=1000(alt) gid=1000(alt) groups=1000(alt),4(adm),24(cdrom),27(sudo),30(dip),105(lxd),988(docker)
```

## 1. Lancement de conteneurs
### docker run
```console 
docker run -it -d -p9999:80 nginx
```

```console 
alt@TP1-YMCA:~$ curl 127.0.0.1:9999  
<!DOCTYPE html>  
<html>  
<head>  
<title>Welcome to nginx!</title>  
<style>  
html { color-scheme: light dark; }  
body { width: 35em; margin: 0 auto;  
font-family: Tahoma, Verdana, Arial, sans-serif; }  
</style>  
</head>  
<body>  
<h1>Welcome to nginx!</h1>  
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p>  
  
<p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at  
<a href="http://nginx.com/">nginx.com</a>.</p>  
  
<p><em>Thank you for using nginx.</em></p>  
</body>  
</html>
```

### Open the service to Internet
```console 
┌──(alt㉿Altiera)-[~]  
└─$ curl http://51.137.90.104:9999                                                                   
<!DOCTYPE html>  
<html>  
<head>  
<title>Welcome to nginx!</title>  
<style>  
html { color-scheme: light dark; }  
body { width: 35em; margin: 0 auto;  
font-family: Tahoma, Verdana, Arial, sans-serif; }  
</style>  
</head>  
<body>  
<h1>Welcome to nginx!</h1>  
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p>  
  
<p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at  
<a href="http://nginx.com/">nginx.com</a>.</p>  
  
<p><em>Thank you for using nginx.</em></p>  
</body>  
</html>
```

### Container customizartion

```console
docker run -it --name meow -v /home/alt/conf/:/etc/nginx/conf.d/ -v /home/alt/html:/var/www/tp_docker -p7777:7777 -m512M -d nginx
```

## Part II : Images

### Dockerfile 

```Dockerfile 
FROM alpine  
  
RUN apk update  
  
RUN apk add apache2  
  
COPY html/index.html /var/www/localhost/htdocs  
  
COPY html/ура-простоквашино.gif /var/www/localhost/htdocs  
  
CMD [ "/usr/sbin/httpd", "-DFOREGROUND"]
```

## Part III : `docker-compose`

### 2. WikiJS
```docker-compose
services:  
  
 db:  
   image: postgres:15-alpine  
   environment:  
     POSTGRES_DB: wiki  
     POSTGRES_PASSWORD: wikijsrocks  
     POSTGRES_USER: wikijs  
   logging:  
     driver: none  
   restart: unless-stopped  
   volumes:  
     - db-data:/var/lib/postgresql/data  
  
 wiki:  
   image: ghcr.io/requarks/wiki:2  
   depends_on:  
     - db  
   environment:  
     DB_TYPE: postgres  
     DB_HOST: db  
     DB_PORT: 5432  
     DB_USER: wikijs  
     DB_PASS: wikijsrocks  
     DB_NAME: wiki  
   restart: unless-stopped  
   ports:  
     - "80:3000"  
  
volumes:  
 db-data:
```


### 3. Meow

```Dockerfile
FROM alpine  
  
COPY python-app /app  
  
ENV PYTHONUNBUFFERED=1  
RUN apk add --update --no-cache python3 py3-pip  
RUN pip3 install -r /app/requirements.txt --break-system-packages  
  
CMD ["python3","/app/app.py"]
```

```docker-compose
#app  
services:  
 meowapp:  
   image: meowapp:latest  
   restart: always  
   ports:  
     - '8888:8888'  
  
#redis  
 cache:  
   image: redis:6.2-alpine  
   restart: always  
   ports:  
     - '6379:6379'  
   hostname: db  
   command: redis-server --save 20 1 --loglevel warning    
   volumes:    
     - cache:/data  
volumes:  
 cache:  
   driver: local
```


## Part IV : Docker security

### Le groupe docker
```console 
alt@TP1-YMCA:~$ sudo su  
root@TP1-YMCA:/home/alt# cd /root  
root@TP1-YMCA:~# echo "This is a secret file" > root.txt  
root@TP1-YMCA:~# exit  
exit  
alt@TP1-YMCA:~$ whoami  
alt  
alt@TP1-YMCA:~$ id  
uid=1000(alt) gid=1000(alt) groups=1000(alt),4(adm),24(cdrom),27(sudo),30(dip),105(lxd),988(docker)  
alt@TP1-YMCA:~$ docker -H unix:///run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash  
root@ccd05538b617:/# cat /root/root.txt  
This is a secret file  
root@ccd05538b617:/# whoami  
root
```

### 2. Scan de vuln
#### WikiJS
```console 
alt@TP1-YMCA:~$ trivy image ghcr.io/requarks/wiki:2 | tail -n20  
2025-03-31T14:58:51+02:00       INFO    [vulndb] Need to update DB  
2025-03-31T14:58:51+02:00       INFO    [vulndb] Downloading vulnerability DB...  
2025-03-31T14:58:51+02:00       INFO    [vulndb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-db:2"  
61.66 MiB / 61.66 MiB [-----------------------------------------------------------------------------------------------------------------------------------------] 100.00% 8.62 MiB p/s 7.4s  
2025-03-31T14:58:58+02:00       INFO    [vulndb] Artifact successfully downloaded       repo="mirror.gcr.io/aquasec/trivy-db:2"  
2025-03-31T14:58:58+02:00       INFO    [vuln] Vulnerability scanning is enabled  
2025-03-31T14:58:58+02:00       INFO    [secret] Secret scanning is enabled  
2025-03-31T14:58:58+02:00       INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning  
2025-03-31T14:58:58+02:00       INFO    [secret] Please see also https://aquasecurity.github.io/trivy/dev/docs/scanner/secret#recommendation for faster secret detection  
2025-03-31T14:59:13+02:00       INFO    Detected OS     family="alpine" version="3.21.3"  
2025-03-31T14:59:13+02:00       WARN    This OS version is not on the EOL list  family="alpine" version="3.21"  
2025-03-31T14:59:13+02:00       INFO    [alpine] Detecting vulnerabilities...   os_version="3.21" repository="3.21" pkg_num=71  
2025-03-31T14:59:13+02:00       INFO    Number of language-specific files       num=1  
2025-03-31T14:59:13+02:00       INFO    [node-pkg] Detecting vulnerabilities...  
2025-03-31T14:59:13+02:00       INFO    Table result includes only package filenames. Use '--format json' option to get the full path to the package file.  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
│                                     ├─────────────────────┼──────────┤          │                   ├────────────────────────────────────┼───────────────────────────────────────────────  
───────────────┤  
│                                     │ CVE-2021-32640      │ MEDIUM   │          │                   │ 7.4.6, 6.2.2, 5.2.3                │ nodejs-ws: Specially crafted value of the       
              │  
│                                     │                     │          │          │                   │                                    │ `Sec-Websocket-Protocol` header can be used to  
...            │  
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2021-32640      
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
├─────────────────────────────────────┼─────────────────────┤          │          ├───────────────────┼────────────────────────────────────┼───────────────────────────────────────────────  
───────────────┤  
│ xml2js (package.json)               │ CVE-2023-0842       │          │          │ 0.4.19            │ 0.5.0                              │ node-xml2js: xml2js is vulnerable to prototype  
pollution     │  
│                                     │                     │          │          │                   │                                    │ https://avd.aquasec.com/nvd/cve-2023-0842       
              │  
│                                     │                     │          │          ├───────────────────┤                                    │                                                 
              │  
│                                     │                     │          │          │ 0.4.23            │                                    │                                                 
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
│                                     │                     │          │          ├───────────────────┤                                    │                                                 
              │  
│                                     │                     │          │          │ 0.4.4             │                                    │                                                 
              │  
│                                     │                     │          │          │                   │                                    │                                                 
              │  
└─────────────────────────────────────┴─────────────────────┴──────────┴──────────┴───────────────────┴────────────────────────────────────┴───────────────────────────────────────────────  
───────────────┘
```


#### Postgres
```console
alt@TP1-YMCA:~$ trivy image postgres:15-alpine | tail -20  
2025-03-31T13:03:25Z    INFO    [vulndb] Need to update DB  
2025-03-31T13:03:25Z    INFO    [vulndb] Downloading vulnerability DB...  
2025-03-31T13:03:25Z    INFO    [vulndb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-db:2"  
61.66 MiB / 61.66 MiB [------------------------------------------------------------------------------------------------------------------------------------------] 100.00% 4.37 MiB p/s 14s  
2025-03-31T13:03:40Z    INFO    [vulndb] Artifact successfully downloaded       repo="mirror.gcr.io/aquasec/trivy-db:2"  
2025-03-31T13:03:40Z    INFO    [vuln] Vulnerability scanning is enabled  
2025-03-31T13:03:40Z    INFO    [secret] Secret scanning is enabled  
2025-03-31T13:03:40Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning  
2025-03-31T13:03:40Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection  
2025-03-31T13:04:10Z    INFO    Detected OS     family="alpine" version="3.21.3"  
2025-03-31T13:04:10Z    INFO    [alpine] Detecting vulnerabilities...   os_version="3.21" repository="3.21" pkg_num=46  
2025-03-31T13:04:11Z    INFO    Number of language-specific files       num=1  
2025-03-31T13:04:11Z    INFO    [gobinary] Detecting vulnerabilities...  
2025-03-31T13:04:11Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.61/docs/scanner/vulnerability#severity-selection for details.  
│         │ CVE-2024-34158 │          │        │                   │                                  │ go/build/constraint: golang: Calling Parse on a "// +build"  │  
│         │                │          │        │                   │                                  │ build tag line with...                                       │  
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-34158                   │  
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤  
│         │ CVE-2024-45336 │          │        │                   │ 1.22.11, 1.23.5, 1.24.0-rc.2     │ golang: net/http: net/http: sensitive headers incorrectly    │  
│         │                │          │        │                   │                                  │ sent after cross-domain redirect                             │  
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-45336                   │  
│         ├────────────────┤          │        │                   │                                  ├──────────────────────────────────────────────────────────────┤  
│         │ CVE-2024-45341 │          │        │                   │                                  │ golang: crypto/x509: crypto/x509: usage of IPv6 zone IDs can │  
│         │                │          │        │                   │                                  │ bypass URI name...                                           │  
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-45341                   │  
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤  
│         │ CVE-2025-22866 │          │        │                   │ 1.22.12, 1.23.6, 1.24.0-rc.3     │ crypto/internal/nistec: golang: Timing sidechannel for P-256 │  
│         │                │          │        │                   │                                  │ on ppc64le in crypto/internal/nistec                         │  
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2025-22866                   │  
│         ├────────────────┼──────────┤        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤  
│         │ CVE-2022-30629 │ LOW      │        │                   │ 1.17.11, 1.18.3                  │ golang: crypto/tls: session tickets lack random              │  
│         │                │          │        │                   │                                  │ ticket_age_add                                               │  
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2022-30629                   │  
└─────────┴────────────────┴──────────┴────────┴───────────────────┴──────────────────────────────────┴──────────────────────────────────────────────────────────────┘
```

#### Custom Apache
```console
alt@TP1-YMCA:~$ trivy image hooray:latest | tail -20  
2025-03-31T13:05:03Z    INFO    [vuln] Vulnerability scanning is enabled  
2025-03-31T13:05:03Z    INFO    [secret] Secret scanning is enabled  
2025-03-31T13:05:03Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning  
2025-03-31T13:05:03Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection  
2025-03-31T13:05:03Z    INFO    Detected OS     family="alpine" version="3.21.3"  
2025-03-31T13:05:03Z    INFO    [alpine] Detecting vulnerabilities...   os_version="3.21" repository="3.21" pkg_num=21  
2025-03-31T13:05:03Z    INFO    Number of language-specific files       num=0  
  
Report Summary  
  
┌───────────────────────────────┬────────┬─────────────────┬─────────┐  
│            Target             │  Type  │ Vulnerabilities │ Secrets │  
├───────────────────────────────┼────────┼─────────────────┼─────────┤  
│ hooray:latest (alpine 3.21.3) │ alpine │        0        │    -    │  
└───────────────────────────────┴────────┴─────────────────┴─────────┘  
Legend:  
- '-': Not scanned  
- '0': Clean (no security findings detected)
```

#### NGINX
```console
alt@TP1-YMCA:~$ trivy image nginx | tail -20  
2025-03-31T13:06:18Z    INFO    [vuln] Vulnerability scanning is enabled  
2025-03-31T13:06:18Z    INFO    [secret] Secret scanning is enabled  
2025-03-31T13:06:18Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning  
2025-03-31T13:06:18Z    INFO    [secret] Please see also https://trivy.dev/v0.61/docs/scanner/secret#recommendation for faster secret detection  
2025-03-31T13:06:42Z    INFO    [javadb] Downloading Java DB...  
2025-03-31T13:06:42Z    INFO    [javadb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-java-db:1"  
705.31 MiB / 705.31 MiB [--------------------------------------------------------------------------------------------------------------------------------------] 100.00% 9.63 MiB p/s 1m13s  
2025-03-31T13:07:56Z    INFO    [javadb] Artifact successfully downloaded       repo="mirror.gcr.io/aquasec/trivy-java-db:1"  
2025-03-31T13:07:56Z    INFO    [javadb] Java DB is cached for 3 days. If you want to update the database more frequently, "trivy clean --java-db" command clears the DB cache.  
2025-03-31T13:07:56Z    INFO    Detected OS     family="debian" version="12.10"  
2025-03-31T13:07:56Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=149  
2025-03-31T13:07:56Z    INFO    Number of language-specific files       num=0  
2025-03-31T13:07:56Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.61/docs/scanner/vulnerability#severity-selection for details.  
│ tar                │ CVE-2005-2541       │          │              │ 1.34+dfsg-1.2+deb12u1   │                  │ tar: does not properly warn the user when extracting setuid  │  
│                    │                     │          │              │                         │                  │ or setgid...                                                 │  
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2005-2541                    │  
│                    ├─────────────────────┤          │              │                         ├──────────────────┼──────────────────────────────────────────────────────────────┤  
│                    │ TEMP-0290435-0B57B5 │          │              │                         │                  │ [tar's rmt command may have undesired side effects]          │  
│                    │                     │          │              │                         │                  │ https://security-tracker.debian.org/tracker/TEMP-0290435-0B- │  
│                    │                     │          │              │                         │                  │ 57B5                                                         │  
├────────────────────┼─────────────────────┤          │              ├─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤  
│ util-linux         │ CVE-2022-0563       │          │              │ 2.38.1-5+deb12u3        │                  │ util-linux: partial disclosure of arbitrary files in chfn    │  
│                    │                     │          │              │                         │                  │ and chsh when compiled...                                    │  
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2022-0563                    │  
├────────────────────┤                     │          │              │                         ├──────────────────┤                                                              │  
│ util-linux-extra   │                     │          │              │                         │                  │                                                              │  
│                    │                     │          │              │                         │                  │                                                              │  
│                    │                     │          │              │                         │                  │                                                              │  
├────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼──────────────────┼──────────────────────────────────────────────────────────────┤  
│ zlib1g             │ CVE-2023-45853      │ CRITICAL │ will_not_fix │ 1:1.2.13.dfsg-1         │                  │ zlib: integer overflow and resultant heap-based buffer       │  
│                    │                     │          │              │                         │                  │ overflow in zipOpenNewFileInZip4_6                           │  
│                    │                     │          │              │                         │                  │ https://avd.aquasec.com/nvd/cve-2023-45853                   │  
└────────────────────┴─────────────────────┴──────────┴──────────────┴─────────────────────────┴──────────────────┴──────────────────────────────────────────────────────────────┘
```