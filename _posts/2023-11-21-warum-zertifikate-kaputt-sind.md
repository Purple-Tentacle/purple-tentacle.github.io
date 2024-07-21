---
layout: post
title: "Warum Zertifikate kaputt sind"
date: 2023-11-21 00:00:00 +0200
categories: IT-security
---
## Einführung

Zuerst eine kurze Erklärung für alle, die nicht so technisch unterwegs sind. Fast jeder nutzt täglich Zertifikate. Auch wenn die allermeisten sich dessen gar nicht bewusst sind. Aber jeder Aufruf der Google Internetseite ist mittlerweile per TLS verschlüsselt, erkennbar am https:// in der Adresszeile. Das gilt heutzutage für fast alle Internetseiten. Aber auch außerhalb des World Wide Web werden Zertifikate häufig eingesetzt.

## Vorteile von Zertifikaten
Ich als Benutzer solcher mit TLS abgesicherten Seiten habe einige Vorteile. Der Datenkanal zwischen mir und dem Server auf dem die Seite liegt ist verschlüsselt. Es kann also niemand Drittes die Daten lesen. (Zumindest so lange der private Schlüssel des Seitenanbieters geheim bleibt). Außerdem kann ich sicher sein, dass ich auch wirklich mit Google kommuniziere und nicht mit jemanden der das nur vogibt.

## Beispiel
Wie wird das gelöst? Na logisch, mit Zertifikaten. Hier mal ein Ausschnitt des LinkedIn Zertifikats.

![zertifikat](/assets/images/2023-11-21/zertifikat.png)

Dort ist erst einmal beschrieben, wofür das Zertifikat verwendet werden darf, nämlich zum Identitätsnachweis. Das Zertifikat ist sozusagen der Personalausweis der LinkedIn Seite.

Dann gibt es noch eine Information darüber, an wen das Zertifikat ausgestellt wurde, nämlich an www.linkedin.com. Schließlich folgen noch Informationen über den Herausgeber und den Gültigkeitszeitraum des Zertifikats.

Klingt doch erst einmal alles super und logisch. Bis man mal näher drüber nachdenkt. Bleiben wir bei der Analogie des Personalausweises. Was macht diesen vertrauenswürdig? Warum wird dieser bei einer Polizeikontrolle akzeptiert? Da er von einer vertrauenswürdigen Stelle ausgegeben wurde, der Personalausweisbehörde des Hauptwohnsitzes. Gedruckt wird er dann von der Bundesdruckerei, welche ebenfalls als vertrauenswürdig angesehen wird.

## Herausgeber
Auch bei Zertifikaten gibt es einen Herausgeber, er ist ja oben im Screenshot schon aufgelistet (DigiCert). Es gibt sogar eine genaue Aufschlüsselung.

![herausgeber](/assets/images/2023-11-21/herausgeber.png)

Es gibt also einen kleinen Pfad von Herausgebern. Ganz oben steht DigiCert. Das ist die Wurzel des Pfades (Fachbegriff Root Certification Authority). Auch diese hat wieder ein Zertifikat, schauen wir uns das mal genauer an.

![herausgeber2](/assets/images/2023-11-21/herausgeber2.png)

Neben einer viel umfangreicheren Liste an Zwecken und einer viel längeren Laufzeit fällt vor allem eines auf. Das Zertifikat wurde von “DigiCert Global Root CA” an “DigiCert Global Root CA” ausgegeben. Das wäre ja so, als ob ich mir selbst meinen Personalausweis ausstellen würde. Ich habe es nicht probiert, habe aber starke Zweifel an einer problemlosen Polizeikontrolle mit so einem “Ausweis”.

## Technisches Vertrauen
Warum stellt sich DigiCert selbst ein Zertifikat aus? Naja, irgendwo muss der Baum ja anfangen. Über der Wurzel ist halt nichts mehr, was das Zertifikat ausgeben könnte. Klingt aber trotzdem nicht sehr vertrauenswürdig. Rein technisch ist das auch nur vertrauenswürdig, weil das Root CA Zertifikat so eingestuft wurde. Nämlich an zwei Stellen. Einmal in Windows und einmal im Browser.

![browsertrust](/assets/images/2023-11-21/browsertrust.png)

Neben vielen anderen Zertifikaten findet sich in Windows auch das DigiCert Root CA Zertifikat wieder.

Auch im Google Chrome gibt es eine Liste mit als vertrauenswürdig eingestuften Root CA Zertifikaten.

https://docs.google.com/spreadsheets/d/e/2PACX-1vQ7Jtb4NxCSaEtCaisz2u3NQZcHejDUjI3Q-utBnL-C5E7w4crv6QZ9GRDb2bFGbLgUQsgQyF0Y8eoN/pubhtml

Auch dort findet sich unsere DigiCert Global Root CA wieder.

Das Besondere ist nun, dass durch die Einstufung als vertrauenswürdig auch alle weiteren Zertifikate im Baum als vertrauenswürdig eingestuft werden. Also wenn andere Webseitenbetreiber sich auch ein Zertifikat von DigiCert besorgen, vertrauen Google Chrome und Windows diesen ebenso.

So eine Root CA ist also extrem sensibel. Wird der private Schlüssel kompromittiert, müssen unter Umständen hunderttausende Zertifikate als ungültig eingestuft werden. Mehr dazu später.

## Organisatorisches Vertrauen
Neben dem technischen Vertrauen muss man dem Herausgeber eines Zertifikats auch organisatorisch vertrauen. Es würde ja nichts bringen, wenn sich einfach jeder ein Zertifikat für www.linkedin.com holen könnte um damit eine Phishing Seite zu betreiben. Der Anbieter muss also kontrollieren, dass ein Anspruch auf so ein Zertifikat besteht. Das kann man technisch lösen, zum Beispiel indem man einen bestimmten TXT Eintrag in den DNS Einstellungen von linkedin.com machen muss. Damit habe ich nachgewiesen, dass mir die Domain linkedin.com gehört, ich bin also anspruchsberechtigt.

## Der Fall Symantec
Symantec hat auch lange eine eigene Zertifizierungsstelle betrieben. Diese war auch allgemein als vertrauenswürdig eingestuft. Irgendwann gab es aber Auffälligkeiten. So wurden Zertifikate ausgegeben, ohne dass die Antragsteller dafür berechtigt waren. Sowas darf auf keinen Fall passieren, da somit das ganze Vertrauenssystem von Zertifikaten zerstört wird. Folgerichtig haben Browser- und Betriebssystemhersteller Symantec das Vertrauen entzogen. Damit wurden dann auch alle Zertifikate von Symantec ungültig. Also alles toll, das System funktioniert? Nicht wirklich. Denn es hat lange gedauert, bis das wirklich umgesetzt wurde. Zu viele populäre Webseiten nutzten noch Zertifikate von Symantec und man wollte die Benutzer nicht verunsichern.

## Zurückgezogene Zertifikate
Es gibt aber auch kleinere, alltäglich Fälle, bei denen Zertifikate zurückgezogen werden müssen. Stellen wir uns einfach einen Webserver vor, auf der der private Schlüssel des Zertifikats liegt. Dieser ist von Crackern angegriffen worden, der private Schlüssel ist vielleicht in unbefugte Hänge gelangt. Damit könnte man einen gefälschten Webserver betrieben und der Benutzer hätte keine Chance diesen zu erkennen.

Dazu kann man Zertifikate auch zurückziehen. Man meldet das also der Zertifizierungsstelle, die dieses Zertifikat auf eine Liste mit gesperrten Zertifikaten setzt.

Wie komme ich aber an diese Liste? Ganz einfach, der Link zu der Liste steht im Zertifikat.

![crllink](/assets/images/2023-11-21/crllink.png)

Und so sieht die Liste dann aus.

![crl](/assets/images/2023-11-21/crl.png)

Besonders lang ist die Liste ja nicht, nur 3 Zertifikate. Stichprobenartig habe ich mir mal die Sperrlisten anderer großer Zertifizierungsstellen angeschaut. Alle sind nicht besonders umfangreich, enthalten teilweise kein einziges gesperrtes Zertifikat. Ich kann mir wirklich nicht vorstellen, dass es nur so wenige Fälle gibt, bei denen private Schlüssel kompromittiert wurden. Ich gehe von einer großen Meldelücke aus.

## Das Problem mit Zertifikaten
Die gesamte Zertifikatsinfrastruktur ist dezentral aufgebaut. Theoretisch kann jeder seine eigene Root CA aufbauen. Das wird auch gemacht, zum Beispiel um innerhalb des Unternehmens Zertifikate verteilen zu können. Natürlich sind diese Zertifizierungsstellen dann nicht außerhalb des Unternehmens vertrauenswürdig.

Diese Dezentralität bezieht sich auch auf den Umgang mit Zertifikaten. Wie eine Software oder ein Betriebssystem mit Zertifikaten umgeht, ist ihr überlassen. Wird also keine sichere Überprüfung von Zertifikaten implementiert, bringt das ganze System nichts.

So gibt es Statistiken, dass bis zu 10% aller Zertifikatssperrlisten nicht abrufbar sind. Das müsste dann eigentlich dazu führen, dass der Browser eine Warnmeldung ausgibt und darauf hinweist, dass das Zertifikat nicht überprüft werden konnte. Aber genau das passiert nicht. Die Browserhersteller möchten nicht zu viele Fehlalarme produzieren, Ich finde das verständlich, denn wenn man andauernd gewarnt wird, ignoriert man das nur noch. Damit würde auch eine berechtigte Warnung wahrscheinlich ignoriert werden.

## Die “Lösung”
Obwohl das System der Zertifikatssperrungen gut durchdacht ist und in der Theorie funktioniert, wird es in der Praxis nicht angewandt. Damit wird praktisch nicht überprüft, ob ein Zertifikat überhaupt noch gültig ist. Und selbst wenn, es werden ja kaum Zertifikate zurückgezogen. Ein selbstverstärkender Effekt.

Aber die Browserhersteller haben sich eine “Lösung” einfallen lassen. Alle Zertifikate, die länger als ein Jahr gültig sind, werden nicht mehr als sicher eingestuft. Es darf also praktisch nur noch Zertifikate geben, die maximal 12 Monate Gültigkeit haben. Toll, so wird nur ein Jahr lang nicht geprüft, ob das Zertifikat vielleicht zurückgezogen wurde. Das erhöht das Vertrauen in die sichere Datenkommunikation durch Zertifikate bei mir wirklich. Wie geht es euch?

## Alternativen
Obwohl Zertifikate ernste Probleme haben, sind sie extrem verbreitet. Es gibt zwar erste Ansätze wie DNSSEC, aber es wird wohl noch Jahre oder sogar Jahrzente brauchen, bis Zertifikate in der jetzigen Form nicht mehr verwendet werden.

In anderen Bereichen würde ich auch heute schon keine Zertifikate verwenden, nämlich wenn es um Email- und Datenverschlüsselung in Microsoft 365 geht. Dort gibt es bessere Alternativen nämlich Microsoft Purview Message Encryption und Sensitivity Labels.

## Abschluss
In der Theorie sind Zertifikate toll. Sie haben auch immer noch ihre Daseinsberechtigung. Vieles geht auch gar nicht ohne Zertifikate. Man sollte sich nur der oben beschriebenen Einschränkungen bewusst sein und evtl. prüfen, ob es für einen Anwendungsfall bessere Alternativen gibt.
