---
layout: post
title: "Framework 12 Tablet Mode reparieren"
date: 2026-04-05 02:00:00 +0200
categories:  Framework Ubuntu Linux Gadget Tablet
---
# Einführung

Ich habe mir ja vor einiger Zeit das [Framework 12](https://frame.work/de/en/laptop12) gekauft. Bei diesem kann man ja den Bildschirm umklappen um es als Tablet zu nutzen. Am Anfang hat das auch wunderbar funktioniert, allerdings irgendwann nicht mehr. Ich vermute es wurde durch irgendeine andere Konfiguration oder durch ein update verursacht.

Framework hat dazu einen [Supportartikel](https://knowledgebase.frame.work/tablet-mode-and-screen-rotation-on-linux-SJkaIhBSbg), der allerdings eine wichtige Information auslässt, nämlich wie man die Ladereihenfolge der Kernel-Module dauerhaft festlegt.

# Wie macht Ubuntu die Umschaltung?

Damit die Umschaltung auf den Tablet-Modus funktioniert (also die Tastatur abgeschaltet wird und der Bildschirm passend rotiert wird) müssen 2 Kernel-Module geladen werden.

- `pinctrl_tigerlake` — Must be built into the kernel or loaded first
- `soc_button_array` — Handles GPIO button events including tablet mode switch

Diese sind bei aktuellen Ubuntu-Versionen enthalten. Bei meiner Installation von Ubuntu hat es am Anfang ja auch funktioniert.

# Problemsuche 

Bei mir hat der Kernel den zuständigen GPIO für die Umschaltung nicht erkannt. Der folgende Befehl lieferte nichts zurück.

```bash
journalctl -k | grep gpio-keys

# expected output
Apr 05 08:21:17 fw12 kernel: input: gpio-keys as /devices/platform/INT33D3:00/gpio-keys.1.auto/input/input5
```

Das Problem ist eine falsche Ladereihenfolge. Das Kernel-Modul `pinctrl_tigerlake` wurde erst nach `soc_button_array` geladen.

Wir können die Module in der richtigen Folge nachladen.

```bash
sudo rmmod soc_button_array
sudo modprobe soc_button_array
```

Wenn jetzt alles geht haben wir das Problem behoben, allerdings nur temporär.  Beim nächsten Neustart wird das Problem wieder auftreten.

Wir müssen die Reihenfolge dauerhaft festlegen.

```bash
sudo vim /etc/modules-load.d/tablet-fix.conf
# type the name of the module in the file:
# pinctrl_tigerlake

sudo vim /etc/dracut.conf.d/tablet-fix.conf
# Add this line (the whitespace inside the quotes is important):
add_drivers+=" pinctrl_tigerlake "

# Regenerate your Boot Image
sudo dracut -f
```

Dann noch neu starten und das Problem sollte behoben sein.



