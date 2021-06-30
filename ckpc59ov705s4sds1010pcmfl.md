## Run Damn Vulnerable Web Application (DVWA) from a Docker container

The best way to learn hacking is to keep practicing. But, remember what does it mean to hack ethically:

> Actions should only be executed on your **own** system, lab environment or a system that you are **charged with protecting**. If ownership and responsibility lie with another party, be sure to get **clear written instructions** with **explicit permission** to conduct ethical hacking activities. **Do not** investigate individuals, websites, servers, or conduct any illegal activities on any system you do not have permission to analyze.

So, at some point of your learning path, in my opinion, you should have many vulnerables at your disposal, and you should be in total control over them. One of such examples can be Damn Vulnerable Web Application.

***
# Contents
1. [Obtaining](#obtaining)
2. [Running](#running)
3. [Access](#access)
4. [Configuration](#configuration)
5. [Persistence](#persistence)
6. [Additional readings](#additional-readings)
***

# Obtaining

###   ⚠**Disclaimer**⚠

> Damn Vulnerable Web Application is damn vulnerable! **Do not** upload it to your hosting provider's public html folder or any Internet facing servers, as they will be **compromised**. 
> We **do not** take responsibility for the way in which any one uses this application (DVWA). We have made the purposes of the application clear and it **should not** be used maliciously. We have given warnings and taken measures to prevent users from installing DVWA on to live web servers. If your web server is compromised via an installation of DVWA, it is not our responsibility, it is the responsibility of the person/s who uploaded and installed it. // Quoted from DVWA page, formatted by me

Pull the docker image as described in the [Docker section](https://github.com/digininja/DVWA#docker-container) of GitHub README.

```sh
# DockerHub: https://hub.docker.com/r/vulnerables/web-dvwa/

$ docker pull vulnerables/web-dvwa
```

Ensure docker image is available

```sh
$ docker images

REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
hello-world            latest    d1165f221234   2 months ago   13.3kB
vulnerables/web-dvwa   latest    ab0d83586b6e   2 years ago    712MB
```

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring on the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/bePatron?u=57522747) which also gives you some bonus perks.

%%[patreon-btn]

# Running

Run the docker image using the following options

`docker run --rm -it -d -p 127.0.0.1:80:80 vulnerables/web-dvwa`

```text
--rm                   Automatically remove the container when it exits
-i, --interactive      Keep STDIN open even if not attached
-t, --tty              Allocate a pseudo-TTY
-d, --detach           Run container in background and print container ID
-p, --publish list     Publish a container's port(s) to the host
```

Using `-p 127.0.0.1:80:80` will ensure that docker is not exposing the port to the outside. It binds `:80` for `127.0.0.1` interface only.

![Open ports after docker run](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272280056/WJ91XhpRI.png)

# Access

If your DVWA is also bound to `:80`, navigate in browser to `localhost`.

![DVWA login page](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272303218/lCI2MyiPZ.png)

Login using default `admin/password` credentials.

![Database Setup](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272322285/UY7L7lCpt.png)

As you can see, my instance is missing the following features:
* PHP allow_url_include
* reCAPTCHA key

# Configuration

Copy `php.ini` directly from the docker using `docker cp` command and change the turn `allow_url_include` on

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND      CREATED          STATUS          PORTS                  NAMES
35369924c11e   vulnerables/web-dvwa   "/main.sh"   50 minutes ago   Up 50 minutes   127.0.0.1:80->80/tcp   flamboyant_dirac

$ docker cp 35369924c11e:/etc/php/7.0/apache2/php.ini .
$ subl php.ini
```

![Change php.ini to allow URL include](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272407083/lZABoWEW-.png)

Copy the `php.ini` back to docker container

```sh
$ docker cp php.ini 35369924c11e:/etc/php/7.0/apache2/php.ini
```

Attach to the docker instance with bash shell and restart Apache

```text
$ docker exec -it 35369924c11e bash
root@35369924c11e:/# service apache2 restart
[....] Restarting Apache httpd web server: apache2AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
. ok
```

Now if you refresh the page you will see that `allow_url_include` is no longer reporting as disabled.

![Database Setup after php.ini changes](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272424519/_wptBmth9.png)

## reCAPTCHA

Now to the reCAPTCHA configuration. Go to [reCAPTCHA Admin](https://google.com/recaptcha/admin/create) (or search for "google recaptcha admin"). Generate v3 API keys, domain names don't matter.


![reCAPTCHA admin panel](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272474887/Xybaz8a9S.png)

You can navigate in docker shell to `/var/www/html/config` and try to modify the config file - but none of the text editors is installed, so we go with the `docker cp` way.

Add reCAPTCHA keys to the config file:

```sh
$ docker cp 35369924c11e:/var/www/html/config/config.inc.php .
$ subl config.inc.php
```

![reCAPTCHA API keys](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272608809/FPkwgNNPc.png)

And copy back to the docker.

`$ docker cp config.inc.php 35369924c11e:/var/www/html/config/`

Now last time, refresh the `setup.php`

![Final configuration](https://cdn.hashnode.com/res/hashnode/image/upload/v1622273843587/ptZG4Szrh.png)

Click _Create/Reset Database_. You will be logged out after a while, login again. Congratulation, you have configured **Damn Vulnerable Web Application**.

![DVWA Home](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272770664/8BhowN0U1.png)

Last thing to say is that DVWA has 4 difficulty (security) settings. Go to the _DVWA Security_ section to set the _Security Level_.

![DVWA Security](https://cdn.hashnode.com/res/hashnode/image/upload/v1622272787791/Y4pHAxYmQ.png)

# Persistence

Now that we have modified the **running** docker container, we should somehow make our changes persist through consecutive runs (because next time we start a docker image it will start as a fresh instance - that's the whole idea of containers). This can be done at least in two ways I am aware of - and I didn't find any meaningful differences between them.

## Patch the original image

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND      CREATED       STATUS       PORTS                  NAMES
35369924c11e   vulnerables/web-dvwa   "/main.sh"   3 hours ago   Up 3 hours   127.0.0.1:80->80/tcp   flamboyant_dirac

$ docker commit 35369924c11e vulnerables/web-dvwa:patched
```

And verify that a new image is created.

```sh
$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
vulnerables/web-dvwa       patched   85375b203721   15 seconds ago   826MB
hello-world                latest    d1165f221234   2 months ago     13.3kB
vulnerables/web-dvwa       latest    ab0d83586b6e   2 years ago      712MB

$ docker images vulnerables/web-dvwa
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
vulnerables/web-dvwa   patched   a17d6c740d6e   17 seconds ago   826MB
vulnerables/web-dvwa   latest    ab0d83586b6e   2 years ago      712MB
```

If you want to run patched version specify tag, like so:

```sh
$ docker run --rm -it -d -p 127.0.0.1:80:80 vulnerables/web-dvwa:patched
```

## Create a new image

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND      CREATED       STATUS       PORTS                  NAMES
35369924c11e   vulnerables/web-dvwa   "/main.sh"   3 hours ago   Up 3 hours   127.0.0.1:80->80/tcp   flamboyant_dirac

$ docker commit 35369924c11e vulnerables/dvwa-patched
```

And verify that a new image is created - you start patched version by running this new image.

```sh
$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
vulnerables/dvwa-patched   latest    85375b203721   15 seconds ago   826MB
hello-world                latest    d1165f221234   2 months ago     13.3kB
vulnerables/web-dvwa       latest    ab0d83586b6e   2 years ago      712MB
```

> **Did you know about DVWA before reading this article? Have you been using it? Let me know in the comments below.**

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)


* [DVWA Home](https://dvwa.co.uk/)
* [Official DockerHub for DVWA](https://hub.docker.com/r/vulnerables/web-dvwa)
* [Docker Docs: Container networking](https://docs.docker.com/config/containers/container-networking/)
* [Publish or expose port (-p, --expose)](https://docs.docker.com/engine/reference/commandline/run/#publish-or-expose-port--p---expose)
* [Google reCAPTCHA Docs](https://developers.google.com/recaptcha/docs/v3)