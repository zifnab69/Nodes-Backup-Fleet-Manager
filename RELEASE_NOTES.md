# Nodes Backup & Fleet Manager — Notes de version / Release notes

> Texte de version prêt à coller dans une **GitHub Release** — tag **V1.96**.
> Il décrit les **fonctions utiles à connaître** qui ne sont **pas encore détaillées**
> sur la page des releases (https://github.com/zifnab69/Nodes-Backup-Fleet-Manager/releases).
>
> Numéro de version applicatif des fichiers exportés : **2.6** (schéma d'export interne).

---

## 🇫🇷 Français

### ✨ Nouveautés V1.96

- **Interface vraiment bilingue.** En mode anglais, certaines fenêtres restaient en français :
  confirmation de restauration, création de profil flotte, avertissements d'intégrité, fin de
  session multi-nœuds, message de dépendances manquantes… **Tous ces textes sont désormais
  traduits.**
- **Le compte rendu de restauration est traduit.** Le détail affiché à la fin d'un import
  (canaux écrits, sections et modules appliqués, avertissements) suit maintenant la langue
  choisie, alors qu'il était entièrement en français.
- **Listes déroulantes traduites.** Les descriptions des **régions LoRa** et des **presets modem**
  de l'éditeur s'affichent en anglais en mode anglais. Les codes officiels (`EU_868`, `LONG_FAST`…)
  restent inchangés.
- **Rapport HTML bilingue.** Le titre et les en-têtes de colonnes du rapport suivent la langue
  (le rapport était jusqu'ici uniquement en anglais).
- **Nom de fichier du profil flotte adapté à la langue** : `profil_flotte_…` en français,
  `fleet_profile_…` en anglais. Les deux restent **totalement interchangeables** : vos fichiers
  existants sont reconnus sans rien changer, quelle que soit la langue de l'interface.
- **Correction — preset modem « VeryLongSlow ».** Dans l'éditeur, choisir *VeryLongSlow*
  enregistrait en réalité *LongSlow*. C'est corrigé.
  ⚠ Les fichiers `.NBFM` créés **avant** cette version avec *VeryLongSlow* contiennent la mauvaise
  valeur : repassez dans l'éditeur si ce réglage comptait pour vous.

### 🧩 Rappel des nouveautés V1.95

- **Compatible petits écrans.** L'onglet principal dispose d'un **ascenseur global** : le bloc
  « Restaurer la configuration » et la barre d'état restent accessibles quelle que soit la hauteur
  de la fenêtre (avant, ils étaient tronqués en bas d'écran). L'ascenseur ne s'affiche que si le
  contenu ne tient pas ; en plein écran, l'affichage est identique aux versions précédentes.
- **Taille de fenêtre adaptée à l'écran.** La fenêtre principale et l'éditeur de champs clés
  s'ouvrent bornés à la hauteur disponible (utile sur les portables 1366×768).
- **Éditeur de champs clés utilisable en fenêtre réduite** : onglets « Principal » et « Canaux »
  défilables, boutons **Enregistrer / Enregistrer sous / Annuler** toujours visibles.
- **Molette souris plus prévisible** : elle agit sur la zone réellement survolée (liste de fichiers,
  aide, formulaires) au lieu de faire défiler l'onglet Aide en arrière-plan.

### 🧩 Rappel des nouveautés V1.9

- **Éditeur de champs clés en deux onglets** (« Principal » + « Canaux »). L'onglet Principal
  regroupe owner, LoRa, override duty cycle, rôle et multiplicateur ADC ; l'onglet Canaux gère
  les **8 canaux**.
- **Les 8 canaux éditables**, chacun avec une case **« Activé »** qui définit le rôle
  (le canal 0 reste primaire, verrouillé). Saisir un nom **auto-active** le canal, et les canaux
  actifs sont **tassés sans trou** à la sauvegarde. Correction du bug où un canal nommé restait
  désactivé (role=0).
- **Précision GPS par canal** (position_precision : NA / 23 km / … / 1 m) et **multiplicateur ADC**
  avec presets par appareil.
- **Injection des canaux fiabilisée** : ordre canonique (primaire d'abord, aligné sur `setURL`),
  **vérification post-commit** (relecture des canaux) et **relance automatique** des canaux
  silencieusement rejetés par l'appareil (nœuds non vierges / clé PKI). Testé sur Heltec V4 /
  firmware 2.7.x.
- **Validation de la longueur des PSK** dans l'éditeur (1/16/32 octets) — bloque une clé invalide
  avant qu'elle ne soit rejetée en silence par le firmware.
- **Visualiseur JSON éditable** : case « ✏ Éditer » + bouton « 💾 Enregistrer » (validation JSON
  stricte, copie horodatée dans `Backup/` avant écrasement).
- **Journaux d'import copiables** (copier/coller du détail de restauration).

### 🧩 Rappel des nouveautés récentes (V1.8)

- **Barre de progression de la restauration.** Fenêtre de progression étape par étape
  (propriétaire, sections, modules, canaux, validation finale), en mono comme en multi-nœuds.
- **Restauration fiable des clés de chiffrement des canaux (PSK).** Les clés exportées en Base64
  sont restaurées correctement (hex ET Base64 acceptés). Auparavant, restaurer une sauvegarde
  standard pouvait vider les PSK de certains canaux (les noms revenaient, pas les clés). Corrigé.
- **Persistance de la langue et du dossier de travail** (`NBFM_Config.json`).

### 🔑 Fonctions à connaître (déjà présentes mais non listées sur la page des releases)

- **Sauvegarde fidèle à 100 %.** L'export enregistre *toute* la configuration du nœud : owner,
  LoRa, Bluetooth, réseau, position, puissance, affichage, sécurité, **tous les modules** (même
  ceux que l'interface n'expose pas), canaux et nœuds connus.
- **Restauration exhaustive.** L'import réécrit *toutes* les sections et *tous* les modules présents
  dans le fichier — rien n'est ignoré en silence.
- **Profil flotte déployable.** Génère une configuration épurée (clés uniques, identifiant, owner,
  nœuds, identifiants WiFi retirés) tout en **conservant** `admin_key`, la config LoRa, les canaux
  et leurs PSK, les modules — prête à déployer sur toute une flotte.
- **Export ET restauration multi-nœuds en série.** Traitez plusieurs appareils à la suite dans une
  même session, avec sélection du port à chaque étape (COM1 exclu automatiquement).
- **Éditeur de champs clés.** Modifiez sans éditeur externe : nom long/court, région LoRa, modem
  preset, **fréquence override (MHz)**, **override duty cycle** (limite légale 1 % EU868), rôle de
  l'appareil, noms et clés PSK des canaux 0-2.
- **Générateur de clés PSK** intégré (Défaut / AES-128 / AES-256), copie en un clic.
- **Nettoyage avancé** dans l'éditeur : supprimer tous les canaux (garder le canal par défaut),
  supprimer le paramètre ADC, supprimer les nœuds connus.
- **Validation d'intégrité avant restauration** : le fichier est vérifié (présence de `local_config`,
  `lora`, `channels`, cohérence région/modem) et vous êtes averti avant d'écraser un appareil.
- **Interface bilingue FR / EN**, commutable à chaud sans redémarrer l'application.
- **Format ouvert.** Un fichier `.NBFM` est un JSON lisible et éditable avec n'importe quel éditeur
  de texte.
- **Exécutable autonome.** Compilation possible en `.exe` Windows via PyInstaller (aucune
  installation Python requise pour l'utilisateur final).

### ⚠ Rappels d'usage

- Après toute restauration, **redémarrez l'appareil** pour appliquer la configuration.
- **Exportez toujours avant** de modifier un fichier ou d'appliquer un profil flotte.
- Câble **USB DATA** requis (pas un câble de charge seul) ; pilotes CP210x / CH340 installés.
- Le paramétrage fin du firmware se fait sur https://client.meshtastic.org — NBFM est un outil de
  **sauvegarde, restauration et déploiement**, pas un configurateur complet.

---

## 🇬🇧 English

### ✨ What's new in V1.96

- **A genuinely bilingual interface.** In English mode, several dialogs were still showing French:
  restore confirmation, fleet profile creation, integrity warnings, end of a multi-node session,
  missing-dependency message… **All of these are now translated.**
- **The restore report is translated.** The details shown at the end of an import (channels written,
  sections and modules applied, warnings) now follow the selected language — it used to be entirely
  in French.
- **Translated drop-down lists.** The **LoRa region** and **modem preset** descriptions in the editor
  now read in English when the app is in English. The official codes (`EU_868`, `LONG_FAST`…) are
  unchanged.
- **Bilingual HTML report.** The report title and column headers follow the selected language
  (the report used to be English-only).
- **Fleet profile file name follows the language**: `profil_flotte_…` in French,
  `fleet_profile_…` in English. Both remain **fully interchangeable**: your existing files are
  recognised as before, whatever the interface language.
- **Fix — "VeryLongSlow" modem preset.** In the editor, picking *VeryLongSlow* actually saved
  *LongSlow*. This is now fixed.
  ⚠ `.NBFM` files created **before** this version with *VeryLongSlow* hold the wrong value —
  reopen them in the editor if that setting mattered to you.

### 🧩 Recap of V1.95

- **Small-screen friendly.** The main tab now has a **global scrollbar**: the "Restore configuration"
  block and the status bar stay reachable whatever the window height (they used to be cut off at the
  bottom). The scrollbar only appears when the content does not fit; on a full-size window the layout
  is identical to previous versions.
- **Window size fitted to the screen.** The main window and the key-fields editor open bounded to the
  available height (handy on 1366×768 laptops).
- **Key-fields editor usable in a small window**: scrollable "Main" and "Channels" tabs, with
  **Save / Save as / Cancel** buttons always visible.
- **More predictable mouse wheel**: it scrolls the area actually under the cursor (file list, help,
  forms) instead of scrolling the Help tab in the background.

### 🧩 Recap of V1.9

- **Two-tab key fields editor** ("Main" + "Channels"). The Main tab groups owner, LoRa, override
  duty cycle, role and ADC multiplier; the Channels tab manages all **8 channels**.
- **All 8 channels editable**, each with an **"Enabled"** checkbox that sets the role (channel 0
  stays primary, locked). Typing a name **auto-enables** the channel, and active channels are
  **compacted without gaps** on save. Fixes the bug where a named channel stayed disabled (role=0).
- **Per-channel GPS precision** (position_precision: NA / 23 km / … / 1 m) and **ADC multiplier**
  with per-device presets.
- **Hardened channel injection**: canonical order (primary first, aligned with `setURL`),
  **post-commit verification** (channel re-read) and **automatic retry** of channels silently
  rejected by the device (non-blank nodes / PKI key). Tested on Heltec V4 / firmware 2.7.x.
- **PSK length validation** in the editor (1/16/32 bytes) — blocks an invalid key before the
  firmware silently rejects it.
- **Editable raw-JSON viewer**: "✏ Edit" checkbox + "💾 Save" button (strict JSON validation,
  timestamped copy in `Backup/` before overwriting).
- **Copyable import logs** (copy/paste the restore details).

### 🧩 Recent additions recap (V1.8)

- **Restore progress bar.** Step-by-step progress window (owner, sections, modules, channels, final
  commit), for single-node and multi-node restores.
- **Reliable channel encryption key (PSK) restore.** Keys exported in Base64 are restored correctly
  (both hex and Base64 accepted). Previously, restoring a standard backup could wipe some channels'
  PSKs (names came back, keys did not). Fixed.
- **Language and working-directory persistence** (`NBFM_Config.json`).

### 🔑 Functions worth knowing (already present but not listed on the releases page)

- **100 % faithful backup.** Export saves the *entire* node configuration: owner, LoRa, Bluetooth,
  network, position, power, display, security, **all modules** (even those the UI does not expose),
  channels and known nodes.
- **Exhaustive restore.** Import rewrites *all* sections and *all* modules found in the file — nothing
  is silently skipped.
- **Deployable fleet profile.** Generates a trimmed configuration (unique keys, ID, owner, nodes, WiFi
  credentials removed) while **keeping** `admin_key`, LoRa config, channels and their PSKs, modules —
  ready to deploy across a whole fleet.
- **Multi-node sequential export AND restore.** Process several devices in a row within one session,
  selecting the port at each step (COM1 automatically excluded).
- **Key fields editor.** Edit without an external tool: long/short name, LoRa region, modem preset,
  **override frequency (MHz)**, **override duty cycle** (EU868 1 % legal limit), device role, channel
  0-2 names and PSK keys.
- **Built-in PSK key generator** (Default / AES-128 / AES-256), one-click copy.
- **Advanced cleanup** in the editor: clear all channels (keep the default channel), clear the ADC
  setting, clear known nodes.
- **Integrity check before restore**: the file is validated (presence of `local_config`, `lora`,
  `channels`, region/modem consistency) and you are warned before overwriting a device.
- **Bilingual FR / EN interface**, switchable on the fly without restarting the app.
- **Open format.** A `.NBFM` file is readable JSON, editable with any text editor.
- **Standalone executable.** Can be compiled to a Windows `.exe` via PyInstaller (no Python install
  required for the end user).

### ⚠ Usage reminders

- After any restore, **restart the device** to apply the configuration.
- **Always export first** before editing a file or applying a fleet profile.
- A **USB DATA** cable is required (not charge-only); CP210x / CH340 drivers installed.
- Fine firmware tuning is done on https://client.meshtastic.org — NBFM is a **backup, restore and
  deployment** tool, not a full configurator.
