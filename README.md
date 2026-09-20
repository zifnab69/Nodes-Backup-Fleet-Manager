> 🇫🇷 **French below — Version française plus bas**

---

# Nodes Backup & Fleet Manager

A Python/Tkinter desktop application to **backup, restore and deploy Meshtastic node configurations** over USB on Windows (and MAC\Linux).

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![License](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey)
![Platform](https://img.shields.io/badge/Platform-Windows-blue)
![Meshtastic](https://img.shields.io/badge/Meshtastic-2.x-green)

---

## Features

- **Full config export** — reads the complete node configuration and saves it as a structured JSON file (owner, LoRa, channels + PSK, Bluetooth, network, modules, security keys…)
- **Full config restore** — writes back every section to the node, including AES-256 channel PSKs, admin keys and security keys
- **Fleet profile generation** — strips device-unique data (private/public keys, owner, WiFi credentials) and keeps the shared config (admin_key, LoRa, channels, modules…) for multi-node deployment
- **Sequential multi-node export** — export multiple nodes one after another in the same session
- **Sequential multi-node import** — deploy the same JSON file to multiple nodes in one session
- **Two-tab key fields editor** — edit owner, LoRa region, modem preset, override frequency, override duty cycle and device role in the "Main" tab; edit all 8 channels (enable/role, name, Base64 PSK, GPS precision, key generator) in the "Channels" tab — no need to touch the JSON
- **Restore progress bar** — step-by-step progress window (owner, sections, modules, channels, final commit) for single- and multi-node restores
- **Robust channel injection** — canonical restore order with post-commit verification and automatic retry of channels silently rejected by the device
- **Integrity validation** — automatic pre-restore checks with warnings
- **Editable raw-JSON viewer** and **copyable import logs**
- **Persistent settings** — language and last working directory saved in `NBFM_Config.json`
- **Small-screen friendly** — global scrollbar on the main tab and scrollable editor tabs: every block stays reachable whatever the window height
- **Standalone EXE** — compiled with PyInstaller, no Python installation required
- **ENG and FR** — UI in English or French.

---

## Screenshots

![alt](https://github.com/zifnab69/Nodes-Backup-Fleet-Manager/blob/a27df849a4daddf056448f14d2b23e034a597be1/Images/CM_Firstpannel%201.77.jpg)
---

## Requirements

### Running from source

```
Python 3.10+
meshtastic
pyserial
protobuf
```

```bash
pip install meshtastic pyserial protobuf
python NBFM_V1.96.py
```

### Running the standalone EXE

No installation needed. Download the EXE from the [Releases](../../releases) page and run it directly.

---

## Quick start

1. Connect your Meshtastic device via USB
2. Launch the application
3. Select the COM port (COM1 is automatically excluded — Windows system port)
4. Click **EXPORT FULL CONFIG → JSON** to save your configuration
5. To restore, select a JSON file in the list and click **RESTORE selected file → Device**
6. Restart the device to apply changes

---

## Fleet deployment workflow

1. Export a reference node to NBFM
2. Click **GENERATE FLEET PROFILE** — this removes unique keys and owner info
3. Select the generated `*_fleet.NBFM` file
4. Click **Multi-node import** and restore to each node in sequence

---

## JSON structure

| Section | Description |
|---|---|
| `owner` | long_name, short_name, hw_model |
| `local_config` | LoRa, Bluetooth, network, display, position, power, device, security |
| `module_config` | MQTT, serial, telemetry, canned messages, range test, … |
| `channels` | Channels 0–7 with PSK (AES-256), name, role, module_settings |
| `known_nodes` | List of known mesh nodes |
| `my_info` | Raw local node info |
| `metadata` | Firmware metadata |

---

## File naming

```
meshtastic_[short_name]_[YYYYMMDD]_[HHMMSS].NBFM
```

Example: `meshtastic_JMC_5F7B_20260509_095500.NBFM`

The short_name includes the last 4 hex characters of the node's MAC address for unique identification.

---

## Channel restore order

Channels are restored in the canonical order used by the official Meshtastic CLI (`setURL`):

1. Primary channel (role = 1) first (index 0)
2. Secondary channels (role = 2) next
3. Disabled channels (role = 0) last

After the commit, NBFM re-reads the channels and automatically retries any active channel silently rejected by the device.

---
## Disclamer
I’m not a software developer by training. This tool was built with the help of an AI and a lot of work on my side to design, test, and refine it. If you’re a developer and would like to contribute, any help to keep this project alive and evolving is greatly appreciated.
## License

GPLv3

---

## Author

**zifnab69** — ZIFNAB69_fr@yahoo.fr


---


# 🇫🇷 Version française

# Nodes Backup and Fleet Manager

Application Python/Tkinter pour **sauvegarder, restaurer et déployer les configurations de nœuds Meshtastic** via USB sous Windows.

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Licence](https://img.shields.io/badge/Licence-CC%20BY--NC--SA%204.0-lightgrey)
![Plateforme](https://img.shields.io/badge/Plateforme-Windows-blue)
![Meshtastic](https://img.shields.io/badge/Meshtastic-2.x-green)

---

## Fonctionnalités

- **Export complet de la configuration** — lit toute la config du nœud et la sauvegarde en JSON (owner, LoRa, canaux + PSK, Bluetooth, réseau, modules, clés de sécurité…)
- **Restauration complète** — réécrit chaque section vers le nœud, y compris les PSK AES-256, les admin_key et les clés de sécurité
- **Génération de profils flotte** — supprime les données uniques (clés privées/publiques, owner, WiFi) et conserve la config partagée (admin_key, LoRa, canaux, modules…) pour déploiement multi-nœuds
- **Export multi-nœuds séquentiel** — exporter plusieurs nœuds l'un après l'autre dans la même session
- **Import multi-nœuds séquentiel** — déployer le même fichier JSON sur plusieurs nœuds en une session
- **Éditeur de champs clés en deux onglets** — onglet « Principal » (owner, région LoRa, modem preset, fréquence override, override duty cycle, rôle) ; onglet « Canaux » (les 8 canaux : activation/rôle, nom, PSK Base64, précision GPS, générateur de clés) — sans toucher au JSON
- **Barre de progression de la restauration** — fenêtre étape par étape (owner, sections, modules, canaux, commit final), en mono et multi-nœuds
- **Injection canaux robuste** — ordre de restauration canonique, vérification post-commit et relance automatique des canaux silencieusement rejetés par l'appareil
- **Validation d'intégrité** — vérifications automatiques avant chaque restauration
- **Visualiseur JSON éditable** et **journaux d'import copiables**
- **Réglages persistants** — langue et dernier dossier de travail conservés dans `NBFM_Config.json`
- **Compatible petits écrans** — ascenseur global sur l'onglet principal et onglets d'édition défilables : tous les blocs restent accessibles quelle que soit la hauteur de la fenêtre
- **EXE autonome** — compilé avec PyInstaller, aucune installation Python requise
- **ENG ou FR** — UI en anglais ou en français à la demande

---

## Captures d'écran

![alt](https://github.com/zifnab69/Nodes-Backup-Fleet-Manager/blob/a5b74e2e3f281fce2b458d4eb60ab02c5f94e7fd/Images/CM_vueprincipale%201.7.7.jpg)


---

## Prérequis

### Depuis le source Python

```
Python 3.10+
meshtastic
pyserial
protobuf
```

```bash
pip install meshtastic pyserial protobuf
python NBFM_V1.96.py
```

### Depuis l'EXE autonome

Aucune installation requise. Télécharger l'EXE depuis la page [Releases](../../releases) et le lancer directement.

---

## Démarrage rapide

1. Brancher l'appareil Meshtastic via USB
2. Lancer l'application
3. Sélectionner le port COM (COM1 exclu automatiquement — port système Windows)
4. Cliquer **EXPORTER CONFIG COMPLÈTE → JSON** pour sauvegarder la configuration
5. Pour restaurer, sélectionner un fichier JSON dans la liste et cliquer **RESTAURER le fichier sélectionné → Appareil**
6. Redémarrer l'appareil pour appliquer les changements

---

## Déploiement en flotte

1. Exporter un nœud de référence en NBFM
2. Cliquer **GÉNÉRER PROFIL FLOTTE** — les clés uniques et les infos owner sont supprimées
3. Sélectionner le fichier `*_fleet.NBFM` généré
4. Cliquer **Import multi-nœuds** et restaurer sur chaque nœud à la suite

---

## Structure du fichier JSON

| Section | Description |
|---|---|
| `owner` | long_name, short_name, hw_model |
| `local_config` | LoRa, Bluetooth, réseau, display, position, power, device, security |
| `module_config` | MQTT, série, télémétrie, messages prédéfinis, range test, … |
| `channels` | Canaux 0–7 avec PSK (AES-256), nom, rôle, module_settings |
| `known_nodes` | Liste des nœuds mesh connus |
| `my_info` | Informations brutes du nœud local |
| `metadata` | Métadonnées firmware |

---

## Nommage automatique des fichiers

```
meshtastic_[short_name]_[YYYYMMDD]_[HHMMSS].NBFM
```

Exemple : `meshtastic_JMC_5F7B_20260509_095500.NBFM`

---

## Ordre de restauration des canaux

Ordre canonique du CLI Meshtastic officiel (`setURL`) :

1. Canal primaire (role = 1) en premier (index 0)
2. Canaux secondaires (role = 2) ensuite
3. Canaux désactivés (role = 0) en dernier

Après le commit, NBFM relit les canaux et relance automatiquement tout canal actif silencieusement rejeté par l'appareil.

---

## Licence
GPLv3

---

## Auteur

**zifnab69** — ZIFNAB69_fr@yahoo.fr

## Information
Je ne suis absolument pas développeur de formation. Ce logiciel a été conçu avec l’aide d’une IA et beaucoup d’efforts de ma part pour le penser, le tester et l’améliorer. Si vous êtes développeur et que vous souhaitez contribuer, toute aide sera la bienvenue pour faire vivre et progresser ce projet.

---



# Nodes Backup & Fleet Manager
Outil Python/Tkinter pour sauvegarder, restaurer et déployer les configurations de nœuds Meshtastic via USB et sous windows. Export complet en JSON, génération de profils flotte (suppression des clés uniques à l'appareil), Conçu pour n’importe quel  nœuds .

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC_BY--NC_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
