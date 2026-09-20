# CLAUDE.md — Nodes Backup & Fleet Manager

> Mémo technique pour Claude. Référence obligatoire avant toute modification du code.

---

## Table des matières

- [**État du projet — 20 septembre 2026 (lire en premier)**](#état-du-projet--20-septembre-2026-lire-en-premier)
1. [Objectif global](#1-objectif-global)
2. [Contexte technique](#2-contexte-technique)
3. [Structure du projet](#3-structure-du-projet)
4. [Workflow d'exécution](#4-workflow-dexécution)
5. [Format NBFM réel](#5-format-nbfm-réel--observations-sur-fichier-de-référence)
6. [Architecture du code](#6-architecture-du-code)
7. [Conventions de code](#7-conventions-de-code)
8. [Contraintes spécifiques](#8-contraintes-spécifiques)
9. [Notes API Meshtastic Python](#9-notes-api-meshtastic-python--faits-confirmés)
10. [Roadmap / TODO](#10-roadmap--todo)

---

## État du projet — 20 septembre 2026 (lire en premier)

### ▶ POUR REPRENDRE LA PROCHAINE FOIS

- **Fichier de travail actif** : `NBFM_V1.96.py` (= `NBFM_20260920_1704.py` renommé pour partage ; ~4320 lignes, fichier unique). Compile OK. Fonctionnel sur matériel réel (T-Echo, Heltec V3, **Heltec V4**). Le nommage de travail reste `NBFM_YYYYMMDD_HHMM.py` ; un `NBFM_Vx.y.py` est un renommage pour le partage. Prédécesseur `NBFM_V1.95.py` dans `Backup/`. **Avant la prochaine modif** : copier l'actif dans `Backup/` puis le renommer en `NBFM_YYYYMMDD_HHMM.py` (protocole de versioning).
- ⚠ **`NBFM_V1.95.exe` à la racine du dépôt est périmé** (compilé depuis la v1.95) — à reconstruire via PyInstaller et à remplacer lors de la publication de la release.
- **Version affichée : v1.96** (docstring d'en-tête, titre de fenêtre, en-tête d'écran, pied de l'onglet Aide — 4 occurrences, à bumper ensemble). À ne pas confondre avec `_app_version` (= `"2.6"`), qui versionne le **format de fichier** `.NBFM` et n'a pas bougé.
- **État** : application stable. **Aucun bug bloquant.** Tous les bugs A, B, C, E–M sont corrigés et validés. **Seul Bug D reste ouvert** (basse priorité, neutralisé — voir §Bug D).
- **Avant toute modif** : appliquer le protocole de versioning (timestamp FR → copie dans `Backup/` → renommer en `NBFM_YYYYMMDD_HHMM.py` → modifier → `py_compile`).
- **Environnement critique** : **protobuf 7.34.1 / Python 3.14** (voir Pièges). Toujours tester en intégration avec un faux nœud + vrais protos `localonly_pb2`/`channel_pb2`.
- **⚠ Décision d'architecture MODIFIÉE (1519)** : l'ordre d'écriture des canaux est passé de « primaire en DERNIER » à « **primaire en PREMIER** » (aligné sur `setURL` du CLI Meshtastic). Voir §Pièges et §8/§9.
- **Test GUI possible sans écran** : Tkinter fonctionne en headless dans l'env de dev. On smoke-teste une fenêtre en construisant la méthode avec un faux `self` (attributs `root/lang_var/work_dir/_get_selected_file/refresh_files/set_status/_edit_popup_refresh`), puis `.invoke()` sur les boutons + relecture du fichier produit (a validé l'éditeur en onglets : build + save).
- **Session 12/07/2026** : injection canaux robuste (ordre + relance), éditeur 2 onglets (canaux activables + précision GPS + ADC multiplier), auto-activation au nom + tassement des canaux, `view_file` éditable, journaux d'import copiables, fix `connect_device`. Tous validés (compile + smoke-tests headless). Commit/renommage effectué par l'utilisateur (GitHub Desktop).
- **Session 20/09/2026** : **bilinguisme complet** — toutes les chaînes affichées passent désormais par `UI_STRINGS`/`tr()` (détail dans « Travaux récents », règles dans §7 « Langue et UI »). Aucune autre fonction touchée.

### ★ Pistes d'évolution proposées (rappel demandé par l'utilisateur)

À proposer / faire quand l'utilisateur le souhaite (aucune urgente) :

| Piste | Priorité | Note |
|---|---|---|
| ✅ Validation longueur PSK dans l'éditeur | — | **FAIT (0001)** — bloque la sauvegarde si la clé Base64 ≠ 1/16/32 octets |
| Rapport CSV (en plus du HTML) | moyenne | Réutiliser la génération HTML existante |
| Firmware dans la bulle de survol | moyenne | Champ `metadata`, sans nouvelle colonne |
| Support YAML natif à l'import | moyenne | Le glob accepte déjà `*.yaml/*.yml` mais `import_full_config` ne lit que JSON |
| Doc PyInstaller dans le README | moyenne | Commande exacte + `--icon`, `--windowed` |
| Icône d'application pour l'EXE | basse | `--icon=nbfm.ico` |
| Auto-détection port par VID/PID (CP210x/CH340) | basse | Pré-sélection plus fine |
| Mode ligne de commande (sans GUI) | basse | Automatisation de flotte |
| ✅ Bug M — preset modem `VERY_LONG_SLOW` | — | **FAIT (1704)** — comparaison exacte du code en tête de libellé |
| Bug D — `statusmessage` inscriptible | basse | Voie alternative API à investiguer |

### Travaux récents (résumé)

- **Bug M + nettoyage Ruff (20260920_1704)** : correction de `_modem_int_from_label` (voir §Bug M) et des 28 remarques Ruff **hors convention projet** :
  - **TRY002** (4) : nouvelle classe `NBFMError(Exception)` en tête de fichier, utilisée par `connect_device` (3 sites) et la sentinelle interne `"vide"` de `export_full_config`. Tous les appelants font `except Exception` → comportement strictement identique, seul le type se précise.
  - **DTZ005** (9) : les **7** sites `strftime` (noms de fichiers, affichage) passent en `datetime.now().astimezone()` — sortie identique au caractère près, vérifié. Les **2** sites qui écrivent `_export_date` / `_profile_date` **dans le .NBFM** gardent volontairement l'heure locale **naïve** (`# noqa: DTZ005` + commentaire) : y ajouter un décalage changerait le format de fichier documenté au §5. Décision validée avec l'utilisateur.
  - **F401** (2) : `import … , time` redondant retiré de `connect_device` (le module l'importe déjà) ; l'import `ParseDict` d'`import_full_config` est **conservé** avec `# noqa: F401` — il ne sert pas au code mais de **sonde de disponibilité** du stack protobuf, le retirer casserait le garde `except ImportError`.
  - **F841** (2) : `cur` (dans `_apply_lang`) et `grp_iid` (dans `refresh_files`) étaient assignés sans être lus ; l'appel `tree.insert(...)` est conservé, seule l'affectation morte disparaît.
  - **C414** (1) `sorted(list(glob))` → `sorted(glob)` · **SIM102** (1) `if warns:` / `if not askyesno` fusionnés en `and` (court-circuit ⇒ `askyesno` toujours appelé seulement si `warns`) · **I001** (4) blocs d'imports triés · **UP006/UP045/UP035** (5) `Dict[str, Any]` → `dict[str, Any]`, `Optional[X]` → `X | None` (OK dès Python 3.10, cible du projet).
  - **NON corrigé, volontairement** : les **108 `BLE001`** (blind-except) et **43 `S110`** (try-except-pass) restants. C'est la convention explicite du §7 « Gestion des erreurs » (*broad `try/except Exception` partout — priorité à la robustesse sur la précision*, *ne jamais supprimer un fallback existant*). Les corriger contredirait la règle n°1 « zéro régression ». **Un `ruff check` sans `--select` renverra donc toujours 151 remarques : c'est normal, ne pas les « corriger ».** La vérification de référence est `--select E9,F63,F7,F82`.
- **Nom du profil flotte bilingue + v1.96 (20260920_1704)** : le nom suggéré à la création d'un profil flotte suit désormais la langue — nouvelle clé `fleet_filename` (`profil_flotte_{date}.NBFM` en FR, `fleet_profile_{date}.NBFM` en EN), utilisée dans `generate_fleet_profile`. **Les deux conventions sont totalement interchangeables** : le type d'un fichier vient de `_profile_type` dans le JSON, **jamais** de son nom — aucun code ne teste le préfixe. Vérifié par test : un `profil_flotte_*.NBFM` s'affiche « 🚀 Fleet » en anglais, un `fleet_profile_*.NBFM` s'affiche « 🚀 Flotte » en français, et les deux déclenchent l'alerte « déjà un profil flotte » dans les deux langues. Version affichée passée à **v1.96**. Ruff `E9,F63,F7,F82` : `All checks passed!`.
- **Bilinguisme complet FR/EN (20260920_1704)** : un utilisateur signalait des popups restés en français en mode anglais. Cause : de nombreuses chaînes affichées étaient codées en dur en FR **alors que la clé `UI_STRINGS` existait déjà** (`popup_restore_confirm_text`, `popup_fleet_created_text`, `popup_multi_import_done_text`, `deps_missing_*`…) — elles n'étaient simplement pas utilisées. Corrigé partout, sans toucher à la logique :
  - **Cœur (hors UI)** : `check_dependencies`, `_apply_security_to_node`, `_apply_section_to_node`, `_apply_module_section` et **tout le journal de `import_full_config`** passent par `tr()`. ~40 nouvelles clés `log_*` (préfixes `log_section_`, `log_module_`, `log_tx_`, `log_owner_`, `log_ch_`, `log_sec_`).
  - **UI** : libellés de type (`🚀 Flotte`/`💾 Backup`), en-tête de groupe « Autres », note « COM1 exclu », filtre de fichiers « Tous », popups export/flotte/intégrité/confirmation de restauration/session multi-nœuds, en-têtes des journaux copiables, avertissement « Redémarrez l'appareil ».
  - **Rapport HTML** : titre, ligne « Généré », les 13 en-têtes de colonnes et l'attribut `<html lang=…>` suivent la langue (il était entièrement en anglais dans les deux langues).
  - **Régions LoRa / presets modem** : `LORA_REGIONS`/`MODEM_PRESETS` (FR) restent la **source de vérité** des codes et de l'ordre ; deux tables `LORA_REGIONS_EN`/`MODEM_PRESETS_EN` ont été ajoutées **à côté**, plus `_region_desc(i)`/`_modem_desc(i)` qui choisissent selon `load_lang()`. Seule la partie descriptive du libellé change : le code (`EU_868`) et l'index (`[3]`) restent intacts, donc `_region_int_from_label`/`_modem_int_from_label` fonctionnent dans les deux langues.
  - **Piège évité** : dans `_do_import`, la lecture de `self.lang_var` reste **dans le `lambda`** passé à `root.after` — jamais dans le corps du thread de travail (règle Threading du §7).
  - **Piège f-string** : dans le bloc HTML `f"""…"""`, les expressions utilisent des guillemets **simples** (`{T['report_col_type']}`) — les guillemets doubles imbriqués ne sont valides qu'à partir de Python 3.12 alors que le projet cible 3.10+.
  - **Non touché volontairement** : `_profile_note` de `build_fleet_profile` (écrit **dans le fichier** .NBFM — le localiser rendrait le format dépendant de la langue), le sentinelle interne `tag` (`🚀FLOTTE`/`💾BACKUP` de `read_file_meta`, jamais affiché, comparé par `startswith`/`in`), l'id de colonne Treeview `"fichier"`, et les noms de fichiers suggérés (`profil_flotte_…`, `_copie_`) qui sont une convention documentée au §7.
  - **Validation** : `py_compile` OK ; parité des 294 clés FR/EN + placeholders identiques ; les 242 clés référencées dans le code existent dans les deux langues ; **journal d'import FR identique octet pour octet à v1.95** (test d'intégration faux nœud + vrais protos `localonly_pb2`/`channel_pb2`, cas nominal ET rejet silencieux d'un secondaire), libellés région/modem FR identiques à v1.95 sur toutes les valeurs y compris hors bornes, textes de popups FR identiques à v1.95, smoke-test GUI headless FR+EN (liste, confirmation de restauration, profil flotte, rapport HTML, éditeur : ouverture + sauvegarde).
- **Ascenseur global / petits écrans (20260812_1321)** : sur écran bas, le bloc « Restaurer » (packé en dernier) disparaissait — pack sert les widgets dans l'ordre de déclaration. Trois helpers dans `NBFMApp` : `_fit_to_screen(win, w, h)` (borne la géométrie à l'écran, ne l'agrandit jamais — appliqué à la fenêtre principale et à l'éditeur), `_make_scrollable(parent)` → `(outer, inner)` (Canvas + Scrollbar ; l'ascenseur n'apparaît QUE si `inner.reqheight > canvas.height`, sinon `inner` est étiré à la hauteur du canvas → l'`expand=True` de la liste de fichiers se comporte comme avant), `_bind_mousewheel_global()` (UN seul `bind_all("<MouseWheel>")` : remonte la hiérarchie depuis le widget survolé, ignore Treeview/Text/Listbox qui défilent seuls, sinon défile le premier Canvas marqué `_nbfm_scroll`). Appliqué à : onglet principal (barre de statut sortie de la zone défilante et packée `before=` le conteneur pour être servie en premier), onglets Principal/Canaux de l'éditeur (le notebook reçoit `page_main`/`page_chan`, le contenu va dans les frames internes `tab_main`/`tab_chan` — `nb.tab()` doit viser les `page_*`), et l'onglet Aide (son ancien `bind_all` local défilait l'aide en arrière-plan quand on scrollait ailleurs — supprimé). **Éditeur** : `frm_cleanup` + `btn_row` créés AVANT le notebook et packés `side="bottom"` → boutons Enregistrer/Annuler toujours visibles (validé jusqu'à 300 px de haut). Smoke-tests headless : ascenseur présent à 420/600 px, absent à 900 px ; save de l'éditeur toujours fonctionnel.
- **Éditeur en onglets + canal « activable » (1519b)** : `edit_config_fields` passé en `ttk.Notebook` 2 onglets. **Onglet Canaux** : les 8 canaux, chacun avec case **« Activé »** (→ `role`=SECONDARY si coché, DISABLED sinon ; canal 0 = PRIMARY verrouillé), nom, PSK, **précision GPS** compacte (`module_settings.position_precision` via `POSITION_PRECISION` NA/23km…/1m), + générateur de clés. **Onglet Principal** : owner/LoRa/rôle + **ADC multiplier** (champ éditable + menu `ADC_DEFAULTS` par appareil ; vide = clé supprimée). **Bug corrigé** : l'ancien éditeur écrivait nom+PSK d'un canal mais JAMAIS son `role` → un canal nommé restait `role=0` (désactivé sur l'appareil). Désormais le save reconstruit les 8 canaux avec le bon rôle. `save_and_close` : rebuild 8 canaux (préserve champs annexes), validation PSK/doublons sur 8 canaux, écriture ADC. Case « Supprimer réglages puissance (ADC) » retirée (doublon).
- **Auto-activation + tassement des canaux (1519c)** : deux correctifs liés au rôle des canaux. (1) **Auto-cocher « Activé » quand on tape un nom** (`nm_var.trace_add`) — sinon un nom saisi dans une ligne vide restait décoché → `role=0` → canal importé « désactivé » (cause d'un bug remonté : canal « bidule » nommé mais désactivé, PSK en hex = signature de l'éditeur). (2) **Tassement (compaction) au save** façon `deleteChannel` Meshtastic : primaire en 0, secondaires ACTIVÉS packés en 1,2,3… **sans trou**, reste en DISABLED vide. Décocher un canal du milieu ne laisse donc jamais de trou (un trou = canal désactivé au milieu d'actifs = non standard, canaux suivants potentiellement masqués). Validé par smoke-tests headless (build + save → canal nommé ressort role=2 ; trou supprimé ; saisie d'un nom auto-active + packe).
- **Robustesse injection canaux (1519, Heltec V4 / firmware 2.7.26)** : sur nœud non vierge, un canal secondaire pouvait ne pas être injecté (writeChannel accepté par la lib mais rejeté en silence côté device — clé de session PKI). Trois changements dans `import_full_config` : (1) **ordre canonique** — tri `!= 1` → primaire (index 0) écrit EN PREMIER puis secondaires puis désactivés (avant : primaire en dernier) ; (2) **vérification post-commit** — relecture `requestChannels()` (poll borné 8 s, jamais `waitForConfig` qui a un timeout 300 s) + comparaison via `_channel_applied()` (rôle/nom/PSK) ; (3) **relance directe** hors transaction (une passe) des canaux non appliqués, sinon message « reset usine conseillé ». Registre `_ch_written` = {index: Channel clone} des canaux actifs. Entièrement gardé (zéro régression). Validé par test d'intégration (faux nœud + `channel_pb2`, simulation rejet silencieux).
- **`view_file` éditable (1519)** : checkbox « ✏ Éditer » → dégrouille le `ScrolledText` + bouton « 💾 Enregistrer » ; validation `json.loads` stricte avant écriture (refus si JSON invalide), copie horodatée dans `Backup/` avant écrasement.
- **Journal d'import copiable (1519)** : nouveau helper `_show_copyable_log(title, header, log_text, warn)` (Toplevel + `ScrolledText` + bouton « 📋 Copier ») remplace les `messagebox.showinfo` d'import (mono + multi) qui ne permettaient pas le copier-coller.
- **`connect_device` — code mort corrigé (1519)** : `isConnected` est un `threading.Event` (pas un bool) → l'ancien `not getattr(iface,"isConnected",False)` était toujours faux (attente/timeout jamais exécutés). Remplacé par `ev.wait(8)` (+ repli bool). Le constructeur `SerialInterface` bloquait déjà, donc pas de régression, mais le timeout explicite est désormais réel.
- **Validation longueur PSK (0001)** : éditeur — `_validate_psk_b64()` dans `save_and_close` bloque la sauvegarde et avertit si une clé Base64 ne décode pas en 1/16/32 octets (avant : longueur non vérifiée → rejet silencieux du firmware).
- **Persistance langue + dossier (2348, Bugs A/B)** : `save_lang()` au démarrage dans `__init__` (crée `NBFM_Config.json` même en anglais) ; helpers `load_work_dir()`/`save_work_dir()`, lecture au démarrage, persistance dans `choose_work_dir()` + 3 chemins export. **Port COM non persisté.**
- **Barre de progression restauration (2332)** : `import_full_config(iface, config, progress=cb)` ; callback `progress(done,total,kind,detail)` ; UI `_open_progress(threaded=...)` + `_progress_label()` (mono-nœud=thread+`root.after` ; multi-nœuds=synchrone+`win.update()`).
- **Aide in-app FR+EN à jour** + **`RELEASE_NOTES.md`** (notes de version GitHub bilingues : fonctions absentes de la page des releases).
- **Bug L (PSK base64 vs hex) corrigé (2308, validé matériel réel)** : l'export stocke la PSK en base64 mais l'import la décodait en hex (`bytes.fromhex`) → clés perdues. Fix : `_psk_str_to_bytes()` tolère hex ET base64 (contrôle de longueur 1/16/32 o pour désambiguïser), utilisé dans les 2 chemins d'import canaux.

### Script actif

`NBFM_V1.96.py` (= `NBFM_20260920_1704.py` renommé pour partage ; ~4320 lignes, fichier unique).  
Backup du prédécesseur dans `Backup/` : `NBFM_V1.95.py`, plus la copie horodatée `NBFM_20260920_1704.py`. Anciennes versions numérotées dans `Old release/`.

### Pièges & leçons techniques durables (À LIRE avant de toucher export/import)
- **protobuf 7.x (upb)** : ne JAMAIS supposer l'API descriptor. Utiliser `field.is_repeated` (pas `.label`).
  MessageToDict : `always_print_fields_with_no_presence` + `use_integers_for_enums` impératifs.
- **`our_exit()` de la lib Meshtastic → `sys.exit()` → `SystemExit`** (hérite de BaseException). `except
  Exception` ne l'attrape PAS. Tout appel lib peut tuer le thread silencieusement.
- **Clé de session admin rotative** (firmware + admin_key/PKI) : encadrer les écritures par une transaction
  et espacer (0,5 s) pour laisser la clé se rafraîchir.
- **TOUJOURS tester en intégration** avec faux nœud + vrais protos `localonly_pb2`/`channel_pb2` sous le
  protobuf réellement installé. Les tests unitaires isolés masquent les régressions (cf. `re` non importé).
- **Auto-reboot** : le device redémarre seul à la fin du download. Le message « redémarrez l'appareil »
  est conservé volontairement (inoffensif, rassure l'utilisateur). Pas de changement.
- **Reboot après commit ⇒ relecture canaux impossible** : après `commitSettingsTransaction`, l'appareil
  redémarre souvent → la vérification post-commit (`requestChannels`, poll 8 s) échoue (« vérification auto
  impossible »). Ce n'est PAS une erreur : les canaux sont écrits ; la vérif/relance auto ne fonctionne que
  si l'appareil ne reboote pas. Le correctif de fond reste l'ordre canonique (primaire d'abord).
- **Canal par défaut = nom VIDE** (vérifié dans la lib) : l'identité d'un canal = `generate_channel_hash(name, psk)`.
  Le primaire standard a `name=""` (l'app affiche le nom du preset, ex « LongFast ») et `psk=0x01`/`AQ==`
  (= clé par défaut, cf. `util.py:68 bytes([1])`). **Le nommer « default » changerait le hash → incompatibilité.**
  Donc `clear_channels` garde `name=""` : c'est correct, NE PAS mettre « default ».

### Ce que fait le logiciel — ce qui fonctionne

L'application est **fonctionnelle et utilisée sur matériel réel** (T-Echo, Heltec V3). Tout est ✅ fonctionnel :

- Export complet d'un nœud → fichier `.NBFM` (tous les champs + tous les modules, même inconnus)
- Restauration `.NBFM` → nœud (lora, canaux, owner, PSK, transaction admin) **avec barre de progression**
- Génération de profil flotte (épure les clés uniques, conserve `admin_key` + LoRa + canaux)
- Export / restauration multi-nœuds séquentiels
- Éditeur de champs clés : owner, région LoRa, modem, fréquence override, override_duty_cycle, rôle, canaux 0-2 (nom + PSK) — **avec validation de longueur PSK**
- Générateur de clés PSK (AES-128 / AES-256), « supprimer tous les canaux », nettoyage ADC / known_nodes
- Liste : groupement par MAC, tri, renommage (double-clic), menu contextuel, bulles de survol, notes par fichier
- Rapport HTML, validation d'intégrité avant restauration
- UI bilingue FR/EN à chaud, persistance langue + dossier de travail (`NBFM_Config.json`)
- Compilation EXE via PyInstaller

### Bugs — état

Tous les bugs historiques sont **résolus** : A, B (persistance langue/dossier), C (suppression known_nodes),
E (LoRa no-op / protobuf 7.x), F (bruit `statusmessage`), G (clé session admin / transaction), H/I (enums →
tableau + ordre canaux), J (PSK effaçable), K (noms canaux en double), L (PSK base64 vs hex),
M (preset modem `VERY_LONG_SLOW`). **Seul Bug D reste ouvert** (basse priorité, neutralisé) :

#### Bug D — `statusmessage` non inscriptible (priorité basse — neutralisé, plus bloquant)
**État (mis à jour 30/05)** : le proto `moduleConfig.statusmessage` EXISTE désormais (protobuf récent), mais
`writeConfig` de la lib ne le gère pas → `our_exit()` → SystemExit. C'est attrapé (`_apply_module_section`)
et le `print` parasite est avalé (`_write_config_quiet`). Donc **plus de crash ni de bruit shell** ; le module
est simplement ignoré (ligne `⚠` dans le popup). Reste théoriquement non restaurable via l'API Python standard.
**À investiguer un jour** : voie alternative pour écrire `statusmessage` (peut-être un module interne non exposé).

#### Bug M — preset modem `VERY_LONG_SLOW` mal relu dans l'éditeur — ✅ CORRIGÉ (1704)
**Découvert le 20/09/2026**, **présent à l'identique dans v1.95 et antérieures** (donc pas une régression du
bilinguisme — juste mis en évidence par les tests d'aller-retour ajoutés). `_modem_int_from_label()` testait
`if code in label` en parcourant `MODEM_PRESETS` dans l'ordre 0→8 : pour le libellé `VERY_LONG_SLOW    — …`,
le code `LONG_SLOW` (index **1**) est une sous-chaîne de `VERY_LONG_SLOW` et matchait **avant** l'index 2.
**Conséquence** : choisir « VeryLongSlow » dans l'éditeur enregistrait `modem_preset = 1` (LongSlow).
**Correctif appliqué** : `_modem_labels()` construit le libellé sous la forme `"<CODE>    — <desc>"`, donc le
code est le PREMIER token → comparaison **exacte** sur `label.split(" ", 1)[0]`. Le repli en sous-chaîne est
**conservé** (libellé d'autre provenance, tronqué…) mais parcourt désormais les codes **du plus long au plus
court**, pour que `VERY_LONG_SLOW` soit testé avant `LONG_SLOW`.
**Validé** : aller-retour `_modem_label_from_int` → `_modem_int_from_label` correct sur les 9 presets, en FR
ET en EN, plus les cas de repli (`"VERY_LONG_SLOW"` nu, libellé noyé dans une phrase, chaîne vide).
⚠ **Les fichiers .NBFM créés avant ce correctif avec « VeryLongSlow » contiennent `modem_preset = 1`** :
rien ne les corrige automatiquement, il faut repasser dans l'éditeur si le preset comptait.

### Règles non négociables pour toute modification

1. **Zéro régression** — tout changement préserve le comportement existant. En cas de doute, ajouter *à côté*, pas à la place.
2. **Import exhaustif** — jamais de liste blanche de modules/sections. Tout ce qui est dans le JSON doit être restauré.
3. **NBFM = backup/déploiement, pas configurateur** — le paramétrage source reste sur https://client.meshtastic.org.
4. **Nommage** — tout nouveau fichier Python : `NBFM_YYYYMMDD_HHMM.py`.

### Fichier de référence matériel disponible

`meshtastic_GB_A7F9_20260525_094518.nbfm` — export réel d'un T-Echo (EU868, 869.5 MHz, MediumSlow).  
Utiliser ce fichier pour valider tout développement sur le parsing/import/export.  
Contient : 2 canaux actifs (AES-256), 6 nœuds connus, modules `statusmessage` + `traffic_management` + `audio` + `remote_hardware`, `adc_multiplier_override: 2.0`.

---

## 1. Objectif global

Application desktop **Python/Tkinter** pour **sauvegarder, restaurer et déployer des configurations de nœuds Meshtastic** via USB, sous Windows uniquement.

**Public cible** : utilisateurs Meshtastic (radioamateurs, équipes de communication d'urgence) qui veulent gérer plusieurs nœuds ou déployer une config commune en flotte.

### Philosophie de conception — à garder en tête en permanence

Le paramétrage fin du matériel Meshtastic se fait via le client web officiel
(**https://client.meshtastic.org**), qui expose toutes les fonctionnalités à jour des firmwares.
**NBFM n'a pas vocation à remplacer ce site.**

Le rôle de NBFM est strictement :
1. **Sauvegarder** fidèlement 100 % de la configuration d'un nœud
2. **Restaurer** fidèlement 100 % de cette configuration, y compris les sections que l'UI n'expose pas
3. **Générer des profils flotte** (config commune déployable sur plusieurs nœuds)
4. **Modifier à la marge** quelques paramètres courants (région LoRa, canaux, owner, PSK…)

> **Conséquence directe pour le code** : l'export et l'import doivent traiter *tous* les champs
> et *tous* les modules présents dans le JSON — même les inconnus. Ne jamais ignorer
> silencieusement une section sous prétexte que l'UI ne l'expose pas.
> Un paramètre non géré par l'UI doit quand même être sauvegardé et restauré.

**Fonctionnalités principales** :
- Export complet de la config d'un nœud → fichier JSON (.NBFM)
- Restauration complète d'un fichier .NBFM → nœud (tous les champs, sans exception)
- Génération de profil flotte (suppression des clés uniques, conservation de l'admin_key et de la config LoRa/canaux)
- Export/import multi-nœuds séquentiels dans une même session
- Éditeur intégré : région LoRa, modem preset, noms de canaux, clés PSK, rôle appareil
- Générateur de clés PSK (AES-128 / AES-256)
- UI bilingue FR/EN commutable sans redémarrage
- Rapport HTML exportable de tous les fichiers NBFM
- Compilation en EXE autonome via PyInstaller

**Auteur** : zifnab69 — ZIFNAB69_fr@yahoo.fr  
**Licence** : CC BY-NC-SA 4.0

---

## 2. Contexte technique

| Paramètre | Valeur |
|---|---|
| Langage | Python 3.10+ |
| GUI | tkinter + ttk (stdlib) |
| Dépendances runtime | `meshtastic`, `pyserial`, `protobuf` (google.protobuf) |
| Dépendance build | `pyinstaller` |
| OS cible | Windows uniquement |
| Protocole | USB série (COM ports), Meshtastic Python API |
| Format de sauvegarde | JSON renommé `.NBFM` |

**Installation des dépendances** :
```bash
pip install meshtastic pyserial protobuf
```

---

## 3. Structure du projet

```
Nodes-Backup-Fleet-Manager/
├── NBFM_V1.96.py             ← script principal actif (~4320 lignes) — voir « Script actif » pour le nom exact
├── Backup/                   ← versions horodatées précédentes (protocole de versioning)
├── RELEASE_NOTES.md          ← notes de version GitHub (bilingue)
├── README.md                 ← documentation bilingue FR/EN
├── CONTRIBUTING.md           ← guide de contribution
├── LICENCE                   ← CC BY-NC-SA 4.0
├── Images/                   ← screenshots pour le README
└── ...
```

> ⚠ Le nom du script actif change à chaque session (versioning horodaté). Se fier à la section
> « Script actif » en tête de ce fichier, pas au nom figé ci-dessus.

**Fichiers générés à l'exécution** (jamais commités) :
- `NBFM_Config.json` — persistance langue (fr/en) **et** dernier dossier de travail (`work_dir`), dans le dossier du script/EXE. Le port COM n'y est PAS stocké.
- `NBFM_notes.json` — notes personnelles par fichier NBFM, dans le dossier de travail
- `*.NBFM` — fichiers de sauvegarde (JSON), dans le dossier de travail choisi par l'utilisateur
- `NBFM_report_YYYYMMDD_HHMMSS.html` — rapports HTML

**Convention de nommage des fichiers NBFM exportés** :
```
meshtastic_[ShortName]_[YYYYMMDD]_[HHMMSS].NBFM
# Exemple : meshtastic_JMC_5F7B_20260509_095500.NBFM
```
Le `short_name` contient les 4 derniers hex de l'adresse MAC du nœud (`JMC_5F7B`).

---

## 4. Workflow d'exécution

> Remplacer `NBFM_<actif>.py` par le nom du script actif (voir « Script actif »).

### Lancer depuis le source
```bash
python NBFM_<actif>.py
```

### Compiler en EXE (PyInstaller)
```bash
pip install pyinstaller
pyinstaller --onefile --windowed --name "NBFM" NBFM_<actif>.py
# EXE généré dans dist/NBFM.exe
```
Le code gère les deux modes via `get_app_dir()` : `sys.frozen` pour l'EXE, `__file__` pour le source.

### Convention de nommage des nouveaux scripts
**Toujours utiliser le format `NBFM_YYYYMMDD_HHMM.py`** (jamais de numéro de version type `V1_78`).

---

## 5. Format NBFM réel — observations sur fichier de référence

> Fichier de référence : `meshtastic_GB_A7F9_20260525_094518.nbfm` (T-Echo, EU868, MediumSlow, 869.5 MHz)

### Structure JSON constatée sur matériel réel

| Champ | Valeur observée / notes |
|---|---|
| `_app_version` | `"2.2"` — évolue avec les versions du script (actuellement `"2.6"`) |
| `_export_date` | ISO 8601 avec microsecondes |
| `_profile_date` | Peut coexister avec `_export_date` sur des fichiers issus d'anciennes versions — ne pas supposer que sa présence signifie profil flotte (vérifier `_profile_type`) |
| `my_info.nodedb_count` | Nombre de nœuds connus au moment de l'export |
| `my_info.pio_env` | Environnement PlatformIO du firmware (ex : `"t-echo-inkhud"`) |
| `local_config.lora.override_frequency` | Float en MHz (ex : `869.5`) — prioritaire sur `frequency` pour l'affichage |
| `local_config.lora.override_duty_cycle` | `true` sur T-Echo France (contournement limite légale 1% duty cycle EU868) |
| `local_config.power.adc_multiplier_override` | `2.0` sur T-Echo — c'est ce que l'option "clear ADC" supprime |
| `local_config.device.tzdef` | Fuseau horaire POSIX (`"CET-1CEST,M3.5.0/2:00:00,M10.5.0/3:00:00"` pour la France) |
| `local_config.security.admin_key` | Liste de 3 entrées (2 clés base64 + 1 chaîne vide `""`) — forme normale sur T-Echo |
| `channels` | Liste de 8 objets (index 0–7), toujours présents même si désactivés (role=0, PSK="") |
| `channels[n].settings.psk` | Hex string 64 chars = 32 bytes = AES-256 ; 32 chars = AES-128 ; `""` = canal désactivé |
| `known_nodes` | Dict keyed par `!hex_node_id` (ex : `"!1666a7f9"`) — **pas une liste** |
| `known_nodes[id].user.publicKey` | Base64 — correspond aux valeurs de `security.admin_key` pour les nœuds de confiance |

### Modules présents dans ce fichier

Depuis la session 30/05/2026, `import_full_config` itère sur **tous** les modules présents dans le JSON (itération dynamique). Les modules `traffic_management`, `audio`, `remote_hardware` sont désormais restaurés. `statusmessage` est présent dans le fichier mais non exposé par l'API `writeConfig` — il est tenté silencieusement et ignoré si introuvable (voir Bug D).

### Extension de fichier
Le fichier de référence utilise l'extension `.nbfm` (minuscules). Le code recherche `*.NBFM` (majuscules). Sous Windows le filesystem est insensible à la casse — ça fonctionne. Sur Linux ce serait cassé. À garder en tête si portabilité Linux envisagée.

---

## 6. Architecture du code


### Organisation actuelle — fichier unique

Le script actif (voir « Script actif » pour le nom exact) contient tout le code (~4320 lignes).
Sections principales dans l'ordre (⚠ numéros de ligne **approximatifs** — ils dérivent à chaque édition ;
**se fier aux noms de fonctions, pas aux numéros**) :

| Lignes | Contenu |
|---|---|
| ~37–109 | Utilitaires, `_ToolTip`, `check_dependencies`, `get_app_dir`, `list_serial_ports` |
| ~115–135 | Connexion : `connect_device` |
| ~142–447 | Export : `proto_to_dict`, clés de sécurité, `export_full_config` |
| ~454–490 | Profil flotte : `build_fleet_profile` |
| ~495–510 | **`_coerce_repeated_fields()`** — normalise les champs repeated pour ParseDict |
| ~512–740 | Import : `_apply_section_to_node`, `_apply_module_section`, `import_full_config` |
| ~750–775 | Validation : `validate_config_integrity` |
| ~782–880 | Mappings LoRa/modem : `LORA_REGIONS`, `MODEM_PRESETS`, helpers de conversion |
| ~883–985 | Lecture métadonnées : `read_file_meta` |
| persistance | `load_lang`/`save_lang`, `load_work_dir`/`save_work_dir`, `load_notes`/`save_notes` |
| ~1036–1610 | Chaînes UI : `UI_STRINGS` (FR + EN), `tr()` |
| ~1618–3185 | Classe `NBFMApp` (UI Tkinter complète) |
| ~3190–3200 | Point d'entrée : `main()` |

### Découpage en modules — approche recommandée si le code grossit

Il n'y a pas d'obligation de rester en fichier unique. Si une nouvelle fonctionnalité
ou un refactor le justifie, découper en modules est bienvenu. Découpe naturelle suggérée :

```
nbfm/
├── __main__.py          ← point d'entrée (check_dependencies + mainloop)
├── core/
│   ├── connect.py       ← connect_device, list_serial_ports
│   ├── export.py        ← export_full_config, proto_to_dict, clés sécurité
│   ├── import_.py       ← import_full_config, _apply_section_to_node
│   ├── fleet.py         ← build_fleet_profile
│   └── validate.py      ← validate_config_integrity
├── ui/
│   ├── app.py           ← classe NBFMApp
│   ├── tooltip.py       ← _ToolTip
│   └── edit_popup.py    ← éditeur de champs clés (edit_config_fields)
└── data/
    ├── strings.py       ← UI_STRINGS, tr()
    ├── mappings.py      ← LORA_REGIONS, MODEM_PRESETS, DEVICE_ROLES
    └── persistence.py   ← load_lang, save_lang, load_notes, save_notes, read_file_meta
```

Si tu crées un module ou un nouveau fichier Python, nomme-le `NBFM_YYYYMMDD_HHMM.py`
ou place-le dans la structure ci-dessus selon son rôle.

### Fonctions et méthodes importantes à connaître

**Connexion / export**
- `connect_device(port)` — tente chaque port, timeout 8 s, vérifie `isConnected`
- `proto_to_dict(obj)` — convertit protobuf → dict Python (5 stratégies de fallback)
- `export_full_config(iface)` — exporte : my_info, metadata, owner (5 méthodes de fallback), local_config, module_config, channels, known_nodes

**Clés de sécurité**
- `_get_security_section()` — export des clés en base64
- `_apply_security_to_node()` — restaure admin_key via `del[:] + append()` (méthode CLI officielle Meshtastic)

**Import**
- `_coerce_repeated_fields(section_data, proto_obj)` — convertit les scalaires en listes pour les champs `repeated` avant ParseDict (corrige `ignore_incoming: 0` → `[]`)
- `_apply_section_to_node()` — sauvegarde proto via CopyFrom, `Clear()` + `_coerce_repeated_fields()` + `ParseDict()` + `writeConfig` ; restauration en cas d'échec ParseDict
- `_apply_module_section()` — idem pour les modules
- `import_full_config(iface, config)` — owner, **toutes** les sections local_config (itération dynamique), **tous** les modules (itération dynamique), canaux
- **Ordre des canaux (MODIFIÉ 1519)** : **primaire (role=1) EN PREMIER** (index 0), puis secondaires (role=2) par index, puis désactivés (role=0) — tri `_channel_role_to_int(role) != 1, index`. Aligné sur `setURL` du CLI Meshtastic. `writeChannel()` ne provoque PAS de reboot (confirmé source node.py) ; l'ancien ordre « primaire en dernier » (anti-reboot) n'était plus nécessaire et fiabilise mal les secondaires sur firmware récent. Suivi d'une **vérification post-commit + relance** (voir Travaux récents 1519).

**Profil flotte**
- `build_fleet_profile(config)` — supprime : my_info, metadata, owner, known_nodes, public_key, private_key, wifi_ssid, wifi_psk, compteurs version ; **conserve** admin_key

**UI**
- `_apply_lang()` — mise à jour inline de tous les textes sans reconstruire l'UI
- `refresh_files()` — recharge la liste, groupe par MAC (4 derniers hex du short_name), tri par mtime
- `export_config()` — export single node (thread daemon, `root.after()` pour les callbacks UI)
- `edit_config_fields()` — popup éditeur en **2 onglets** (`ttk.Notebook`) : **Principal** (owner, LoRa, rôle, ADC multiplier via `ADC_DEFAULTS`, nettoyage) et **Canaux** (les 8 canaux : case Activé→rôle, nom, PSK, précision GPS via `POSITION_PRECISION`, générateur de clés). Le save reconstruit les 8 canaux avec le rôle correct (nommer + activer suffit à activer un canal).
- `export_report()` — génère un rapport HTML et l'ouvre dans le navigateur

---

## 7. Conventions de code

### Nommage des fichiers créés
- **Scripts Python** : `NBFM_YYYYMMDD_HHMM.py` — jamais de numéro de version `V1_XX`
- **Fichiers NBFM** : `meshtastic_[ShortName]_[YYYYMMDD]_[HHMMSS].NBFM`
- **Profil flotte** : `profil_flotte_[YYYYMMDD]_[HHMMSS].NBFM` en FR, `fleet_profile_[YYYYMMDD]_[HHMMSS].NBFM` en EN (clé `fleet_filename`). Ce n'est qu'une **suggestion** dans la boîte d'enregistrement : les deux conventions sont équivalentes et l'utilisateur peut renommer librement. Ne JAMAIS déduire le type d'un fichier de son nom — utiliser `_profile_type == "fleet"`.
- **Rapport HTML** : `NBFM_report_[YYYYMMDD_HHMMSS].html`

### Règle absolue — zéro régression
Avant toute modification d'une fonction existante :
1. Comprendre précisément ce qu'elle fait aujourd'hui
2. Identifier les cas limites et les fallbacks en place
3. S'assurer que le comportement résultant est identique ou strictement meilleur
4. Ajouter le nouveau comportement *en complément*, pas en remplacement, si le moindre doute subsiste
5. Ne jamais supprimer un fallback existant, même s'il semble inutile

### Gestion des erreurs
- Broad `try/except Exception` partout — priorité à la robustesse sur la précision
- Toujours un fallback (ex : `writeConfig` seul si `ParseDict` échoue)
- Messages d'erreur affichés via `messagebox.showerror` ou `set_status()`
- Jamais de crash silencieux : tout est loggué dans `log[]` ou dans `set_status()`

### Threading
- Les opérations bloquantes (connexion série, export, import) tournent dans des **threads daemon**
- Les mises à jour UI depuis un thread se font **exclusivement** via `root.after(0, callback)`
- Ne jamais appeler `set_status()` ou tout widget tkinter directement depuis un thread

### Protobuf
- `Clear()` avant `ParseDict()` — garantit l'écrasement total, pas de merge partiel
- **Sauvegarder l'état proto avant `Clear()`** via `CopyFrom` — si ParseDict échoue, restaurer avant le fallback `writeConfig` (sinon l'appareil reçoit un proto à zéros)
- **`_coerce_repeated_fields()` obligatoire avant ParseDict** — les champs `repeated` peuvent être stockés en scalaire (`0`) dans d'anciens fichiers NBFM au lieu de liste (`[]`)
- `admin_key` : `del sec.admin_key[:] + append()` — méthode CLI officielle (pas ParseDict)
- `ignore_unknown_fields=True` dans ParseDict — compatibilité firmware
- PSK stockée en **hex** dans le JSON, convertie en **base64** dans l'UI et en **bytes** pour le protobuf

### Langue et UI
- `UI_STRINGS` est la **seule source de vérité** pour les textes UI — ne jamais hardcoder du texte en dehors
- `tr(key, **kwargs)` recharge la langue à chaque appel — pas de variable globale de langue
- Ajout d'une clé UI = l'ajouter dans **les deux** dictionnaires `"fr"` et `"en"`
- `_apply_lang()` met à jour **tous** les widgets sans détruire/recréer l'UI (sauf l'onglet Aide)
- **Le cœur aussi est bilingue** : `tr()` est utilisable hors de `NBFMApp` (pas de dépendance tkinter, il
  relit `NBFM_Config.json`). Tout message de log renvoyé par `_apply_section_to_node`,
  `_apply_module_section`, `_apply_security_to_node` ou `import_full_config` finit dans un popup :
  il doit passer par `tr()`, jamais par une f-string en dur.
- **Depuis un thread**, ne jamais lire `self.lang_var` directement : mettre `UI_STRINGS[self.lang_var.get()]`
  **à l'intérieur** du `lambda` passé à `root.after()` (voir `_do_import`).
- Dans une f-string `f"""…"""` (bloc HTML du rapport), utiliser des guillemets **simples** dans les
  accolades : `{T['report_col_type']}`. Les guillemets doubles imbriqués n'existent qu'à partir de Python 3.12.
- Ce qui NE doit PAS être traduit : le contenu écrit dans les fichiers `.NBFM` (`_profile_note`), les
  sentinelles internes (`tag` = `🚀FLOTTE`/`💾BACKUP`), les identifiants de colonnes Treeview, les noms
  d'enums Meshtastic (`DISABLED`, `EU_868`, `CLIENT_MUTE`) et les noms de fichiers suggérés.

### Pas de tests automatisés
- Aucun framework de test (pytest, unittest)
- Validation manuelle sur matériel réel (T-Echo, ESP32 V3)

### Vérification Python (Ruff) — obligatoire en fin de série
- Quand tu as terminé **TOUTES** les modifications demandées sur un ou plusieurs fichiers Python,
  lance **une seule fois** :
  ```bash
  python -m ruff check <fichier> --select E9,F63,F7,F82
  ```
  (fichier = le script actif, p. ex. `NBFM_V1.96.py` — voir « Script actif »)
- Si Ruff signale des erreurs, corrige-les, puis relance la vérification jusqu'à obtenir
  `All checks passed!`.
- **Ne lance pas Ruff après chaque petite modification**, uniquement à la fin de la série.
- **Ne modifie pas d'autres parties du code que celles demandées** — une remarque Ruff sur du code
  hors périmètre se signale à l'utilisateur, elle ne se corrige pas d'office.
- Ces règles ne couvrent que les erreurs bloquantes : `E9` (syntaxe / indentation), `F63` (comparaisons
  invalides), `F7` (`break`/`return` hors contexte…), `F82` (nom indéfini). Elles ne remplacent ni
  `py_compile` ni les smoke-tests headless — elles s'y ajoutent.
- ⚠ **Un `ruff check` SANS `--select` renvoie ~151 remarques `BLE001` / `S110`** : c'est la convention
  assumée du §7 « Gestion des erreurs » (broad `try/except Exception` partout, fallbacks jamais supprimés).
  **Ne pas les corriger** — ce serait une régression au regard de la règle n°1. Les `# noqa` déjà en place
  (`DTZ005` sur `_export_date`/`_profile_date`, `F401` sur l'import-sonde `ParseDict`) sont également
  volontaires et commentés : ne pas les retirer.

---

## 8. Contraintes spécifiques

| Contrainte | Détail |
|---|---|
| **Zéro régression** | Toute modification du code doit préserver le comportement existant. Avant de changer une fonction : identifier ce qu'elle fait, s'assurer que le résultat est identique ou strictement meilleur. En cas de doute, ajouter le nouveau comportement *à côté* de l'ancien, pas à la place. Les fallbacks existants ne doivent jamais être supprimés. |
| Client web officiel | Le paramétrage source se fait sur https://client.meshtastic.org — NBFM est un outil de backup/déploiement, pas un configurateur complet |
| Exhaustivité de l'import | Tous les modules et sections présents dans le JSON doivent être restaurés, y compris les inconnus. Ne jamais ignorer silencieusement un champ. |
| Windows uniquement | `os.startfile()`, COM ports style Windows |
| COM1 exclu | Port système Windows (BIOS/souris), jamais un appareil USB Meshtastic |
| Redémarrage obligatoire | Après toute restauration, l'appareil doit être redémarré manuellement |
| Ordre canaux (MODIFIÉ 1519) | **Primaire (role=1) EN PREMIER**, puis secondaires (role=2) par index, puis désactivés. Aligné sur `setURL`. Puis vérification post-commit (relecture `requestChannels`) + relance directe des canaux non appliqués. `writeChannel()` ne provoque PAS de reboot (source `node.py`). |
| private_key non restaurée dans les profils flotte | Chaque nœud garde ses propres clés cryptographiques |
| Frozen/non-frozen | `get_app_dir()` doit être utilisé pour tout chemin relatif à l'app |
| Dossier de travail variable | L'utilisateur peut choisir n'importe quel dossier ; `work_dir` est une variable |
| Connexion série fragile | Timeout 8 s, fallback sur tous les ports si non spécifié, `iface.close()` dans `finally` |

---

## 9. Notes API Meshtastic Python — faits confirmés

> Issus de l'analyse du source `node.py` (github.com/meshtastic/python) lors de la session 30/05/2026.

| Point | Détail |
|---|---|
| `writeConfig(name)` — sections locales valides | `device`, `position`, `power`, `network`, `display`, `lora`, `bluetooth`, `security` |
| `writeConfig(name)` — modules valides | `mqtt`, `serial`, `external_notification`, `store_forward`, `range_test`, `telemetry`, `canned_message`, `audio`, `remote_hardware`, `neighbor_info`, `detection_sensor`, `ambient_lighting`, `paxcounter`, `traffic_management` |
| `writeChannel(index)` | N'entraîne **pas** de reboot dans les firmwares actuels. `p.set_channel.CopyFrom(self.channels[index])` → écrit l'objet à cet index. |
| `setURL(url)` (méthode canonique de restauration des canaux) | Le CLI officiel (`--configure`) restaure TOUS les canaux via `setURL` : boucle `i=0..n` → **primaire en premier (i=0)**, secondaires ensuite, écrit **uniquement les canaux actifs** (pas les désactivés), puis `set_config.lora` À LA FIN. NBFM s'aligne sur cet ordre (1519) mais écrit aussi les désactivés pour purger l'ancien état. |
| `requestChannels()` | **Asynchrone** : met `channels=None` puis repeuple via le thread lecteur (handler `onResponseRequestChannel`). Ne bloque pas seul — utiliser `waitForConfig("channels")` (timeout **300 s** — trop long) ou un poll borné maison (NBFM : 8 s). |
| `isConnected` | **`threading.Event`**, pas un bool. Tester `iface.isConnected.wait(timeout)` / `.is_set()`, jamais `not iface.isConnected` (toujours faux). |
| `setOwner(long_name, short_name)` | Tronque `short_name` à **4 caractères** automatiquement (message d'avertissement dans le terminal — comportement normal, pas un bug NBFM) |
| `beginSettingsTransaction()` / `commitSettingsTransaction()` | Existent dans l'API. Utiles pour les nœuds distants (mesh). Non utilisés actuellement pour les connexions USB directes. |
| `statusmessage` | Présent dans les fichiers NBFM du T-Echo mais **absent** de la liste writeConfig officielle. Non accessible via `moduleConfig.statusmessage`. Peut-être un module interne non exposé par l'API Python. |

---

## 10. Roadmap / TODO

> Tous les bugs (A, B, C, E–M) + la barre de progression + la validation PSK sont **faits** — voir
> « Travaux récents » en tête. Ci-dessous, uniquement les pistes ouvertes (aucune urgente). La liste
> priorisée pour rappel utilisateur est dans « ★ Pistes d'évolution proposées » en tête de fichier.

### Priorité moyenne
- [ ] **Rapport CSV** : format CSV en plus du HTML (plus facile à filtrer dans Excel)
- [ ] **Support YAML natif** : le glob inclut `*.yaml/*.yml` mais `import_full_config` ne gère que JSON
- [ ] **Firmware dans la bulle de survol** : afficher la version firmware (`metadata`) — pas de colonne supplémentaire
- [ ] **Documenter la commande PyInstaller** dans le README (commande exacte + `--icon`, `--windowed`)

### Priorité basse
- [ ] **Icône application** pour l'EXE PyInstaller
- [ ] **Tests basiques** : au moins un test sur `build_fleet_profile()` et `validate_config_integrity()`
- [ ] **Auto-détection port** plus fine : filtrer par VID/PID Silicon Labs (CP210x) ou CH340
- [ ] **Mode ligne de commande** : export/import sans GUI (pour automatisation)
- [ ] **Refactoring modulaire** : découper en packages `core/`, `ui/`, `data/` si le code dépasse ~4000 lignes
- [ ] **Bug D — `statusmessage`** : vérifier si ce module est accessible autrement dans l'API Meshtastic Python
