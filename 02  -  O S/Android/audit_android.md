# Audit Android

## Installation d'Android Studio sous Ubuntu

Télécharger sur https://developer.android.com/studio, puis :

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
Pour windows :
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

Dans l'émulateur, définir Burp comme proxy dans les paramètres (trois points tout en bas de la barre d'outils à côté de l’écran du téléphone -> Settings -> Proxy)

## Utiliser un appareil Android physique

Vous pouvez également utiliser un appareil Android rooté pour utiliser Frida et Objection, c'est parfois plus simple et évite de consommer trop de ressources sur le PC. 
Pour se connecter avec Adb, vous pouvez soit vous connecter par cable, soit vous connecter par le Wifi.

Il faut que le téléphone soit sur le meme réseau Wifi et que vous installiez l'application de **WADB** (Wireless ADB) pour ouvrir le port 5555. Je recommande cette application : https://github.com/RikkaApps/WADB

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

Téléchargez le serveur Frida : https://github.com/frida/frida/releases (attention à bien dérouler la liste des pour sélectionner frida-server-XX.X.X-android-XXX.xz avec votre architecture android adaptée).

![[android_frida1.png]]

![[android_frida2.png]]

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

Les bases pour apprendre à utiliser Objection peuvent être retrouvées sur ce site : https://hacktricks.wiki/en/mobile-pentesting/android-app-pentesting/frida-tutorial/objection-tutorial.html
Mais il n'englobe pas la totalité des fonctionnalités disponibles, il faut faire des recherches plus approfondies pour découvrir tout ce que propose l'outil (dans l'outil directement qui propose une interface claire et intuitive).
Il n'est également pas recommandé d'utiliser ChatGPT ou tout autre intelligence artificielle pour trouver des commandes, ces dernières générant des commandes qui n'existent pas dans la majorité des cas.
