# URL

- https://gist.github.com/huilapman/8a5603c8a35b9dc97b8be67c646bc829

- Website <https://redquants.com> <https://www.redquants.com>
- File manager <https://file.redquants.com>
- Web SSH Client <https://ssh.redquants.com>
- Monitoring <https://netdata.redquants.com>
- Wordpress <https://wordpress.redquants.com>
- Webmail <https://webmail.redquants.com>
- Webmail admin <https://webmail.redquants.com/admin/>

# Hack
 - <https://nitifilter.com/en/ive-been-hacked/>
 - <https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user>

Crypto bots are back
This week-end, our server ( VPS @OVH ) has been hacked ( brute force root login ).
I noticed that I was unable to log in through ssh as usual ( using RSA or ED25519 keys ) but need a password.

After successfully logging in, I found out that my authorized_keys file has been change with only one RSA key ( all others have been overwritten ) having the comment mdrfckr.

After a little Googling, I find the following article : Outlaw is Back, a New Crypto-Botnet…

Even if my version was not exactly identical, I did get me on the right path to clean it :

Erased the authorized_keys file
Killed the rsync process
Looked at the crontab for user root ( crontab -l ) – get a bunch of stuff :
1 1 */2 * * /root/.configrc/a/upd>/dev/null 2>&1
@reboot /root/.configrc/a/upd>/dev/null 2>&1
5 8 * * 0 /root/.configrc/b/sync>/dev/null 2>&1
@reboot /root/.configrc/b/sync>/dev/null 2>&1
        0 0 */3 * * /tmp/.X25-unix/.rsync/c/aptitude>/dev/null 2>&1
        Deleted the crontab for user root ( crontab -d ). (As you can see, on reboot it will reinstall itself otherwise )
        Deleted the directory : rm -fR /root/.configrc and anything in /tmp that seems suspicious.
        Find out through top that a process was grabbing 99% of the CPU ( mine was named kswapd0 )
        killed this process
        did a find / -name kswapd0 to find where this code was : it was not in /.rsync as in the article but in /root/.configrc so  already cleaned ( step 5 )
        reboot the server.
        Everything seems OK == according to the link above, this hack is just a crypto bot so I didn’t bother to re install the whole server even it is a best practices
        Hopes it will help somebody somewhere 😉

# Ubuntu

root

    apt update
    apt install lsb-release ca-certificates apt-transport-https software-properties-common -y
    apt install net-tools -y
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    apt update
    apt install docker-ce
    systemctl status docker
    apt-get install docker-compose-plugin
    apt upgrade
    apt install docker-compose
    adduser ubuntu
    su ubuntu -
    usermod -aG sudo ubuntu
    sudo usermod -aG docker $USER
    exit
    vi ~/.ssh/authorized_keys
    mkdir -p /home/ubuntu/.ssh
    cp ~/.ssh/authorized_keys /home/ubuntu/.ssh/
    chown ubuntu:ubuntu -R /home/ubuntu/.ssh
   
ubuntu

    sudo usermod -aG docker $USER
    docker ps -a
    sudo chmod 666 /var/run/docker.sock
    docker ps -a
    sudo systemctl status docker
    sudo systemctl restart docker

time zone

    root@localhost:~# timedatectl
                   Local time: Fri 2022-07-29 22:00:01 UTC
               Universal time: Fri 2022-07-29 22:00:01 UTC
                     RTC time: Fri 2022-07-29 22:00:01
                    Time zone: Etc/UTC (UTC, +0000)
    System clock synchronized: yes
                  NTP service: active
              RTC in local TZ: no
    root@localhost:~# timedatectl list-timezones | grep -i Hong_Kong
    Asia/Hong_Kong
    root@localhost:~# unlink /etc/localtime
    root@localhost:~# ln -s /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
    root@localhost:~# timedatectl
                   Local time: Sat 2022-07-30 06:01:15 HKT
               Universal time: Fri 2022-07-29 22:01:15 UTC
                     RTC time: Fri 2022-07-29 22:01:15
                    Time zone: Asia/Hong_Kong (HKT, +0800)
    System clock synchronized: yes
                  NTP service: active
              RTC in local TZ: no


ssh authorized_keys

    root@localhost:~# cat ~/.ssh/authorized_keys
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCeSdn4FZioICfQYbQXmjTQpG4uwCCSSYScey9uAYYySVOOD3AVTBIhZ1tm1awBS9BMu2nG69pcgYukruqRYow59CZjICOfBfr5ha9dc3O3RVrZPNWtltMEO2f3jMmEdqoqS0RFxI12JUrqs67AkPNOk6Rz0eoiWevqEGEu0uqvkkTmwwcnm+fxvNoWkQNTbhA6jkZENKgFvYvQQTsbHrPklAhKUlQ/z2haUwPUQc53WHX3qr3sMVk3OkXwcSF4w21xL+py6bBb4D/tyA8H23yrkcPcd6idIJSVCrZJTQpc7Qt9SIFoz+Kv1rfKpoO4EhlSbyeLdiYkMmwe6AY/0i6n sysroot@NWTs-MacBook-Pro.local

# Nginx
basic auth
    
    sudo apt-get install apache2-utils
    sudo htpasswd -c /home/ubuntu/nginx/.htpasswd admin    
        
default.conf

    /home/ubuntu/nginx/conf.d/default.conf

    server {
        listen       80 default_server;
        server_name  redquants.com;
        return 301 https://$host$request_uri;
    }

    upstream roundcube {
        server 127.0.0.1:8080;
    }

    upstream filemanager {
        server 127.0.0.1:9080;
    }

    upstream netdata {
        server 127.0.0.1:19999;
    }
    
    server {
        listen       443 ssl;
        server_name  www.redquants.com;

        # ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        #access_log  /var/log/nginx/host.access.log  main;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    server {
        listen       443 ssl;
        server_name  webmail.redquants.com;
        
        client_max_body_size 100M;
        # ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        #access_log  /var/log/nginx/host.access.log  main;

        location = /admin {
            rewrite ^([^.]*[^/])$ $1/ permanent;
            proxy_pass      http://roundcube;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
        location / {
            #rewrite ^/webmail/?(.*)$ /$1 break;
            proxy_pass      http://roundcube;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }

    server {
        listen       443 ssl;
        server_name  file.redquants.com;

        # ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        #access_log  /var/log/nginx/host.access.log  main;

        location / {
            #//turn on auth for this location
            auth_basic "Restricted Content";
            auth_basic_user_file /etc/nginx/.htpasswd;

            proxy_pass      http://filemanager;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
    
    server {
        listen       443 ssl;
        server_name  ssh.redquants.com;

        # ssl_session_timeout  5m;    #缓存有效期
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    #安全链接可选的加密协议
        ssl_prefer_server_ciphers on;   #使用服务器端的首选算法
        #ssl_certificate /etc/nginx/ssl/nginx.crt;
        #ssl_certificate_key /etc/nginx/ssl/nginx.key;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        #access_log  /var/log/nginx/host.access.log  main;

        location / {
            #//turn on auth for this location
            auth_basic "Restricted Content";
            auth_basic_user_file /etc/nginx/.htpasswd;

            # rewrite ^/filemanager/?(.*)$ /$1 break;
            proxy_pass      http://webssh;
            proxy_http_version 1.1;
            proxy_read_timeout 300;
            proxy_set_header   Host $http_host;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection "upgrade";
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Real-PORT $remote_port;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }

    }

    server {
        listen       443 ssl;
        server_name  netdata.redquants.com;

        # ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        #access_log  /var/log/nginx/host.access.log  main;

        location / {
            #//turn on auth for this location
            auth_basic "Restricted Content";
            auth_basic_user_file /etc/nginx/.htpasswd;

            proxy_pass      http://netdata;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }

# Firewall

iptables in docker
    
    ubuntu@localhost:~/docker$ cat /etc/docker/daemon.json
    {
        "iptables": true,
        "icc": true
    }

iptables 
    
    sudo iptables -t raw -A PREROUTING -i eth0 -p tcp --dport 7080 -j DROP
    sudo iptables -t raw -A PREROUTING -i eth0 -p tcp --dport 9080 -j DROP
    sudo iptables -t raw -A PREROUTING -i eth0 -p tcp --dport 10000 -j DROP
    sudo iptables -t raw -A PREROUTING -i eth0 -p tcp --dport 19999 -j DROP
    
    sudo iptables -L -t raw
    Chain PREROUTING (policy ACCEPT)
    target     prot opt source               destination
    DROP       tcp  --  anywhere             anywhere             tcp dpt:7080
    DROP       tcp  --  anywhere             anywhere             tcp dpt:9080
    DROP       tcp  --  anywhere             anywhere             tcp dpt:webmin
    DROP       tcp  --  anywhere             anywhere             tcp dpt:19999

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination
    
ufw

    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 22/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 443/tcp
    sudo ufw allow 25/tcp

    ubuntu@localhost:~/nginx/conf.d$ sudo ufw status
    Status: active

    To                         Action      From
    --                         ------      ----
    80/tcp                     ALLOW       Anywhere
    443/tcp                    ALLOW       Anywhere
    25/tcp                     ALLOW       Anywhere
    22/tcp                     ALLOW       Anywhere
    80/tcp (v6)                ALLOW       Anywhere (v6)
    443/tcp (v6)               ALLOW       Anywhere (v6)
    25/tcp (v6)                ALLOW       Anywhere (v6)
    22/tcp (v6)                ALLOW       Anywhere (v6)

# Docker
## Docker Command
Reload 
  
    docker exec -it nginx nginx -s reload

Get UID

    ubuntu@localhost:~/nginx/html$ id
    uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo),999(docker)

Run

    docker run     \
    --rm     \
    --name nginx     \
    --net=host      \
    -v /home/ubuntu/nginx/html:/usr/share/nginx/html:ro     \
    -v /home/ubuntu/nginx/nginx.conf:/etc/nginx/nginx.conf     \
    -v /home/ubuntu/nginx/.htpasswd:/etc/nginx/.htpasswd     \
    -v /home/ubuntu/nginx/conf.d:/etc/nginx/conf.d     \
    -v /home/ubuntu/nginx/ssl:/etc/nginx/ssl     \
    -v /home/ubuntu/nginx/log:/var/log/nginx     \
    -d nginx:latest

    docker run \
    --rm \
    --name=file \
    --user 1000:1000 \
    -v /home/ubuntu/nginx/html:/data \
    -p 9080:8080 \
    -d serverwentdown/file-manager

    docker run \
    --rm     \
    --net=host \
    -h "mail.redquants.com"     \
    -e "HTTPS=OFF" \
    -e "HTTP_PORT=8080" \
    -e TZ=Asia/Hong_Kong     \
    -e DISABLE_CLAMAV=TRUE     \
    -e DISABLE_RSPAMD=TRUE     \
    -v /etc/localtime:/etc/localtime:ro     \
    -v /home/ubuntu/poste.io/data:/data     \
    --name poste     \
    -d analogic/poste.io

    docker run -d \
    --name=netdata   \
    -p 19999:19999   \
    -v netdataconfig:/etc/netdata   \
    -v netdatalib:/var/lib/netdata   \
    -v netdatacache:/var/cache/netdata   \
    -v /etc/passwd:/host/etc/passwd:ro   \
    -v /etc/group:/host/etc/group:ro   \
    -v /proc:/host/proc:ro   \
    -v /sys:/host/sys:ro   \
    -v /etc/os-release:/host/etc/os-release:ro   \
    --restart unless-stopped   \
    --cap-add SYS_PTRACE   \
    --security-opt apparmor=unconfined   \
    netdata/netdata


## Docker Compose
command

    ubuntu@localhost:~/docker$ docker-compose up -d
    Creating network "docker_default" with the default driver
    Creating nginx   ... done
    Creating file    ... done
    Creating poste   ... done
    Creating ssh     ... done
    Creating netdata ... done    
    ubuntu@localhost:~/docker$ docker-compose ps -a
     Name                Command                       State                              Ports
    ---------------------------------------------------------------------------------------------------------------
    file      docker-entrypoint.sh node  ...   Up             0.0.0.0:9080->8080/tcp,:::9080->8080/tcp
    netdata   /usr/sbin/run.sh                 Up (healthy)   0.0.0.0:19999->19999/tcp,:::19999->19999/tcp
    nginx     /docker-entrypoint.sh ngin ...   Up
    poste     /init                            Up (healthy)
    ssh       /usr/bin/dumb-init bash /e ...   Up             0.0.0.0:10000->8080/tcp,:::10000->8080/tcp
    ubuntu@localhost:~/docker$ docker-compose down
    Stopping netdata ... done
    Stopping ssh     ... done
    Stopping poste   ... done
    Stopping file    ... done
    Stopping nginx   ... done
    Removing netdata ... done
    Removing ssh     ... done
    Removing poste   ... done
    Removing file    ... done
    Removing nginx   ... done
    Removing network docker_default
    ubuntu@localhost:~/docker$ docker stats --no-stream
    CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O         PIDS
    05d74d96672b   file      0.00%     28.9MiB / 971.2MiB    2.98%     114kB / 79.3kB   9.8MB / 328kB     11
    7125abd5ede2   ssh       0.00%     29.42MiB / 971.2MiB   3.03%     14.8kB / 12kB    13.4MB / 418kB    4
    3f190847f5a3   nginx     0.00%     5.984MiB / 971.2MiB   0.62%     0B / 0B          6.59MB / 41kB     2
    3de3924688ad   netdata   2.44%     112MiB / 971.2MiB     11.53%    152kB / 2.05MB   51.2MB / 50.7MB   54
    5aab68246d87   poste     0.53%     112MiB / 971.2MiB     11.53%    0B / 0B          40.6MB / 19.7MB   76


docker-compose.yml

    ubuntu@localhost:~/docker$ cat docker-compose.yml    
    version: '3.8'
    services:
      poste:
        image: analogic/poste.io
        network_mode: host
        restart: unless-stopped
        container_name: poste
        hostname: mail.redquants.com
        environment:
          - HTTPS=OFF
          - HTTP_PORT=8080
          - DISABLE_CLAMAV=TRUE
          - DISABLE_RSPAMD=TRUE
          - TZ=Asia/Hong_Kong
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /home/ubuntu/poste.io/data:/data
      file:
         image: serverwentdown/file-manager
         restart: unless-stopped
         container_name: file
         user: 1000:1000
         volumes:
           - /home/ubuntu/nginx/html:/data
         ports:
           - 9080:8080
         environment:
           - TZ=Asia/Hong_Kong
      ssh:
         #image: snsyzb/webssh
         image: achaiah/alpine-webssh
         restart: unless-stopped
         container_name: ssh
         ports:
           - 10000:8080
         environment:
           - TZ=Asia/Hong_Kong
      netdata:
         image: netdata/netdata
         restart: unless-stopped
         container_name: netdata
         cap_add:
           - SYS_PTRACE
         volumes:
           - netdataconfig:/etc/netdata
           - netdatalib:/var/lib/netdata
           - netdatacache:/var/cache/netdata
           - /etc/passwd:/host/etc/passwd:ro
           - /etc/group:/host/etc/group:ro
           - /proc:/host/proc:ro
           - /sys:/host/sys:ro
           - /etc/os-release:/host/etc/os-release:ro
         security_opt:
           - apparmor=unconfined
         ports:
           - 19999:19999
         environment:
           - TZ=Asia/Hong_Kong
      wordpress_db:
        # We use a mariadb image which supports both amd64 & arm64 architecture
        image: mariadb:10.6.4-focal
        # If you really want to use MySQL, uncomment the following line
        #image: mysql:8.0.27
        container_name: wordpress_db
        command: '--default-authentication-plugin=mysql_native_password'
        volumes:
          - /home/ubuntu/wordpress/db_data:/var/lib/mysql
        restart: unless-stopped
        environment:
          - MYSQL_ROOT_PASSWORD=somewordpress
          - MYSQL_DATABASE=wordpress
          - MYSQL_USER=wordpress
          - MYSQL_PASSWORD=wordpress
          - TZ=Asia/Hong_Kong
        expose:
          - 3306
      wordpress:
        image: wordpress:latest
        container_name: wordpress
        ports:
          - 7080:80
        restart: unless-stopped
        volumes:
          - /home/ubuntu/wordpress/html:/var/www/html
        environment:
          - WORDPRESS_DB_HOST=wordpress_db
          - WORDPRESS_DB_USER=wordpress
          - WORDPRESS_DB_PASSWORD=wordpress
          - WORDPRESS_DB_NAME=wordpress
          - TZ=Asia/Hong_Kong
        depends_on:
          - wordpress_db
        links:
          - wordpress_db
      nginx:
         network_mode: host
         image: nginx:latest
         restart: unless-stopped
         container_name: nginx
         volumes:
           - /home/ubuntu/nginx/html:/usr/share/nginx/html:ro
           - /home/ubuntu/nginx/nginx.conf:/etc/nginx/nginx.conf
           - /home/ubuntu/nginx/.htpasswd:/etc/nginx/.htpasswd
           - /home/ubuntu/nginx/conf.d:/etc/nginx/conf.d
           - /home/ubuntu/nginx/ssl:/etc/nginx/ssl
           - /home/ubuntu/nginx/log:/var/log/nginx
         environment:
           - TZ=Asia/Hong_Kong
    volumes:
      netdataconfig:
      netdatalib:
      netdatacache:



# Let's Encrypt

    sudo certbot certonly --manual -m admin@redquants.com -d redquants.com -d *.redquants.com
    mv /etc/letsencrypt/live/
    cd /etc/letsencrypt/live/
    sudo /etc/letsencrypt/live/
    sudo cd /etc/letsencrypt/live/
    sudo ls -ltr /etc/letsencrypt/live
    sudo cp /etc/letsencrypt/live/redquants.com/fullchain.pem .
    sudo cp /etc/letsencrypt/live/redquants.com/privkey.pem .
    ls -ltr
    sudo chown ubuntu:ubuntu *.pem
    ls -ltr
    mv *.pem ./nginx/ssl/
    docker restart nginx
    sudo certbot renew
    cd nginx/html
    ls -ltr
    mkdir .well-known
    cd .well-known/
    mkdir acme-challenge
    cd acme-challenge/
    vi vS-LKgmwmf2V15IV3LEPGP2YBUO9d4XSX2UkKLh_-1g
    
    root@localhost:~# cat renew_cert.sh
    cp /etc/letsencrypt/live/redquants.com/fullchain.pem /home/ubuntu/nginx/ssl/
    cp /etc/letsencrypt/live/redquants.com/privkey.pem /home/ubuntu/nginx/ssl/
    sudo chown ubuntu:ubuntu /home/ubuntu/nginx/ssl/*.pem
    docker-compose --file /home/ubuntu/docker/docker-compose.yml restart nginx


    root@localhost:~# crontab -l
    # Edit this file to introduce tasks to be run by cron.
    #
    # Each task to run has to be defined through a single line
    # indicating with different fields when the task will be run
    # and what command to run for the task
    #
    # To define the time you can provide concrete values for
    # minute (m), hour (h), day of month (dom), month (mon),
    # and day of week (dow) or use '*' in these fields (for 'any').
    #
    # Notice that tasks will be started based on the cron's system
    # daemon's notion of time and timezones.
    #
    # Output of the crontab jobs (including errors) is sent through
    # email to the user the crontab file belongs to (unless redirected).
    #
    # For example, you can run a backup of all your user accounts
    # at 5 a.m every week with:
    # 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
    #
    # For more information see the manual pages of crontab(5) and cron(8)
    #
    # m h  dom mon dow   command

    0 0 * * * /usr/bin/certbot renew --quiet
    5 0 * * * /root/renew_cert.sh



# File

    ubuntu@localhost:~$ tree -L 2
    .
    ├── docker
    │   └── docker-compose.yml    
    ├── nginx
    │   ├── conf.d
    │   ├── html    
    │   ├── log
    │   ├── nginx.conf
    │   └── ssl
    ├── poste.io
        └── data


# DNS

    A	@	139.162.78.44	600 seconds
    NS	@	ns39.domaincontrol.com.	1 hour	
    NS	@	ns40.domaincontrol.com.	1 hour	
    CNAME	autoconfig	mail.redquants.com.	1/2 hour	
    CNAME	autodiscover	mail.redquants.com.	1/2 hour	
    CNAME	file	redquants.com.	1/2 hour	
    CNAME	ftp	redquants.com.	1/2 hour	
    CNAME	mail	redquants.com.	1/2 hour	
    CNAME	webmail	redquants.com.	1/2 hour	
    CNAME	www	redquants.com.	1/2 hour
    CNAME	_domainconnect	_domainconnect.gd.domaincontrol.com.	1 hour	
    SOA	@	Primary nameserver: ns39.domaincontrol.com.	600 seconds	
    MX	@	mail.redquants.com. (priority：10)	1 hour	
    TXT	@	v=spf1 mx -all	1/2 hour	
    TXT	s20220731942._domainkey	k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAy3kfeDoehTke8q5IzGditJiAyere7CuEfzaYcDkCz1Qp1AvyloKu5WIZ3aP1D2UakDPaRv1pwcYQ/hibd5RIEyFP72vcIkPMMvnQWpvBebONScQTPR0HXa3QT5Nj1731MRi3hdYU/XfYDTAN8zgFk1JQFe42ABbz4XzgtKm83M6cf7xB+vbajT33VqEdHN+rw42KUczRCcXnHooGE4MPF/1PA+uXeHn/w27gJCBjgORfnp7nZXs/LDb2oTwec8VTk/BbFhtwVKcOr6s0hoqp8IyQ/avEfPPIZjES4/dB+a/zWqqBUDrCgrX6yfJSRKyQdXECd9t0CkAoGJmbIVbRvwIDAQAB    1/2 hour
    TXT	_acme-challenge	i6JHwJtZMvuZbvqKhHKygGZbXBu3XajeN8C3HoRqWzU	1 hour		
    TXT	_dmarc	v=DMARC1; p=none; rua=mailto:dmarc-reports@redquants.com	1 hour	
    
# Bookmark

- <https://hub.docker.com/r/analogic/poste.io>
- <https://hub.docker.com/r/serverwentdown/file-manager>
- <https://github.com/serverwentdown/file-manager>
- <https://poste.io/>
- <https://poste.io/doc/>
- <https://andyyou.github.io/2019/04/13/how-to-use-certbot/>
- <https://docs.min.io/docs/generate-let-s-encypt-certificate-using-concert-for-minio.html>
- <https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/>
- <https://www.howtogeek.com/devops/how-to-setup-basic-http-authentication-on-nginx/>
- <https://ithelp.ithome.com.tw/articles/10267079>
- <https://www.cyberciti.biz/faq/how-to-delete-a-ufw-firewall-rule-on-ubuntu-debian-linux/>
- <https://www.markdownguide.org/basic-syntax/>
- <https://ithelp.ithome.com.tw/articles/10191016?sc=hot>
- <https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/>
- <https://linuxtechexpert.com/how-do-we-setup-ufw-firewall-on-ubuntu-and-debian/>
- <https://hub.docker.com/r/snsyzb/webssh>
- <https://github.com/huashengdun/webssh>
- <https://github.com/huashengdun/webssh/issues/96>
- <https://github.com/mjstealey/wordpress-nginx-docker/tree/master/ssl>
- <https://docs.docker.com/samples/wordpress/>
- <https://blog.csdn.net/taiyangdao/article/details/88844558>
- <https://github.com/urre/wordpress-nginx-docker-compose/blob/master/nginx/default.conf.conf>
- <https://www.tutorialworks.com/container-networking/>
- <https://zhuanlan.zhihu.com/p/382779160>
- <https://forums.docker.com/t/restricting-exposed-docker-ports-with-iptables/108075/3>
- <https://ithelp.ithome.com.tw/articles/10244444>
- <https://labs.play-with-docker.com/>
- <https://iottalk.vip/static/iottalk/08/index.html>
- <https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules>
- <https://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-sg-en-4/s1-fireall-ipt-act.html>

