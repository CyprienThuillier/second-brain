## Installation & Setup

```bash
sudo dnf install gobuster
```

Install the wordlists :

```bash
git clone https://github.com/danielmiessler/SecLists.git ~/wordlists/SecLists
```

## Syntax :

```bash
gobuster <command> [command options]
```

#### Commands :

```bash
dir
	Uses directory/file enumeration mode
vhost
	Uses VHOST enumeration mode
dns
	Uses DNS subdomain enumeration mode
fuzz
	Uses fuzzing mode. Peplcaes the keyword FUZZ in the URL, Headers and the request body
tftp
	Uses TFTP enumeration mode
s3
	Uses aws bucket enumeration mode
gcs
	Uses gcs bucket enumeration mode
```

