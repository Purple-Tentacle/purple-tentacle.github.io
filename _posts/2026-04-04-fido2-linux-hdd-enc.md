---
layout: post
title: "Linux Festplattenverschlüsselung mit FIDO2"
date: 2026-04-04 02:00:00 +0200
categories: IT-security FIDO2 Yubikey Encryption
---
# Einführung

[Linux Unified Key Setup (LUKS)](https://gitlab.com/cryptsetup/cryptsetup) bietet eine Festplattenverschlüsselung für Linux und ist Standard bei Ubuntu.

Man kann bei der Installation von Ubuntu ein Verschlüsselungskennwort festlegen, um die Daten zum Beispiel bei Diebstahl des Geräts zu schützen. Allerdings ist die Verschlüsselungsstärke von der Komplexität und Länge des Kennworts abhängig. Wen es interessiert hier mehr Info zu sicheren (und vermeintlich sicheren) Kennwörtern: [BSI](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Checklisten/sichere_passwoerter_faktenblatt.pdf?__blob=publicationFile&v=4#download=1)

So ein Kennwort kann man sich schlecht merken und man möchte es auch nicht bei jedem Start eingeben um die Festplatte zu entschlüsseln. Es liegt also nahe einen FIDO2 Stick wie einen Yubikey zu nutzen.

# Ablauf

Es empfiehlt sich, die Verschlüsselung direkt nach der Installation anzupassen. Falls etwas schief geht installieren wir Ubuntu einfach neu und verlieren wir so keine gespeicherten Dateien oder Konfigurationen.

1. Installation von Ubuntu mit LUKS-Verschlüselung
2. Ermittlung der verschlüsselten Partition
3. Backup des LUKS-Header
4. Registrierung der Yubikeys
5. Aktivierung der Yubikeys
6. Übernahme der Konfiguration
7. Erstellung recovery key
8. Test
9. Entfernen des Kennworts

## Installation von Ubuntu

Damit man später die Entschlüsselung mit einem Yubikey machen kann, muss die Festplatte schon bei der Installation verschlüsselt werden. Der Prozess ist Teil des Installationsassistenten und selbsterklärend. Man kann dabei ein einfaches Kennwort wählen, wir werden dieses später sowieso wieder entfernen.

## Ermittlung der verschlüsselten Partition

Mit dem Befehl `lsblk` kann man die vorhandenen Partitionen anzeigen. Dabei  sollte es eine mit einem "crypt" label geben.

Hier meine Ausgabe als Beispiel

```bash
lsblk

NAME              MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
<snip>
nvme0n1           259:0    0 931.5G  0 disk
├─nvme0n1p1       259:1    0     1G  0 part  /boot/efi
├─nvme0n1p2       259:2    0     2G  0 part  /boot
└─nvme0n1p3       259:3    0 928.5G  0 part
  └─dm_crypt-0    252:0    0 928.4G  0 crypt
    └─ubuntu--vg-ubuntu--lv
                  252:1    0 928.4G  0 lvm   /
```

Wir müssen also `/dev/nvme0n1p3` bearbeiten.

## Backup

Falls wir bei den weiteren Schritten einen Fehler machen, könnte die Festplatte nicht mehr entschlüsselbar sein. Es empfiehlt sich also, den LUKS-header erst einmal zu sichern.

```bash
sudo cryptsetup luksHeaderBackup /dev/nvme0n1p3 --header-backup-file luks_backup.img
```
> <span style="color: #EC253F; font-weight: bold;">&#10071; Wichtig</span>
>
> Natürlich sollte man die  Backup-Datei nicht auf der Festplatte selbst speichern.
> {: .alert}

## Registrierung der Yubikeys

Wir prüfen zuerst, ob unser eingesteckter Yubikey erkannt wird.

```bash
systemd-cryptenroll --fido2-device=list
PATH         MANUFACTURER PRODUCT               COMPATIBLE
/dev/hidraw1 Yubico       YubiKey OTP+FIDO+CCID ✓
```

Wenn man einen anderen FIDO2-stick als einen Yubikey verwendet, sollte man auf das Häkchen bei compatible achten.

Wird der Yubikey erkannt geht es weiter mit der Registrierung. Nicht vergessen den Yubikey zu berühren sobald er blinkt, sonst geht es nicht weiter. Natürlich muss das device durch das device eures Geräts ersetzt werden.

```bash
sudo systemd-cryptenroll --fido2-device=auto /dev/nvme0n1p3
```

## Aktivierung der Yubikeys

Bei der Installation und Verschlüsselung von Ubuntu wurde bereits eine Konfigurationsdatei angelegt. Dieser müssen wir nun die Möglichkeit der Entschlüsselung mit einem FIDO2-stick hinzufügen.

```bash
sudo vim /etc/crypttab

# add fido2-device=auto to the end of the line like this:
dm_crypt-0 UUID=abcd1234-1234-abcd-1234-1234ab7890ab none luks, fido2-device=auto
```

> <span style="color: #EC253F; font-weight: bold;">&#10071; Wichtig</span>
>
> Es sollten mindestens 2 Yubikeys registriert werden, damit man die Partition bei Verlust oder Defekt eines einzelnen noch entschlüsseln kann. Der zweite ist sicher und räumlich getrennt aufzubewahren.
> {: .alert}

## Übernahme der Konfiguration

``` bash
sudo update-initramfs -u
```

Beim nächsten Neustart wird die Konfiguration aktiv.

## Erstellung recovery key

Die Erstellng eines recovery key ist optional, wenn wir mindestens 2 Yubikey installiert haben. Es ist trotzdem empfehlenswert.

```bash
sudo systemd-cryptenroll --unlock-fido2-device=auto --recovery-key /dev/nvme0n1p3
```

> <span style="color: #EC253F; font-weight: bold;">&#10071; Wichtig</span>
>
> Natürlich sollte man den key nicht auf der Festplatte selbst speichern.
> {: .alert}

## Test

Nun ist es an der Zeit alles zu überprüfen. Wir starten den Rechner neu und schauen ob alle (!) Yubikeys die Festplatte verschlüsseln können. Erst wenn wir sicher sind, dass alles funktioniert können wir zum nächsten Schritt übergehen.

## Entfernen des Kennworts

> <span style="color: #EC253F; font-weight: bold;">&#10071; Wichtig</span>
>
> Vorher unbedingt testen, ob alle registrierten Yubikeys die Festplatte entschlüsseln können.
> {: .alert}

```bash
sudo cryptsetup luksRemoveKey /dev/nvme0n1p3
# Dann die bei der Installation festgelegte passphrase eingeben.
```

Beim nächsten Neustart sollte die passphrase nicht mehr funktionieren.

# Abschluss

Jetzt können wir unsere Festplatte nur noch mit einem FIDO2-stick entschlüsseln.

Keine unsicheren Kennwörter bzw. kein lästiges Eingeben eines komplexen Passworts mehr notwendig.

