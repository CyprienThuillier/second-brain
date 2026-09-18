# Audit Android

## Installation d'Android Studio sous Ubuntu

Télécharger sur [https://developer.android.com/studio](https://developer.android.com/studio), puis :

```bash

tar xvf android-studio-<version>-linux.tar.gz

sudo mv android-studio /usr/local

sudo ln -s /usr/local/android-studio/bin/studio.sh /usr/local/bin/studio

```

## Installation d'Android Debug Bridge

```bash

sudo apt install adb

```

## Téléchargement d'un *virtual device*

- Lancer Android Studio :

```bash

studio

```

- More Actions -> Virtual Device Manager -> Create Device -> Pixel_10_Pro_XL (API 36+)

- Dans la fenêtre "Configure virtual device", sélectionner Services : Google APIs (au lieu de Google Play Store)

- Fermer Android Studio

## Emulation

```bash

nano ~/.bashrc

```

Ajouter ces lignes dans le *.bashrc* :

```bash

export ANDROID_HOME=$HOME/Android/Sdk

export PATH=$PATH:$ANDROID_HOME/emulator

export PATH=$PATH:$ANDROID_HOME/platform-tools

```

Puis recharger le *.bashrc* :

```bash

source ~/.bashrc

```

Lancer l’émulation :

```bash

emulator -list-avds

emulator -avd Pixel_10_Pro_XL -writable-system

```

Si vous avez en permanence des messages "Emulator is not responding", taper la commande suivante dans un terminal sur votre hôte linux :

```bash

gsettings set org.gnome.mutter check-alive-timeout 00

```

Pour lancer l'émulateur sous windows :

```

C:\Users\ABCD1234\AppData\Local\Android\Sdk\emulator>emulator -avd Pixel_10_Pro_XL -writable-system

```

## Installation de l'application

```bash

adb install <app>.apk

```

## Configuration de l'interception

Lancer Burp et exporter le certificat Portswigger au format DER depuis l'onglet *Proxy Settings*, puis :

```bash

$ openssl x509 -inform DER -subject_hash_old -in burp.der

9a5ba575

-----BEGIN CERTIFICATE-----

MIIDqDCCApCgAwIBAgIFAKfp8zowDQYJKoZIhvcNAQELBQAwgYoxFDASBgNVBAYT

C1BvcnRTd2lnZ2VyMRQwEgYDVQQIEwtQb3J0U3dpZ2dlcjEUMBIGA1UEBxMLUG9y

[...]

```

Renommer *burp.der* en fonction du résultat sur la 1ère ligne (ici **"9a5ba575"**), avec un zéro comme extension ; par exemple :

```bash

mv burp.der 9a5ba575.0

```

Pousser le certificat émulé :

```bash

adb root

adb remount

adb push 9a5ba575.0 /system/etc/security/cacerts

adb reboot

```

On doit voir le certificat Portswigger dans la liste des certificats autorisés dans le terminal android émulé. Selon la version android dans l'écran du terminal ouvrir les Paramètres (Settings) puis :

Settings / Security and Privacy / More security & privacy / Encryption & credentials / Trusted credentials

Si ça n'a pas fonctionné : il est également possible d'importer le certificat au format .der directement depuis android après l'avoir déposé dans les Téléchargements du téléphone émulé :

```bash

adb root

adb remount

adb push burp.der /sdcard/Download/

adb reboot

```

puis directement depuis l'écran du téléphone émulé :

Settings / Security and Privacy / More security & privacy / Encryption & credentials / Install a certificate / CA Certificate

Une alerte "Your data won't be private" s'affiche, cliquer sur "INSTALL ANYWAY"

Dans l'écran suivant, cliquer sur le fichier burp.der

On revient à la fenêtre "Install a certificate", avec un message "CA Certificate Installed" qui s'affiche en bas de la fenêtre pendant 3 secondes.

On peut vérifier que le certificat Portswigger a bien été installé, toujours depuis l'écran du téléphone émulé :

Settings / Security and Privacy / More security & privacy / Encryption & credentials / Trusted credentials / User

Le certificat PortSwigger doit être présent dans la liste.

Dans l'émulateur, définir Burp comme proxy dans les paramètres (trois points tout en bas de la barre d'outils à côté de l’écran du téléphone -> Settings -> Proxy)

Attention, pour que le proxy Burp intercepte les requêtes, il peut être nécessaire de désactiver le ssl pinning pour l'app concernée (cf plus bas, utilisation de frida et objection)

## Utiliser un appareil Android physique

Vous pouvez également utiliser un appareil Android rooté pour utiliser Frida et Objection, c'est parfois plus simple et évite de consommer trop de ressources sur le PC.

Pour se connecter avec Adb, vous pouvez soit vous connecter par cable, soit vous connecter par le Wifi.

Il faut que le téléphone soit sur le meme réseau Wifi et que vous installiez l'application de **WADB** (Wireless ADB) pour ouvrir le port 5555. Je recommande cette application : [https://github.com/RikkaApps/WADB](https://github.com/RikkaApps/WADB)

**Installer le certificat Burp :**

```bash

openssl x509 -inform DER -subject_hash_old -in burp.der

# ex: 9a5ba575

mv burp.der 9a5ba575.0

adb push 9a5ba575.0 /sdcard/Download/

```

Puis sur le téléphone : Paramètres -> Sécurité -> Chiffrement et identifiants -> Installer un certificat -> CA -> sélectionner le fichier.

**Promouvoir le cert système via un module Magisk :**

```bash

adb push AlwaysTrustUserCerts.zip /sdcard/Download/

```

Sur le téléphone : ouvrir l'app **Magisk** -> Modules -> Install from storage -> sélectionner le fichier `AlwaysTrustUserCerts.zip` -> reboot.

Après redémarrage, vérifier dans Paramètres -> Sécurité -> Identifiants de confiance -> onglet **Système** que le certificat PortSwigger y apparaît bien.

Ensuite, assurez vous qu'aucun VPN ne pose de problème et configurez votre VM pour pouvoir communiquer avec le telephone. (En bridge)

Lancez l'application et activez le service (bouton en haut a droite), vous obtiendrez alors votre adresse IP privée avec le port associe (souvent 5555).

Sur votre VM, utilisez la commande :

```bash

adb devices

```

si vous n'avez aucun résultat, vous pouvez faire la commande :

```bash

adb connect 192.168.1.16:5555

```

avec la bonne adresse IP et le bon port, évidemment.

## Utiliser Frida et Objection

On installe l'outil **Frida** :

```bash

pip install frida-tools

```

On installe également l'outil **Objection** :

```bash

pip install objection

```

Il peut être préférable d'utiliser un environnement virtuel python. Dans ce cas, utiliser les commandes suivantes pour activer l'environnement virtuel python dans le répertoire courant, y installer frida et objection, puis tester frida avec un frida-ps par exemple :

```bash

python3 -m venv .

source bin/activate

pip install frida-tools

pip install objection

frida-ps

```

Téléchargez le serveur Frida : [https://github.com/frida/frida/releases](https://github.com/frida/frida/releases) (attention à bien dérouler la liste des pour sélectionner frida-server-XX.X.X-android-XXX.xz avec votre architecture android adaptée et la même version de frida que celle installée à l'étape ci-dessus; Si une des 2 conditions n'est pas remplie, ça risque de ne pas marcher, ou pire de ne pas marcher correctement).

Extraire le binaire avec la commande :

```bash

xz -df frida-server-XX.X.X-android-XXX.xz

```

⚠️ Cette commande va extraire un binaire. Nous le renommerons : frida-server

Etant connecté sur l'appareil, on envoie le serveur dans un dossier temporaire de l'appareil :

```bash

adb root && adb push frida-server /data/local/tmp/

```

On ajoute ensuite les autorisations nécessaires pour lancer le serveur :

```bash

adb shell 'chmod 755 /data/local/tmp/frida-server'

```

On lance le serveur :

```bash

adb shell '/data/local/tmp/frida-server &'

```

Une fois le serveur lance, nous sommes en mesure de lancer :

```bash

frida-ps -Uai

```

Qui nous permet d'afficher les applications en cours d'execution sur l'appareil.

On peut désormais utiliser Objection pour se connecter a un PID (id de processus d'application) :

```bash

objection -g <pid> explore

```

Une fois le terminal objection lance, vous pouvez par exemple désactiver le SSLpinning a l'aide de cette commande :

```bash

android sslpinning disable

```

Les bases pour apprendre à utiliser Objection peuvent être retrouvées sur ce site : [https://hacktricks.wiki/en/mobile-pentesting/android-app-pentesting/frida-tutorial/objection-tutorial.html](https://hacktricks.wiki/en/mobile-pentesting/android-app-pentesting/frida-tutorial/objection-tutorial.html)

Mais il n'englobe pas la totalité des fonctionnalités disponibles, il faut faire des recherches plus approfondies pour découvrir tout ce que propose l'outil (dans l'outil directement qui propose une interface claire et intuitive).

Il n'est également pas recommandé d'utiliser ChatGPT ou tout autre intelligence artificielle pour trouver des commandes, ces dernières générant des commandes qui n'existent pas dans la majorité des cas.
## Kreya

Installer Kreya via https://kreya.app/downloads/. 

Pour linux :

```shell
sudo snap install kreya
```

Créer un projet en important un fichier/dossier proto et déclarer ce dossier comme chemin d'import.

Utiliser la fonction `authenticate` pour récupérer des jetons et `getDefaultContract` pour extraire un identifiant de contrat. Utiliser le bouton `send` pour envoyer les requêtes.

## MobSF

Setup avec docker :

```shell
docker pull opensecurity/mobile-security-framework-mobsf:latest
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest

# Default username and password: mobsf/mobsf
```

Puis ouvrir l'interface de MobSF via http://127.0.0.1:8000 en utilisant le login par default : `mobsf/mobsf` et importer l'APK dessus pour générer le rapport et l'analyser.