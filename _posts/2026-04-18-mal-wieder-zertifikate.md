---
layout: post
title: "Mal wieder Zertifikate"
date: 2026-04-18 02:00:00 +0200
categories: IT-security Zertifikate
---
# Einführung

Es gibt mal wieder Neuigkeiten aus der Welt der Zertifikate.

In meinem Beitrag [Warum Zertifikate kaputt sind](https://hemker.de/it-security/2023/11/21/warum-zertifikate-kaputt-sind.html) habe ich ja bereits erläutert, dass die weltweite Zertifikatsinfrastruktur nicht wirklich gut funktioniert und dass deshalb als Notlösung die Laufzeiten der Zertifikate immer weiter verringert werden. Siehe auch meinen Artikel über [Let's Encrypt Laufzeiten](https://hemker.de/it-security/certificate/2026/04/01/letsencrypt-lifetime.html).

Dann kam plötzlich der GAU bei der Zertifizierungsstelle D-Trust. Es mussten sehr kurzfristig fast 60.000 (!) Zertifikate ausgetauscht werden. Und D-Trust ist nicht irgendeine Wald- und Wiesen Zertifizierungsstelle, sondern wird von der Bundesdruckerei betrieben.

# Was war passiert

D-Trust musste fast 60.000 Zertifikate für ungültig erklären, da diese nicht den Regeln des CA/Browser Forum entsprochen haben.

Zur Erinnerung, dieses Forum legt die Zertifikatsregeln fest und die meisten Browserhersteller halten sich daran. Wenn man also gegen Regeln verstößt, droht im schlimmsten Fall der komplette Vertrauensentzug für alle jemals und alle zukünftig ausgegebenen Zertifikate. Das ist dann der Todesstoß für die Zertifizierungsstelle.

Dementsprechend hat D-Trust reagiert und das kleinere Übel gewählt. Besser zehntausende Zertifikate für ungültig erklären, als das Vertrauen der Browserhersteller zu verlieren.

# Die Ursache

Warum drohte der Vertrauensverlust? Da muss ich erst ein wenig ausholen.

## Precertificates

Bei öffentlichen Zertifizierungsstellen werden sogenannte precertificates ausgegeben. Diese gehen nicht an den eigentlichen Anforderer, sondern an bestimmte Dienstleister. Diese führen ein Verzeichnis dieser Zertifikate. Außerdem stellen sie der Zertifizierungstelle einen Zeitstempel (Signed Certificate Timestamp, SCT) zur Verfügung, welcher in das endgültige Zertifikat eingebaut wird.

Dieser ganze Vorgang dient der sogenannten Certificate Transparency. Jeder wird dadurch in die Lage versetzt, ausgestellte Zertifikate einzusehen und sich von der Korrektheit zu überzeugen.

## Linting

Zertifizierungsstellen müssen überprüfen, ob der Antragsteller überhaupt berechtigt ist. So muss überprüft werden, ob die domain dem Antragsteller gehört. Das kann zum Beispiel über einen DNS-Eintrag erfolgen.

Zur Überprüfung der Anträge gibt es Werkzeuge, allen voran ZLint und PKILint. Diese halten sich streng an die Regeln des CA/Browser Forums. So verringern die Zertifizierungsstellen das Risiko sich unbewusst nicht an Regeln zu halten.

Aber warum vorhandene und erprobte Werkzeuge nutzen, wenn man auch was eigenes bauen kann?

## Erwischt

Lange Zeit ist nicht aufgefallen, dass der selbst entwickelte Linter nicht alle notwendigen Überprüfungen vorgenommen hat. Das ist erst aufgefallen, als das CA/Browser Forum bemerkt hat, dass die Precertificates eine zu lange Gültigkeitsdauer hatten. Nur 3 Tage zu viel, aber Regeln sind Regeln. Gerade in so einem sensitiven Bereich.

Also hat das Forum kurzerhand bei D-Trust nachgefragt, wie das linting bewerkstelligt wird. Und es stellte sich heraus, dass es nicht den Erfordernissen des Forums entsprach. Also mussten fast 60.000 Zertifikate zurückgezogen werden und das linting auf ZLint und PKILint umgestellt werden.

# Abschluss

Irgendwie funktioniert es dann doch, die weltweite Zertifikatsinfrastruktur als vertrauenswürdig zu erhalten. Wobei ich mich schon frage wie viele Fälle es gibt, wo Fehler nicht auffallen die das Vertrauen gefährden könnten. Aber eine wirkliche Alternative ist eh nicht in Sicht. Daher ist es schon gut, dass es workarounds gibt.
