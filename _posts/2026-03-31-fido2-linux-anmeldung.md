---
layout: post
title: "FIDO2-Anmeldung unter Linux"
date: 2026-03-31 02:00:00 +0200
categories: IT-security Linux FIDO2 Framework
---
# Einführung

Unter Windows mit Microsoft 365 und Windows Hello for Business nutze ich schon lange FIDO2-sticks für Anmeldungen. Das wollte ich natürlich auch unter Linux nutzen. Meine Wahl fiel dabei auf die Yubikey 5, da ich auch die erweiterten Fähigkeiten wie PIV Smartcard nutzen möchte.

Ich bin ein sehr großer Fan von FIDO2, da eine passwortlose und trotzdem sichere und phishing-resistente Anmeldung ermöglicht wird. Damit sind Passwörter und one-time-token immer mehr obsolet.

# Ablauf

Um sich mit dem Yubikey anmelden zu können, bedarf es mehrerer Schritte.

1. Installation der notwendigen Programme
2. Registrierung der FIDO2 sticks
3. Verfügbar Machen der registrierten sticks für die Anmeldung
4. Testen der Anmeldung mit **beiden** Sticks
5. Passwortanmeldung deaktivieren

> span style="color: #3171FC; font-weight: bold;">Info</span>
>
> Ich rede hier immer von sticks, also Plural. Es ist wichtig mindestens 2 sticks einzurichten, falls mal einer verloren oder kaputt geht. Einen davon sollte man räumlich getrennt und sicher aufbewahren.
{: .alert}

## Installation

```bash
sudo apt update
sudo apt install libpam-u2f pamu2fcfg
mkdir -p ~/.config/Yubico
```

## Registrierung

Nach dem Absetzen der Befehle fängt der Yubikey an zu blinken, dann muss man diesen berühren.

> <span style="color: #EC253F; font-weight: bold;">Wichtig</span>
>
> Man beachte die `>>` beim zweiten Befehl. Wir wollen ja den zweiten stick anhängen und nicht den ersten überschreiben.
{: .alert}

```bash
# first FIDO2-stick
pamu2fcfg > ~/.config/Yubico/u2f_keys #enter PIN and touch key

# second FIDO2-stick
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys #enter PIN and touch key
```

## Registrierung

Die Registrierung der Yubikeys erfolgt wie bei Linux üblich über eine Konfigurationsdatei, nämlich `/etc/pam.d/common-auth`. Hierbei muss man verstehen, dass diese von oben nach unten abgearbeitet wird. Das wird noch wichtig beim Deaktivieren der Passwortanmeldung.

Wir überschreiben also nicht die Standards in der Konfigurationsdatei, sondern fügen unsere hinzu.

Ganz oben (bzw. unter den einführenden Kommentaren) muss diese Zeile eingefügt werden:

```bash
auth [success=done default=ignore] pam_u2f.so authfile=/home/chris/.config/Yubico/u2f_keys cue pinverification=1
```

Darunter folgen die bereits vorhandenen Zeilen. Diese fassen wir nicht an.

```bash
auth   [success=2 default=ignore]      pam_unix.so nullok
auth   [success=1 default=ignore]      pam_sss.so use_first_pass
```

## Test

Jetzt kann man testen, ob alles funktioniert. `sudo` sollte nach der Yubikey-PIN fragen. Nach dem Absetzen der Befehle fängt der Yubikey an zu blinken, dann muss man diesen berühren. Nach dem berühren des Sticks sollte man angemeldet werden.

> <span style="color: #EC253F; font-weight: bold;">Wichtig</span>
>
> Ich rede hier immer von sticks, also Plural. Es ist wichtig mindestens 2 sticks einzurichten.
{: .alert}
>
> Unbedingt beide sticks testen.
{: .alert}

```bash
sudo ls -la
```

![image-20260331121127393](/assets/images/2026-03-31-fido2-linux-anmeldung/image-20260331121127393.png)

Als nächstes testen wir die Benutzeranmeldung.

![image-20260331121241453](/assets/images/2026-03-31-fido2-linux-anmeldung/image-20260331121241453.png)

![image-20260331121259635](/assets/images/2026-03-31-fido2-linux-anmeldung/image-20260331121259635.png)

Wenn beide sticks funktionieren, dann können wir jetzt die passwortbasierte Anmeldung deaktivieren.

## Passwortanmeldung deaktivieren

Hierfür müssen wir nur eine weitere Zeile zur `/etc/pam.d/common-auth` hinzufügen, und zwar zwischen unsere Zeile von vorher und den Standardzeilen.

```bash
# Unsere Zeile von vorher
auth [success=done default=ignore] pam_u2f.so authfile=/home/chris/.config/Yubico/u2f_keys cue pinverification=1

# Neue Zeile, Passwort deaktivieren
auth mandatory pam_deny.so

# Standardzeilen
auth   [success=2 default=ignore]      pam_unix.so nullok
auth   [success=1 default=ignore]      pam_sss.so use_first_pass
```

Da die Datei von oben nach unten abgearbeitet werden, gelten die Standardanmeldemethoden weiterhin. Allerdings gibt es davor einen deny, die Standardmethoden werden also gar nicht erst angeboten.

## Abschluss

Endlich passwortlose und sichere Authentifizierung unter Linux. Bequem und trotzdem extrem sicher.

Je nach Sicherheitsbedürfniss könnte man einen Yubikey nano sogar dauerhaft stecken lassen. Das ist dank PIN immer noch ein sehr guter Schutz vor lokalen Angriffen. Vor entfernten Angriffen ja sowieso, da der stick ja berührt werden muss.

Als nächstes richten wir die Festplattenentschlüsselung mit Yubikey ein.
