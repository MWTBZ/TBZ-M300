# M300 - LB01
## Einleitung
Dies ist die Dokumentation zur LB01 im Modul 300. Wir hatten den Auftrag mit Vagrant ein oder mehrere Service/s automatisch zu installieren.
Ich habe mir die Aufgabe gesetzt einen Sambaserver aufzusetzten und zusätzlich noch einen Apache als demonstration für einen Freigegebenen Ordner.  
Mit Linux haben wir schon viel gearbeitet. Deshalb habe ich in Linux schon gewisse Grundkenntnisse. Ich fande es es bisschen blöd, dass wir nicht mit VM Ware arbeiten konnten, da ich seit beginn der Lehre nur mit VM Ware meine VM mache. Jedoch sind die Virtualisierungssoftwares nicht allzu verschieden. Un dzum Glück funktioniert Virtualbox parallel mit VM Ware. Vagrant kannte ich vor diesem Modul noch nicht. Es hört sich daber sehr interessant an.

***
## Lernumgebung
### Git
In diesem Modul arbeiten wir mit Git. Die ganze Moduldokumentation mit den Anleitungen ist auf GitHub abgespeichert. Unsere Arbeit an der LB01 müssen wir ebenfalls auf GitHub/Gitlab abgeben. Mit Hilfe von GitBash kann man sich die Repositorys z.B. lokal abspeichern. 

#### Lokales Repository aktualisieren/herunterladen
```
git pull
```
#### Online Repository aktualisieren/hochladen
```
git add -A                              #Fügt alle Dateien im Ordner zum commiten hinzu
git commit -m "Kommentar"               #Den Upload Commiten
git push                                #Das Repository aktualisieren
```
### Vagrant
Mit Vagrant kann man in einem File mehrere VMs definieren, die dann automatisch durch Vagrant mit den angegebenen Parametern aufgesetzt werden. Es ist möglich Vagrant Shell-Commands und -Scripts mitzugeben, welche danach auf der VM ausgeführt werden.

### VirtualBox
Damit eine VM automatisch aufgesetzt werden kann, braucht man noch eine Virtualisierungssoftware. Da Vagrant nur VirtualBox voll unterstützt, werden wir VirtualBox verwenden.

### Linux
Die Vagrant-VM wird mit einer Linuxbox aufgesetzt.
***

## Projekt Dokumentation
### Idee
Meine Idee war, dass ich eine VM erstelle und auf der **Samba** installieren. Ich werde mit Samba zwei Ordner freigeben. Der Erste wird der HTML-Folder eines **Apache** Webservers sein, in dem man die Index.html Datei bearbeiten kann.
Ein zweiter Ordner wird erstellt, der als Fileshare dient. Dieser wird auch über Samba freigegeben.
Für die Berechtigungen werden zwei Benutzer erstellet, welche je nur auf einen der beiden Ordner Zugriffsrechte haben.

### Netzwerkplan
![](/Images/Netzwerkplan.png)  

### Vagrant-File
Das Vagrant-File ist das Zentrum der Automation. Hier wird alles festgelegt; Jede Konfiguration, alle Einstellungen, die auszuführenden Befehlen etc.

Mit dem Befehl `Vagrant.configure` wird die ganze Vagrantkonfiguration festgelegt. Jede VM wird mit `config.vm.define` festgelegt. Darin wird muss die Box festgelegt werden, die benutzt wird um die Vm zu erstellen. Es können auch andere Sachen, wie z.B. die IP festgelegt werden.  
Wenn man die Virtualisierungsoftware spezifisch festlegt, kann man die VM-Eigenschaften wie den Arbeitsspeicher oder Name festlegen.

```Ruby
Vagrant.configure("2") do |config|
  config.vm.define "samba" do |smb|
    smb.vm.box = "ubuntu/xenial64"
    smb.vm.network "public_network", ip: "10.71.13.20"

    smb.vm.provider "virtualbox" do |smb|
      smb.memory = "1024"
      smb.name = "Samba-LB01"
    end

    smb.vm.hostname = "Samba-LB01"
  end
end 
```
Um Shell-Befehle auszuführen braucht man folgenden Befehl. Darin können normal Linux-Befehle ausgeführt werden, wie z.B. `apt-get install -y apache2`. Die nachfolgenden Code-Snippets sind alle hier eingetragen.
```Ruby
config.vm.provision "shell", inline: <<-SHELL
    apt-get install -y apache2
SHELL
```

#### Firewall
Da ich meine VM in einem öffentlichen Netz habe, brauche ich eine Firewall. Damit der Webserver trotzdem noch erreichbar sein muss, erstelle ich eine Regel welche den Port 80 öffnet. Für Samba muss ebenfalls der Port 445 geöffnet werden.  
Durch das `-y` kann die Benutzereingabe mitgegeben werden.
```
sudo apt-get install ufw
sudo ufw --force enable
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 445/tcp
```
#### Apache Webserver
Um den Samba Service zu demonstrieren, habe ich Apache installiert.
```
apt-get install -y apache2
```
Mit Hilfe des `echo`-Command kann man den Inhalt eines Files überschreiben / erweitern.
```
echo "
<html>
  <head>
      <title>Vagrant</title>
  </head>
  <body>
      <h1>Vagrant Autoinstallation</h1>
      <br>
      <p>Diese VM/Webseite wurde mit Vagrant automatisch erstellt.</p>
  </body>

</html>
" > /var/www/html/index.html          # ">" überschreibt den Inhalt der Datei / ">>" fügt es hinzu
```

#### Samba
Bevor Samba installiert wird, erstelle ich zwei Benutzer die dann für Samba berechtigt werden.
```
sudo useradd samba --shell /bin/false                     #Add User "samba"
echo Test | passwd samba --stdin                          # Set Password of user "samba"

sudo useradd samba2 --shell /bin/false                    #Add User "samba2"
echo Test2 | passwd samba2 --stdin                        # Set Password of user "samba2"
```
Nun wird Samba installiert und für beide Benutzer wird noch ein Samba-Passwort erstellt. 
```
sudo apt-get install -y samba
(echo Test; echo Test) | smbpasswd -s -a samba          #Define samba-Password for User "samba"
(echo Test2; echo Test2) | smbpasswd -s -a samba2       #Define samba-Password for User "samba2"
```
Um die Rechtevergabe zu zeigen, wird noch ein zweiter Ordner erstellt, welcher dann per Samba freigegeben wird.
```
sudo mkdir /var/FileShare
```
Damit über die Freigabe die Webseite bearbeitet werden kann, muss der Ordner nch freigegeben werden. Dies macht man in der `/etc/samba/smb.conf`. Der [ShareName] ist der name der Freigabe. Auf diese kann man über folgenden Pfad zugreifen: `\\<ip-adress>\<ShareName>`.  
Mit `valid user = samba` wird festgelegt, dass nur der User "samba" darauf zugreifen kann
```
sudo echo "  
    # HTML-Share
    [HTMLShare]
        comment = Foldershare for Apache
        path = /var/www/html
        read only = no
        guest ok = no
        writeable = yes
        valid users = samba 

" >> /etc/samba/smb.conf
```

Das gleiche Spiel ist mit dem Fileshare-Ordner.
```
sudo echo "
    # File-Share
    [FileShare]
        comment = Foldershare for Files
        path = /var/FileShare
        read only = no
        guest ok = no
        writeable = yes
        valid users = samba2
          
" >> /etc/samba/smb.conf
```
Jetzt setzt man noch die lokalen Berechtigungen.
> 0 – no permission  
> 1 – execute  
> 2 – write  
> 3 – write and execute  
> 4 – read  
> 5 – read and execute  
> 6 – read and write  
> 7 – read, write, and execute<br>

Die drei Zahlen zeigen die drei Klassen: Owner, Group, Others
```
sudo chmod 777 /var/www/html
sudo chmod 777 /var/FileShare
```
Zum Schluss muss der Samba-Service noch neugestartet werden.
```
sudo service smbd restart
```

***
## Testing
Um zu testen, ob alles richtig installiert ist,hab ich folgende Tabelle benützt.

| Service | Testfall | Beschreibung | Resultat |
|:--:|:--:|:--|:--|
| Webserver | Webseite erreichbar | Browser öffnen<br>**http://10.71.13.20** | Webseite kann aufgerufen werden<br>*Siehe Bild 2* |
| Samba | Samba erreichbar | Explorer öffnen<br> **\\\10.71.13.20** | Freigegebene Ordner<br>werden angezeigt<br>*Siehe Bild 1* |
| Samba | Berechtigung<br>HTMLShare | Windows Explorer öffnen<br> **\\\10.71.13.20\HTMLShare**<br>Nur Benutzer "samba" ist berechtigt | Nur Benutzer "samba" hat Zugriffsrechte |
| Samba | Berechtigung<br>FileShare | Windows Explorer öffnen<br> **\\\10.71.13.20\FileShare**<br>Nur Benutzer "samba2" ist berechtigt | Nur Benutzer "samba2" hat Zugriffsrechte |
| Firewall | Firewall aktiviert | Firewallstatus überprüfen<br>per SSH verbinden<br>`sudo ufw status` | Die Firewall ist aktiv<br> *Firewall enabled* |
| Firewall | Port 80 ist offen | Browser öffnen<br>**http://10.71.13.20** | Webseite kann aufgerufen werden |
| Firewall | Port 445 ist offen | Explorer öffnen<br> **\\\10.71.13.20** | Freigegebene Ordner<br>werden angezeigt |

**Samba Freigabe**  
![](\Images\SambaShare.png)  
**Webserver**  
![](\Images\Webserver.png)


***
## Fazit / Reflexion
Zu Beginn des Moduls war es ein wenig viel aufs Mal. Doch durch das repetieren, wurde vieles klar. Beim erarbeiten der LB01 ging alles recht gut voran. Bei der Konfiguration von Samba gab es das Problem, das man nicht auf die Freigabe zugreifen konnte, wenn die VM im NAT war.   
Ich musste also der VM die vorgegebene IP geben und ins öffentliche Netz stellen, damit man Samba nutzen konnte. Es gibt nur noch ein Problem, und zwar wenn man sich über denn Explorer in die Freigabe begibt und sich mit dem ersten User anmeldet. Kann man sich nicht mehr mit dem zweiten User anmelden. Windows erkennt wenn man sich auf dem gleichen Samba-Server mit mehreren Benutzern anmelden will und verhindert es. Erst nach einem Neustart kann man sich mit dem zweiten User wieder anmelden.

***
***