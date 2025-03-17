# Part I : Docker basics

## 1. Install

🌞 **Installer Docker votre machine Azure**

~~~bash
azureuser@Test:~$ sudo systemctl start docker.service

azureuser@Test:~$ sudo systemctl status docker.service
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-03-14 10:16:17 UTC; 5min ago
~~~

~~~bash
azureuser@Test:~$ sudo usermod -aG docker $(whoami)
azureuser@Test:~$ exit
~~~

Teste si Docker fonctionne sans sudo (oui donc user bien ajouté):
~~~bash
azureuser@Test:~$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
~~~

## 2. Vérifier l'install

```bash
# Info sur l'install actuelle de Docker
$ docker info

azureuser@Test:~$ docker info
Client: Docker Engine - Community
 Version:    28.0.1
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.21.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.33.1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose
    [...]
```

## 3. Lancement de conteneurs

🌞 **Utiliser la commande `docker run`**

~~~bash
azureuser@Test:~$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html -p 9999:80 nginx
5587fdf48cbc92780cb9211fc385a090611f441f1f04f38010286a952ecf7381
azureuser@Test:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
5587fdf48cbc   nginx     "/docker-entrypoint.…"   9 seconds ago   Up 8 seconds   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp   web
~~~

🌞 **Rendre le service dispo sur internet**

~~~bash
C:\Users\Utilisateur>curl 4.186.56.93:9999
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.27.4</center>
</body>
</html>
~~~

🌞 **Custom un peu le lancement du conteneur**

~~~bash
azureuser@Test:~$ docker run --name meow -d -p 7777:7777 -v /var/www/tp_docker/index.html:/var/www/tp_docker/index.html -v /etc/nginx/conf.d/nginx.conf:/etc
/nginx/conf.d/nginx.conf --memory=512m nginx
c328dbde6944ee9306a00595783afc00386c6a51d0b611f62e5146df2f1785a5

azureuser@Test:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED             STATUS             PORTS                                                 NAMES
c328dbde6944   nginx     "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds       80/tcp, 0.0.0.0:7777->7777/tcp, [::]:7777->7777/tcp   meow
5587fdf48cbc   nginx     "/docker-entrypoint.…"   About an hour ago   Up About an hour   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp               web
~~~

~~~bash
C:\Users\Utilisateur>curl 4.186.56.93:7777
<h1>Page Nginx</h1>
~~~

🌞 **Call me**

# Part II : Images

## Construisez votre propre Dockerfile

🌞 **Construire votre propre image**

`Dockerfile` d'une image Debian :

~~~bash
azureuser@Test:~$ cat Dockerfile
FROM debian:latest

RUN apt update -y && apt install -y apache2

COPY index.html /var/www/html/index.html

COPY apache2.conf /etc/apache2/apache2.conf

EXPOSE 8888

CMD [ "apache2", "-DFOREGROUND" ]
~~~

Test si l'image fonctionne : Build OK :
~~~bash
azureuser@Test:~$ docker build . -t debian
[+] Building 25.7s (9/9) FINISHED                                                                        docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 238B                                                                               0.0s
 => [internal] load metadata for docker.io/library/debian:latest                                                   0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => CACHED [1/4] FROM docker.io/library/debian:latest                                                              0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 797B                                                                                  0.0s
 => [2/4] RUN apt update -y && apt install -y apache2                                                             20.3s
 => [3/4] COPY index.html /var/www/html/index.html                                                                 0.1s
 => [4/4] COPY apache2.conf /etc/apache2/apache2.conf                                                              0.1s
 => exporting to image                                                                                             4.8s
 => => exporting layers                                                                                            4.8s
 => => writing image sha256:4e08a93820a3c02d750943a1e9b76974880b629c3c7fdf423800b91cc61c9ce8                       0.0s
 => => naming to docker.io/library/debian
~~~

Run OK :
~~~bash
azureuser@Test:~$ docker run -d -p 8888:8888 debian
f4a4c5fd322a17f0e45c8550144bcb13226ca95fe1352d75538f516eff00f0d0
~~~

Accès à l'index.html serveur NICKEL :
~~~bash
azureuser@Test:~$ curl 4.186.56.93:8888
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bienvenue sur mon serveur Apache</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
            background-color: #f4f4f4;
        }
        h1 {
            color: #333;
        }
        p {
            font-size: 18px;
        }
    </style>
</head>
<body>
    <h1>Bienvenue sur mon serveur Apache dans Docker ! 🚀</h1>
    <p>Ce serveur tourne dans un conteneur Docker basé sur Debian.</p>
    <p>Si vous voyez cette page, c'est que tout fonctionne bien !</p>
</body>
</html>
~~~

# Part III : `docker-compose`

🌞 **Installez un WikiJS** en utilisant Docker

~~~bash
azureuser@Test:~/WikiJS$ cat docker-compose.yml
version: '3'

services:
  db:
    image: postgres:15
    container_name: wikijs_db
    restart: always
    environment:
      POSTGRES_DB: wiki
      POSTGRES_USER: wikijs
      POSTGRES_PASSWORD: mysecretpassword
    volumes:
      - db_data:/var/lib/postgresql/data

  wikijs:
    image: requarks/wiki:2
    container_name: wikijs
    restart: always
    depends_on:
      - db
    ports:
      - "8787:3000"
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: mysecretpassword
      DB_NAME: wiki

volumes:
  db_data:
~~~

~~~bash
azureuser@Test:~/WikiJS$ docker-compose up -d
Creating network "wikijs_default" with the default driver
Creating volume "wikijs_db_data" with default driver
[...]
Creating wikijs_db ... done
Creating wikijs    ... done
~~~

~~~bash
azureuser@Test:~/WikiJS$ curl http://4.186.56.93:8787/
<!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-scale=5"><meta name="theme-color" content="#1976d2"><meta name="msapplication-TileColor" content="#1976d2"><meta name="msapplication-TileImage" content="/_assets/favicons/mstile-150x150.png"><title>Wiki.js Setup</title><link rel="apple-touch-icon" sizes="180x180" href="/_assets/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="192x192" href="/_assets/favicons/android-chrome-192x192.png"><link rel="icon" type="image/png" sizes="32x32" href="/_assets/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/_assets/favicons/favicon-16x16.png"><link rel="mask-icon" href="/_assets/favicons/safari-pinned-tab.svg" color="#1976d2"><link rel="manifest" href="/_assets/manifest.json"><script>var siteConfig = {"title":"Wiki.js"}
</script><link type="text/css" rel="stylesheet" href="/_assets/css/setup.22871ffac1b643eed4d9.css"><script type="text/javascript" src="/_assets/js/runtime.js?1738531300"></script><script type="text/javascript" src="/_assets/js/setup.js?1738531300"></script></head><body><div id="root"><setup wiki-version="2.5.306"></setup></div></body></html>
~~~

## 3. Make your own meow



🌞 **Vous devez :**

`Dockerfile` pour python3 :

~~~bash
azureuser@Test:~/Meow$ cat Dockerfile
FROM python:3.10

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8888

CMD ["python", "app.py"]
~~~

`docker-compose.yml` pour python + redis :

~~~bash
azureuser@Test:~/Meow$ cat docker-compose.yml
version: '3'

services:
  redis:
    image: redis:latest
    container_name: my_redis
    restart: always
    ports:
      - "6379:6379"

  app:
    build: .
    container_name: my_python_app
    restart: always
    depends_on:
      - redis
    ports:
      - "8888:8888"
    environment:
      REDIS_HOST: redis
      REDIS_PORT: 6379
~~~

Build des conteneur validé :

~~~bash
azureuser@Test:~/Meow$ docker-compose up -d --build
Building app
[...]
my_redis is up-to-date
Starting my_python_app ... done
~~~

App accessible :

~~~bash
azureuser@Test:~/Meow$ curl http://4.186.56.93:8888/
<h1>Add key</h1>
<form action="/add" method="POST">
    Key:
    <input type="text" name="key">
    Value:
    <input type="text" name="value">
    <input type="submit" value="Submit">
</form>

<h1>Check key</h1>
<form action="/get" method="POST">
    Key:
    <input type="text" name="key">
    <input type="submit" value="Submit">
</form>
~~~

# Part IV : Docker security

🌞 **Prouvez que vous pouvez devenir `root`**

~~~bash
azureuser@Test:~$ docker run --rm -v /:/mnt alpine cat /mnt/etc/shadow
root:*:20140:0:99999:7:::
daemon:*:20140:0:99999:7:::
bin:*:20140:0:99999:7:::
sys:*:20140:0:99999:7:::
sync:*:20140:0:99999:7:::
games:*:20140:0:99999:7:::
man:*:20140:0:99999:7:::
lp:*:20140:0:99999:7:::
mail:*:20140:0:99999:7:::
news:*:20140:0:99999:7:::
uucp:*:20140:0:99999:7:::
[...]
~~~

## 2. Scan de vuln

🌞 **Utilisez Trivy**

Installation de Trivy :

~~~bash
sudo snap install trivy
~~~

Test complet sur nginx :

~~~bash
azureuser@Test:~$ trivy image nginx:latest
2025-03-17T20:07:29Z    INFO    Vulnerability scanning is enabled
2025-03-17T20:07:29Z    INFO    Secret scanning is enabled
2025-03-17T20:07:29Z    INFO    If your scanning is slow, please try '--scanners vuln' to disable secret scanning
[...]

nginx:latest (debian 12.9)

Total: 156 (UNKNOWN: 0, LOW: 99, MEDIUM: 42, HIGH: 13, CRITICAL: 2)

[TABLEAU DE LA MORT QUI TUE...]
~~~

Scan plus "rapide" :

~~~bash
azureuser@Test:~$ trivy image --scanners vuln debian:latest
2025-03-17T20:28:31Z    INFO    Vulnerability scanning is enabled
2025-03-17T20:28:34Z    INFO    Detected OS     family="debian" version="12.9"
2025-03-17T20:28:34Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=88
2025-03-17T20:28:34Z    INFO    Number of language-specific files       num=0

debian:latest (debian 12.9)

Total: 76 (UNKNOWN: 0, LOW: 57, MEDIUM: 17, HIGH: 1, CRITICAL: 1)

[TABLEAU DE LA MORT QUI TUE...]
~~~

## 3. Petit benchmark secu


🌞 **Utilisez l'outil Docker Bench for Security**

Score au top 100% secure no problem :

~~~bash
Section C - Score

[INFO] Checks: 117
[INFO] Score: 5
~~~

Outils toujours très sympa à connaitre mais peut-être un petit manque de doc sur le score; okay 5 mais sur combien ? Globalement c'est pas terrible sans surprise mais avoir un petit message qui nous rajoute si c'est 5 parce que c'est de la merde ou 5 parce que rien n'est opti mais pas trop de "risque" ça sera cool. Avec pour référence d'autre outil d'audit PingCastle, où y'a presque TROP de doc c'est très différent mais ça reste cool à connaître !!
