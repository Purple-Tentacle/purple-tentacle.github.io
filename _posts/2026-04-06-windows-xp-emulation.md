---
layout: post
title: "Framework 12 Tablet Mode reparieren"
date: 2026-04-06 02:00:00 +0200
categories:  WindowsXP Retro Gaming Virtualization
---
# Einführung

Ich bin ein großer Fan von alten Spielen und nutze dafür neben vielen anderen Systemen auch ein altes Windows XP. Dieses nutze ich z. B., um [Stars!](https://en.wikipedia.org/wiki/Stars!) zu spielen. Wobei ich immer frage,ob noch jemand in Deutschland oder Europa dieses absolute Nischenspiel kennt bzw. sogar spielt.

![image-20260406104129928](/assets/images/2026-04-06-windows-xp-emulation/image-20260406104129928.png)

Jedenfalls hatte ich unter Linux mit QEMU im virtualisierten Windows XP einige Probleme. So ging die Systemuhr viel zu schnell und Mausklicks wurden mehrfach registriert, was gerade bei einem Spiel wie Stars! problematisch wird.

# Problemlösung

Einige Recherche hat ergeben, dass es wohl einen bug in QEMU gibt und dass man das System der virtuellen Maschine in QEMU umstellen soll. Das hat bei mir allerdings zu Folgefehlern in der XML-Datei geführt. Es musste also eine andere Lösung her.

Am Ende war es viel einfacher als gedacht, lediglich das Power Management in Windows XP musste umgestellt werden. Sonst führt ein energiesparender Modus der CPU wohl dazu, dass CPU-cycles nicht mehr richtig funktionieren.

Also `Start` - `Control Panel` - `Power Options` und dort das Power Scheme auf `Always On` stellen.

![image-20260406104015241](/assets/images/2026-04-06-windows-xp-emulation/image-20260406104015241.png)

Schon sind alle Probleme verschwunden.





