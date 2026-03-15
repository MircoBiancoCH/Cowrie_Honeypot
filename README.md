Cowrie Honeypot: Deployment & Threat Intelligence Analysis
----------------------------------------------------------
Projektübersicht:
Dieses Projekt demonstriert die Einrichtung und den Betrieb eines interaktiven SSH- und Telnet-Honeypots (Cowrie) auf einem Ubuntu-Cloud-Server. Ziel des Projekts war es, automatisierte Angriffe aus dem Internet (Botnetze, Scanner) in Echtzeit zu erfassen, deren Reconnaissance-Skripte zu analysieren und hochgeladene Malware/Schlüssel sicher zu isolieren.

Technologien: Linux (Ubuntu), Docker, UFW (Uncomplicated Firewall), iptables (Port-Forwarding), Cowrie (Python-basierter Honeypot), Bash.
_______________________________________________________________________________________________________________________________________________

Architektur & Sicherheitskonzept:
Um den Host-Server zu schützen, wurde ein zweistufiges Sicherheitskonzept angewendet:

Versteckter Host-Zugang: Der echte SSH-Dienst des Servers wurde auf einen unauffälligen High-Port (55522) verlegt.

Isolierte Falle: Der Honeypot läuft in einem abgeriegelten Docker-Container. Eingehender Traffic auf den Standard-Ports (22, 23) wird über iptables unsichtbar an den Container weitergeleitet.
_______________________________________________________________________________________________________________________________________________

Installation & Setup:

1. Host-Absicherung & Firewall
  (Echten SSH-Port ändern und Dienst neu laden)
   
       sudo systemctl daemon-reload
   
       sudo systemctl restart ssh.socket

  (Host-Port erlauben und Honeypot-Ports (22, 23) öffnen)
   
       sudo ufw allow 55555/tcp
   
       sudo ufw allow 22/tcp
   
       sudo ufw allow 23/tcp

3. Cowrie Honeypot (Docker Deployment)
   
   Um Persistenz für Logs und gesammelte Malware zu gewährleisten, wurde ein dediziertes Docker Volume verwendet.

  (Volume erstellen)
   
      sudo docker volume create cowrie-var'

  (Container starten)
   
      sudo docker run -p 2222:2222 -p 2223:2223 \
   
      -v cowrie-var:/cowrie/cowrie-git/var \
   
      -d --name cowrie-honeypot cowrie/cowrie

3. Port-Forwarding (Die Falle stellen)
   
  Leitet den Traffic der Angreifer transparent in den Honeypot-Container:
  
      sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
    
      sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
    
      sudo apt install iptables-persistent
_______________________________________________________________________________________________________________________________________________

Operations & Monitoring (Cheatsheet):
Im operativen Betrieb wurden folgende Befehle zur Echtzeitanalyse genutzt:

Live-Logs verfolgen: 

    sudo docker logs -f cowrie-honeypot

Gezielt nach Logins filtern:

    sudo docker logs cowrie-honeypot | grep "login attempt"

Gefangene Malware prüfen:

    sudo ls -la /var/lib/docker/volumes/cowrie-var/_data/lib/cowrie/downloads/

Aufgezeichnete TTY-Sitzungen ansehen:

    sudo ls -la /var/lib/docker/volumes/cowrie-var/_data/lib/cowrie/tty/
_______________________________________________________________________________________________________________________________________________

Ergebnisse & Threat Intelligence
Innerhalb der ersten Stunden nach Deployment konnten bereits verschiedene automatisierte Angriffsmuster identifiziert werden:

Fund 1: Krypto-Miner Reconnaissance
Ein in Go geschriebener Bot (SSH-2.0-Go) loggte sich über Brute-Force (root:password) ein und führte ein massives Erkundungsskript aus.

Ziel: Identifikation von CPU-Kernen und dedizierten GPUs (grep -i nvidia), um die Rentabilität für Krypto-Mining zu prüfen.

(Füge hier gerne einen Screenshot deiner Logs mit dem grep -i nvidia Skript ein)


Fund 2: SSH-Key Injection (Persistenz-Versuch)
Ein weiteres Botnetz versuchte, den Admin auszusperren und eine Hintertür (Persistenz) einzurichten.

Methode: Löschen des .ssh-Ordners und Injektion eines eigenen Public Keys (ssh-rsa AAAA...mdrfckr).

Ergebnis: Der Honeypot simulierte den Erfolg (Command found), isolierte den Key jedoch sicher im downloads-Verzeichnis des Docker-Volumes.

(Füge hier gerne deinen Screenshot mit dem "mdrfckr"-Schlüssel ein)

_______________________________________________________________________________________________________________________________________________

Key Learnings
Verständnis für die Aggressivität und Geschwindigkeit (tlw. unter 1 Sekunde für einen kompletten System-Scan) moderner Botnetze.

Praktische Erfahrung im Umgang mit iptables NAT-Regeln und Docker-Volume-Management.

Analyse von kompromittierten System-Sitzungen ohne Gefährdung der eigenen Infrastruktur.
