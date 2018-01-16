---
title: Deserialisierung von Java Objekten
layout: post
permalink: deserialisierung-von-java-objekten
published: true
---
Letzten Freitag (6.11.2015) wurde ein Artikel auf FoxGloveSecurity[^n] veroeffentlich der das Problem der Java Deserialisierung aufgreift. Dieses Problem wurde bereits auf der AppSecCali in dem Talk "Marshalling Pickles"[^n] beschrieben und in einem Artikel von IBM 2013 thematisiert[^n]. FoxGloveSecurity beschreibt eine Moeglichkeit um Mithilfe von Deserialisierung von Java Objekten fremden Programmcode auszufuehren in bekannter Software auszufuehren.
Dies ist aber nicht vollstaendig. Hier wird sich nur auf eine Library (die commons-collection) beschraenkt, welche vorgeschlagen wird zu patchen. Die Commons-Collection ist aber an dieser Stelle nicht das Problem, sondern nur ein Mittel zur Ausfuehrung. Patchen der Commons wird nicht das Problem der Deserialisierung von nicht vertrauenswuerdigen Objekten ("deserialization of untrusted data": CWE-502)[^n] beheben. Damit reiht sich Java Serialisation in eine laengere Liste von XML, Yaml und anderen Formaten ein.

Das Serialisierungsformat beschreibt den Aufbau des Objekts und dessen Daten, die bei Wiederherstellung in das Objekt geschrieben werden. Das passiert nun waehrend der String gelesen wird und der Software wird keine Moeglichkeit geboten das Objekt zu verifizieren. Beliebige Objekte, welche lediglich serialisierbar sein muessen, koennen in die Softwareumgebung geladen werden.

# Identifikation
Objekte werden nur deserialisiert wenn die Klasse, von der das Objekt abstammt im Classpath liegt und die angreifbare Klasse serialisierbar ist.
Durch Codeanalysen kann festgestellt werden, ob die Methode ObjectInputStream.readObject, und davon abgeleitete Methoden in der Software zum Einsatz kommen.
Natuerlich kann dieses Problem auch durch einen Blackboxtest erkannt werden, wobei dazu das Burp Plugin von DirectDefense "Superserial"[^n] helfen kann um alle Stellen herauszufinden die mit serialisierten Java Objekten kommunizieren.

# Abwehr
Wie wir oben schon beschrieben haben, kann die Software das Objekt nicht verifizieren, wenn es wieder Zusammengebaut wird. Dazu gibt es eine Art "Look-Ahead"[^n] Variante um Objekte zu deserialisieren. Durch dieses Look-Ahead koennen bestimmte Objekte ausgeschlossen werden. Dadurch betreibt man eine Art Whitelisting und Umgeht das Problem, indem nur auf wenige bekannte und ueberpruefte Objekte zugegriffen werden darf.
Um diese Validierung durchzufuehren gibt es mehrere Tools, die man als SoftwareEntwickler nutzen kann.

  * [wsargent/paranoid-java-serialization]( https://github.com/wsargent/paranoid-java-serialization/): Ueberschreibt ObjectInputStream mit einer sicheren Variante. paranoid-java-serialization bietet JVM Level Kontroller ueber Serialisierte Objekte.

  * [ikkisoft/SerialKiller](https://github.com/ikkisoft/SerialKiller): SerialKiller validiert Java Klassen waehrend der Namensaufloesung und erlaubt White- und Blacklisting.

  * [kantega/NotSoSerial](https://github.com/kantega/notsoserial): Ein Java Agent der als Art Firewall wirkt. Es veraendert waehrend des Ladens einige bekannter angreiferbarer Klassen den Byte Code durch hinzufuegen (oder veraendern) einer readObject Methode, damit diese eine UnsupportedOperationException bei deserialisierung wirft.

Die andere, und etwas radikalere Moeglichkeit, waere ein komplettes deaktiveren bzw. firewallen von Diensten, die nicht-vertrauenswuerdige Daten zulassen.[^n]

Auch kann ein Wechsel zu einem weniger maechtigen Serialisierungsformat, wie JSON, das Problem beheben, wenn man weiterhin Kommunikation auf offenen Ports mit Serialisierten Objekten betreiben moechte. Dies setzt allerdings vorraus, dass alle Objekte in JSON wiedergegeben werden koennen.

[^n]: http://foxglovesecurity.com/2015/11/06/what-do-weblogic-websphere-jboss-jenkins-opennms-and-your-application-have-in-common-this-vulnerability/
[^n]: https://frohoff.github.io/appseccali-marshalling-pickles/
[^n]: http://www.ibm.com/developerworks/library/se-lookahead/
[^n]: https://cwe.mitre.org/data/definitions/502.html
[^n]: https://www.directdefense.com/superserial-java-deserialization-burp-extension/
[^n]: http://www.ibm.com/developerworks/library/se-lookahead/
[^n]: https://tersesystems.com/2015/11/08/closing-the-open-door-of-java-object-serialization/