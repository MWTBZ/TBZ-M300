# LB01
## Einleitung
Dies ist die Dokumentation zur LB01 im Modul 300. Wir hatten den Auftrag mit Vagrant ein oder mehrere Service/s automatisch zu installieren.
Ich habe mir die Aufgabe gesetzt einen Sambaserver aufzusetzten und zusätzlich noch einen Apache als demonstration für einen Freigegebenen Ordner.
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

***

## Projekt Dokumentation
### Idee
Meine Idee war, dass ich eine VM erstelle und auf der **Samba** installieren. Ich werde mit Samba zwei Ordner freigeben. Der Erste wird der HTML-Folder eines **Apache** Webservers sein, in dem man die Index.html Datei bearbeiten kann.
Ein zweiter Ordner wird erstellt, der als Fileshare dient. Dieser wird auch über Samba freigegeben.
Für die Berechtigungen werden zwei Benutzer erstellet, welche je nur auf einen der beiden Ordner Zugriffsrechte haben.

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
Um Shell-Befehle auszuführen braucht man folgenden Befehl. Darin können normal Linux-Befehle ausgeführt werden, wie z.B. `apt-get install -y apache2`.
```Ruby
config.vm.provision "shell", inline: <<-SHELL
    apt-get install -y apache2
SHELL
```
