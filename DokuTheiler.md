# Serverdienste automatisieren mit Vagrant

## Inhaltsverzeichnis
1. Kurzbeschrieb
2. Wissenstand vor der LB1
3. Implementierung
    * Sicherheit
    * Netzwerkplan
    * Vagrantfile
4. Testing
5. Jetziger Wissensstand
6. Fazit

## Kurzbeschrieb
Das Ziel für mich war es einen Server automatisch erstellen zu können, auf dem eine MySQL-Datenbannk, sowie ein Webserver mit phpMyAdmin laufen.
Dieser soll zusätzlich sicherheitstechnisch gesichert sein.

## Wissensstand vor der LB1
*Git* </br>
Git habe ich bereits im Geschäft als normaler Arbeitsbereich und fühle mich dort also ziemlich sicher.
Ich benutze die Eclipse IDE und pushe von dort aus meine Files mit einem einfache Klick.
*Vagrant* </br>
Vagrant kannte ich vor dem Modul noch nicht. Im Unterricht habe ich gelernt, dass man damit VMs automatisiert erstellen kann.
Wie genau dies abläuft mit den dem Erstellen eines Vagrantfiles und dessen Inhalt, musste ich mit dem Unterrichtsmaterial sowie dem Internet beschäftigen.
*Markdown* </br>
Von Markdown hatte ich noch nie etwas gehört. Der erste Berührungspunkt waren die Aufgaben im Unterricht des M300.
*Linux* </br>
Meine Linux-Kenntnisse sind noch ziemlich basisch.
Ich habe leider nicht so viel mit Linux zu tun, weshalb ich die gelernten Befehle schnell wieder vergesse.
*Virtualisierung* </br>
Bezüglich der Virtualisierung habe ich nicht viel Wissen. In der Woche vom 18. März hat ein üK mit dem Kernthema Virtualisierung begonnen, wodurch mein Wissen erst noch wachsen wird.

## Implementierung
### Sicherheit
Folgende Sicherheitsaspekte waren geplant:
1. Nicht-Root-Usererstellung
2. Firewall
3. Reverse-Proxy
4. HTTPS

Folgende Sicherheitsapsekte konnten umgesetzt werden:
1. Nicht-Root-Usererstellung
2. Firewall

*Kommentar*
Leider konnten nicht alle Sicherheitsaspekte implementiert werden. Das läuft zuschulden von zu wenig Wissen. Vieles funktionierte leider nicht, trotz Online-Tutorials und dem Unterrichtsmaterial.

### Netzwerkplan
![Image](/Netzwerkplan.PNG)

### Vagrantfile

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
~~~~

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

## Testing

|Testfall | Erwartet | Resultat |
|:--:|:--:|:--:|
|Webserver|Webserver kann im Browser abgerufen werden|Der Webserver konnte im Browser abgerufen werden|
|phpMyAdmin|Das phpMyAdmin Tool wird angezeigt|Das phpMyAdmin Tool wurde auf dem Webserver angezeigt|
|mySQL-Server|Der mySQL sollte laufen|mySQL ist betriebsbereit|
|Firewall|Die Firewall sollte aktiv sein und Port 80 und 22 offen halten|Firewall ist aktiv und Port 80 und 22 offen|
|User|Man kann sich mit dem erstellten nicht-Root-User einloggen|Ich konnte mich mit dem User einloggen|

## Jetziger Wissensstand
*Git*
Dieser Wissenstand hat sich nicht gross verändert, da ich bereits mit Git vertraut war.
*Vagrant*
Mein Wissen über Vagrant wurde deutlich vergrössert. Ich glaube aber, dass da noch Potential noch oben ist und ich erste die Spitze des Eisbergs gesehen habe.
*Markdown*
Ich konnte Markdown effizent nutzen und befinde es als gutes Dokumentiersystem.
*Linux*
Meine Linux-Kenntnisse haben sich nicht gross verändert, viel musste ich nachschauen.
*Virtualisierung*
Die Virtualisierung habe ich mit dieser LB nur praktisch und nicht in der Theorie kennengelernt.
Da bedarf es noch mehr Theorie um diese wirklich zu verstehen.

## Fazit
Auch wenn ich nicht genau alles erfüllen konnte was ich wollte, denke ich habe ich viel gelernt und kann dies auch im Arbeitsumfeld einsetzen, da wir auch gerade im Zuge der Virtualisierung sind.
Ich werde mich wahrscheinlich noch etwas mehr darüber informieren, da dieses Thema in Zukunft unausweichlich und wichtig ist. Ich finde die LB1 war ein guter Einstieg dazu.
