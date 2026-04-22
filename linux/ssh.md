# SSH Config

Mehrere SSH Keys verwalten z.B. für GitHub, Server, etc.

## Config Datei Beispiel

```bash
Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/github
```

## Private Key einrichten

Datei erstellen und Key reinkopieren:

```bash
nano ~/.ssh/github
```

Rechte setzen (600 = nur du kannst lesen/schreiben, niemand sonst):

```bash
chmod 600 ~/.ssh/github
```

## SSH Verbindung testen

```bash
ssh -T git@github.com
```

## Häufige Fehler

**Permission denied** → Rechte nicht korrekt gesetzt, `chmod 600` nochmal ausführen

**Key wird nicht erkannt** → Prüfen ob der richtige Key in der Config steht:

```bash
cat ~/.ssh/config
```