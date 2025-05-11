# Driftingblues7 - HackMyVM (Easy)

![Driftingblues7.png](Driftingblues7.png)

## Übersicht

*   **VM:** Driftingblues7
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Driftingblues7)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 16. April 2023
*   **Original-Writeup:** https://alientec1908.github.io/Driftingblues7_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Easy"-Challenge war es, Root-Zugriff auf der Maschine "Driftingblues7" zu erlangen. Die Enumeration deckte eine Vielzahl offener Ports auf, darunter SSH (22), HTTP (66 - Python SimpleHTTPServer), HTTP (80 - Apache, Redirect auf HTTPS), HTTPS (443 - Apache, EyesOfNetwork), RPCBind (111), MySQL/MariaDB (3306) und InfluxDB (8086). Der entscheidende Fund war der unsichere Python SimpleHTTPServer auf Port 66. Ein `gobuster`-Scan auf diesen Port enthüllte direkten Zugriff auf `user.txt` und `root.txt`, wodurch beide Flags ohne weitere Eskalation gelesen werden konnten. Weitere Untersuchungen auf Port 66 führten zum Fund einer Base64-kodierten, passwortgeschützten ZIP-Datei (`/eon`). Das Passwort (`killah`) wurde mit `zip2john` und `john` geknackt. Die extrahierte `creds.txt` enthielt Zugangsdaten (`admin:isitreal31__`) für die EyesOfNetwork-Anwendung auf Port 443. Ein Versuch, eine bekannte RCE-Schwachstelle in EyesOfNetwork auszunutzen, scheiterte jedoch.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `curl`
*   `nikto`
*   `wfuzz`
*   `sqlmap` (Versuch)
*   `gobuster`
*   `base64` (Decoder)
*   `unzip`
*   `zip2john`
*   `john` (John the Ripper)
*   `ssh` (Versuch)
*   `snmpwalk` (Versuch)
*   `python3` (Exploit-Skript für EyesOfNetwork, Versuch)
*   Standard Linux-Befehle (`ip`, `vi`, `cat`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Driftingblues7" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration (Port 80/443):**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.114`, Hostname `driftingblues7.hmv` oder `cve.hmv`).
    *   `nmap`-Scan identifizierte SSH (22), HTTP (66 - Python SimpleHTTPServer), HTTP (80 - Apache, Redirect zu 443), HTTPS (443 - Apache, EyesOfNetwork), RPCBind (111), MySQL/MariaDB (3306) und InfluxDB (8086).
    *   Untersuchungen auf Port 80/443 (EyesOfNetwork) mit `nikto`, `wfuzz` (LFI-Versuch) und `sqlmap` (SQLi-Versuch) zeigten veraltete Software und fehlende Header, aber keine direkten Erfolge für den Initial Access.

2.  **Direkter Flag-Fund (Web Enumeration Port 66):**
    *   Ein `gobuster`-Scan auf den Python SimpleHTTPServer (Port 66) enthüllte direkt im Web-Root die Dateien `user.txt` und `root.txt`.
    *   Durch direkten Abruf dieser Dateien (z.B. mit `curl`) wurden beide Flags (`AED5...` und `BD22...`) ohne weitere Exploitation oder Eskalation erlangt.

3.  **Weitere Enumeration & Credential Discovery (Port 66 - `/eon`):**
    *   (Obwohl die Flags bereits gefunden waren, wurde die Enumeration fortgesetzt.)
    *   Auf Port 66 wurde unter dem Pfad `/eon` eine Base64-kodierte Zeichenkette gefunden.
    *   Die Dekodierung ergab eine passwortgeschützte ZIP-Datei.
    *   Mit `zip2john` wurde der Hash des ZIP-Passworts extrahiert und mit `john` und `rockyou.txt` zu `killah` geknackt.
    *   Die entpackte ZIP-Datei enthielt `creds.txt` mit den Zugangsdaten `admin:isitreal31__`.

4.  **Initial Access Versuch (EyesOfNetwork auf Port 443):**
    *   Die Credentials `admin:isitreal31__` wurden verwendet, um sich erfolgreich bei der EyesOfNetwork-Webanwendung (Port 443) anzumelden.
    *   Ein SSH-Login-Versuch mit diesen Credentials scheiterte.
    *   Ein Versuch, eine bekannte RCE-Schwachstelle in EyesOfNetwork 5.3 mit einem Python-Exploit auszunutzen, schlug ebenfalls fehl.

## Wichtige Schwachstellen und Konzepte

*   **Unsicherer Python SimpleHTTPServer (Port 66):** Der Dienst exponierte sensible Dateien (`user.txt`, `root.txt`, Base64-kodierte ZIP-Datei) direkt im Web-Root. Dies war die Hauptschwachstelle.
*   **Information Disclosure:** Flags und eine passwortgeschützte ZIP-Datei mit Credentials waren über den unsicheren Webserver zugänglich.
*   **Schwaches ZIP-Passwort:** Das Passwort `killah` für die ZIP-Datei konnte leicht geknackt werden.
*   **Veraltete Software:** Auf den Ports 80/443 liefen veraltete Versionen von Apache, OpenSSL und PHP.
*   **Offene Dienste ohne Authentifizierung (teilweise):** Die InfluxDB-API auf Port 8086 war ohne Authentifizierung zugänglich und erlaubte das Auflisten von Datenbanken.

## Flags

*   **User Flag (gefunden auf `http://driftingblues7.hmv:66/user.txt`):** `AED508ABE3D1D1303E1C1BC5F1C1BA2B`
*   **Root Flag (gefunden auf `http://driftingblues7.hmv:66/root.txt`):** `BD221F968ACB7E069FC7DDE713995C77`

## Tags

`HackMyVM`, `Driftingblues7`, `Easy`, `Web`, `Python SimpleHTTPServer`, `Information Disclosure`, `File Exposure`, `Apache`, `EyesOfNetwork`, `zip2john`, `John the Ripper`, `Base64`, `InfluxDB`, `Linux`
