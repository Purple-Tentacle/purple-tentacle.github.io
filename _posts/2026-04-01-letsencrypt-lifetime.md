---
layout: post
title: "Let's Encrypt verringert Lebensdauer"
date: 2026-04-01 02:00:00 +0200
categories: IT-security Certificate
---
# Einführung

In meinem Artikel [Warum Zertifikate kaputt sind](https://hemker.de/it-security/2023/11/21/warum-zertifikate-kaputt-sind.html) habe ich ja bereits ausführlich beschrieben, dass die Zertifikatsinfrastruktur zwar Mechanismen hat, die Gültigkeit von Zertifikaten zu überprüfen, diese aber nicht wirklich angewendet werden. Dadurch kann es zu einem falschen Vertrauen in kompromittierte Zertifikate kommen.

Daher haben sich die Browserhersteller darauf geeinigt, die maximale Lebensdauer von Zertifikaten auf ca. ein Jahr zu begrenzen. Let's Encrypt Zertifikate waren von Anfang an sogar nur 90 Tage gültig.

Jetzt gibt es Neuigkeiten bezüglich der vereinbarten Lebensdauer allgemein und der Lebensdauer bei Let's Encrypt.

# Allgemeine Vereinbarung

Wenn ich von den "Browserherstellern" spreche, dann meine ich eigentlich das [CA/Browser Forum](https://cabforum.org/about/). Das ist ein Zusammenschluss von Browserherstellern, die gemeinsam Regeln beschließen.

Eine dieser Regeln ist Ende 2025 beschlossen worden, nämlich die Verkürzung der Lebensdauer von Zertifikaten. Diese wird gestaffelt immer weiter verringert.

| Datum      | Lebensdauer      |
| ---------- | ---------------- |
| 2026-03-15 | maximal 200 Tage |
| 2027-03-15 | maximal 100 Tage |
| 2029-03-15 | maximal 47 Tage  |

[Quelle](https://cabforum.org/working-groups/server/baseline-requirements/requirements/)

Man hat also die Zeichen der Zeit erkannt und gehandelt.

# Let's Encrypt

Let's Encrypt war schon immer strenger bei der Lebensdauer und hat dafür auf automatische Verlängerungen gesetzt. Das wird auch deutlich, wenn man sich die Einführung von kürzeren Lebensdauern anschaut. Diese gehen sowohl beim Einführungszeitpunkt, als auch bei der Lebensdauer deutlich über die Maximalwerte des CA/Browser Forums hinaus.

| Datum      | Lebensdauer                                                  |
| ---------- | ------------------------------------------------------------ |
| 2026-05-13 | maximal 45 Tage<br />opt-in, also nicht verpflichtend<br />es gelten also wie bisher maximal 90 Tage |
| 2027-02-10 | maximal 64 Tage<br />verpflichtend                           |
| 2028-02-16 | maximal 45 Tage<br />verpflichtend                           |

[Quelle](https://letsencrypt.org/2025/12/02/from-90-to-45#timeline-of-changes)

## Automatisierung

Da es so immer häufiger vorkommt, dass ein Zertifikat verlängert werden muss, wird der administrative Aufwand immer größer. Daher sollte man Automatisierung nutzen.

Bei wildcard-Zertifikaten musste dafür bei jeder Verlängerung ein DNS TXT Eintrag gesetzt werden. Das lässt sich natürlich nur automatisieren, wenn man diesen Eintrag auch automatisch setzen kann. In Azure gibt es dafür eine Lösung, die ich auch erfolgreich einsetze: [acmebot](https://github.com/polymind-inc/acmebot).

Die Anzahl der unterstützten DNS-Anbieter ist aber begrenzt. Setzt man einen anderen ein, kann nicht automatisiert werden.

- Amazon Route 53
- Azure DNS
- Cloudflare
- DNS Made Easy
- Gandi LiveDNS
- GoDaddy
- Google Cloud DNS
- TransIP DNS
- Custom DNS provider

Let's Encrypt ist sich der Problematik bewusst und möchte noch 2026 [DNS-PERSIST-01](https://www.ietf.org/archive/id/draft-ietf-acme-dns-persist-00.html) einführen. Damit müsste man nur einmalig einen DNS TXT Eintrag setzen und nicht bei jeder Erneuerung. Ich bin sehr gespannt wann das kommt und wie gut das funktioniert.

## Abschluss

Da die verteilte Zertifikatsinfrastruktur nicht wirklich gut funktioniert, wenn es um Sperrungen von kompromittierten Zertifikaten geht, wird die maximale Lebensdauer immer weiter verringert. Das ist wohl auch der einzige mögliche Weg die Sicherheit zu verbessern, ein weltweiter Ersatz durch eine andere Lösung ist momentan kaum vorstellbar.
