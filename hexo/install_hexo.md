
# [install hexo on CentOS7](https://www.vultr.com/docs/install-hexo-on-centos-7)
for a freshly installed CentOS7 system

## before install hexo
before start with hexo, we need get our server ready, just follow the steps below

### most important -- to avoid some unknown errors
```
yum update -y
```

### install basic pkg
```
yum install -y gcc gcc-c++ make git openssl
```

### install oh-my-zsh -- optional
```
yum install -y zsh
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

### install stable nginx

#### define nginx repo -- some may named yum.repo.d
** caution: the last EOF must at the beginning of a new line and take care of the '&' keywords
```
cat > /etc/yum.repos.d/nginx.repo <<EOF
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/\$basearch/
gpgcheck=0
enabled=1

EOF
```

#### install nginx
```
yum install -y nginx
```

#### set as auto start with system load and start now
```
systemctl enable nginx && systemctl start nginx
```

### install node.js -- hexo is based on nodejs
```
yum install -y nodejs
```

### start with safey
usually we create a new user who has lower permission than root to avoid building an unsafe server
let's create a new user named `u_hexo` and add it to group `nginx`
```
useradd u_hexo -d /home/u_hexo/ -m -r -U -s /bin/bash
passwd u_hexo
usermod -aG nginx u_hexo
visudo
    u_hexo  ALL=(ALL)       ALL
```

### generate a SSH key pair -- optional
click [here](https://en.wikipedia.org/wiki/SSH) to learn more about SSH(Secure Shell)

## deal with hexo
now it's time to get hexo ready

### login as u_hexo the user we just create
```
su u_hexo
```

### install hexo -- sudo is needed
there may come up with some warnnings when finshing the install, just ignore them
```
mkdir -p ~/zajitangzhai.me/hexo
cd ~/zajitangzhai.me/hexo
sudo npm install -g hexo-cli hexo-server
```

### init hexo
with _config.yml you need to update with your server address
```
hexo init && npm install --save
vi _config.yml
hexo g
```

### set up nginx
```
chown -R u_hexo:nginx ~/zajitangzhai.me/

cat > ~/zajitangzhai.me/hexo/hexo_nginx.conf <<EOF
server {
    listen 80;
    listen [::]:80;
    ## if https is desired, please uncomment the following lines
    #listen 443 ssl http2;
    #listen [::]:443 ssl http2;

    server_name zajitangzhai.me;

    ## if forcing https, please uncomment the following lines
    #if (\$scheme = http) {
    #    return 301 https://\$server_name\$request_uri;
    #}

    location / {
        root /home/u_hexo/zajitangzhai.me/hexo/public;
        index index.html;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header Host \$http_host;
        ## if https is desired, please uncomment the following lines
        #proxy_set_header X-Forwarded-Proto https;
    }
}
EOF

sudo ln -sf ~/zajitangzhai.me/hexo/hexo_nginx.conf /etc/nginx/conf.d/
sudo systemctl restart nginx
```

# to be continued...





