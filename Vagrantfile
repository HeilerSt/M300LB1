Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vm.box = "ubuntu/trusty64"
  config.vm.network :forwarded_port, host: 8080, guest: 80
  
config.vm.provision "shell", inline: <<-SHELL
  cat > /etc/apt/sources.list.d/multiverse.list << EOF
deb http://archive.ubuntu.com/ubuntu trusty multiverse
deb http://archive.ubuntu.com/ubuntu trusty-updates multiverse
deb http://security.ubuntu.com/ubuntu trusty-security multiverse
EOF

apt-get update

apt-get install -y apache2 libapache2-mod-fastcgi apache2-mpm-worker

rm -rf /var/www
ln -fs /vagrant /var/www

echo "ServerName localhost" > /etc/apache2/httpd.conf

a2enmod actions fastcgi rewrite
service apache2 reload

apt-get install -y php5 php5-cli php5-fpm curl php5-curl php5-mcrypt php5-xdebug

cat > /etc/apache2/conf-available/php5-fpm.conf << EOF
<IfModule mod_fastcgi.c>
    AddHandler php5-fcgi .php
    Action php5-fcgi /php5-fcgi
    Alias /php5-fcgi /usr/lib/cgi-bin/php5-fcgi
    FastCgiExternalServer /usr/lib/cgi-bin/php5-fcgi -socket /var/run/php5-fpm.sock -pass-header Authorization
    <Directory /usr/lib/cgi-bin>
        Require all granted
    </Directory>
</IfModule>
EOF

php5enmod mcrypt

a2enconf php5-fpm
service apache2 reload

debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'

apt-get install -y mysql-server mysql-client php5-mysql

debconf-set-selections <<< 'phpmyadmin phpmyadmin/dbconfig-install boolean true'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/app-password-confirm password root'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/mysql/admin-pass password root'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/mysql/app-pass password root'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2'

apt-get install -y phpmyadmin

ln -s /etc/phpmyadmin/apache.conf /etc/apache2/sites-enabled/phpmyadmin.conf

sudo a2ensite default-ssl.conf
sudo a2enmod ssl
sudo a2dissite 000-default.conf 

service apache2 restart

adduser "view"
echo 'view:pass' | chpasswd

sudo ufw --force enable
sudo ufw deny out to any
sudo ufw allow out 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow ssh

SHELL

end
