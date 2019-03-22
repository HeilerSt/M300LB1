1. Vagrantfile

Die einzelnen Code Abschnitte des Vagrantfiles.

Erstellung eines http-Ubuntu-Servers.
~~~~
Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vm.box = "ubuntu/trusty64"
  config.vm.network :forwarded_port, host: 8080, guest: 80
~~~~

Updates herunerladen.
~~~~
apt-get update
~~~~

Einen Apache-Serverdienst installieren.
~~~~
apt-get install -y apache2 libapache2-mod-fastcgi apache2-mpm-worker
~~~~

Vagrant-Verzeichnis zum Apache Public Verzeichnis verlinken
~~~~
rm -rf /var/www
ln -fs /vagrant /var/www
~~~~

Den Servernamen dem hhtpd.conf-File hinzufügen
~~~~
echo "ServerName localhost" > /etc/apache2/httpd.conf
~~~~

Module installieren, damit Apache funktioniert.
~~~~
a2enmod actions fastcgi rewrite
service apache2 reload

php-Pakete installieren.
~~~~
apt-get install -y php5 php5-cli php5-fpm curl php5-curl php5-mcrypt php5-xdebug
~~~~

Konfigurationen in Apache erstellen.
~~~~
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
~~~~

php-Module aktivieren.
~~~~
php5enmod mcrypt
~~~~

Änderungen auch im Apache triggern, damit dies up to date ist.
~~~~
a2enconf php5-fpm
service apache2 reload
~~~~

Für den MySQL-Server einen Root-User erstellen.
~~~~
debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
~~~~

MySQL-Pakete installieren.
~~~~
apt-get install -y mysql-server mysql-client php5-mysql
~~~~

Default Settings in phpMyAdmin einstellen.
~~~~
debconf-set-selections <<< 'phpmyadmin phpmyadmin/dbconfig-install boolean true'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/app-password-confirm password root'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/mysql/admin-pass password root'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/mysql/app-pass password root'
debconf-set-selections <<< 'phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2'
~~~~

phpMyAdmin-Pakete installieren.
~~~~
apt-get install -y phpmyadmin
~~~~

Restart durchführen um die Änderungen wirksam zu machen.
~~~~
service apache2 restart
~~~~

Einen neuen User hinzufügen, damit nicht immer der Root-User gebraucht wird, denn dies ist ziemlich riskant.
~~~~
adduser "view"
echo 'view:pass' | chpasswd
~~~~

Die UFW-Firewall einschalten und als erstes alle Ports nach aussen sperren.
Danach habe ich Port 22 und 80 nach draussen wieder geöffnet.
Den ssh-Port habe ich offen gelassen um noch drauf zu greifen zu können.
~~~~
sudo ufw --force enable
sudo ufw deny out to any
sudo ufw allow out 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow ssh
~~~~
