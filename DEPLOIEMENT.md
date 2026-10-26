# Déploiement et exploitation — Uriel

[← Accueil](README.md) · [Documentation](DOCUMENTATION.md) · [Versions](VERSIONS.md)

**État observé : 24 octobre 2026.** Uriel est hébergé sur Oracle Cloud (Ubuntu) et exécuté par le service `uriel` de `systemd`. Les commandes ci-dessous s'exécutent sur une machine autorisée à administrer cette instance.

## Configuration active

| Élément | Valeur |
|---|---|
| Salons surveillés | HL `1522187478340079646` ; Général `1514280298932731954` |
| Rôle Endormi | `1559153132884459520` |
| Salon de développement | `1542885975070351450` (non surveillé en production) |
| Cycle de surveillance | 1 minute |
| Nettoyage automatique | 24 heures après l'heure de début Raid-Helper |
| Dossier serveur | `/home/ubuntu/uriel` |
| Service | `uriel.service` |

## Prérequis

- Python et dépendances installés dans `.venv` (`discord.py`, `python-dotenv`, `aiohttp` et dépendances du projet).
- Fichier `.env` contenant le token Discord, **jamais publié sur GitHub**.
- Permissions Discord adaptées à la lecture, à la création/suppression de fils et à la suppression des messages nécessaires au nettoyage.
- Module `database/database.py` **compatible avec** `main.py` ; la version H+24 exige notamment `get_expired_events` et les fonctions de persistance associées.

## Procédure de mise à jour sûre

1. Préparer les fichiers réellement modifiés et tester leur compatibilité en local. **Le 24 octobre, `main.py` et `database/database.py` ont été déployés ensemble.** Pour les prochaines mises à jour, ne remplacer que les fichiers réellement modifiés et compatibles. Pour une évolution de schéma ou d’API de base, déployer les deux fichiers compatibles ensemble.
2. Faire une sauvegarde des fichiers de production et de la **base SQLite** avant tout remplacement. Ne jamais remplacer la base de production par une base locale.
3. Transférer le ou les fichiers Python modifiés avec un accès SSH/SCP autorisé. Ne pas envoyer `.env`, la base SQLite, les clés SSH ou `.venv` dans le dépôt GitHub.
4. Sur le serveur, vérifier la syntaxe et les imports :

```bash
cd /home/ubuntu/uriel
./.venv/bin/python -m py_compile main.py database/database.py
./.venv/bin/python -c "from database.database import get_expired_events; print('Import OK')"
```

5. Après validation, redémarrer **l'instance systemd existante**, sans démarrer un second bot local :

```bash
sudo systemctl restart uriel
sudo systemctl status uriel --no-pager
sudo journalctl -u uriel -n 50 --no-pager
```

6. Vérifier dans les logs que le bot s'est connecté et que le service ne redémarre pas en boucle. Tester une création de fil sur un événement réel, puis surveiller les rappels et le nettoyage H+24.

## Incident rencontré le 12 octobre 2026

Après le déploiement de `main.py`, le service échouait avec :

```text
ImportError: cannot import name 'get_expired_events' from 'database.database'
```

**Cause :** le module `database/database.py` de production était plus ancien que `main.py`. Le transfert du module local à jour et le redémarrage ont rétabli le fonctionnement. Les logs ont confirmé la connexion d'Uriel à Discord et la création d'un fil sur un raid en cours.

## Diagnostic

```bash
sudo systemctl status uriel --no-pager
sudo journalctl -u uriel -n 100 --no-pager
```

En cas de redémarrages répétés, arrêter temporairement le service avec `sudo systemctl stop uriel`, diagnostiquer les erreurs, restaurer les sauvegardes si nécessaire, puis le relancer. Quitter une session SSH **n'arrête pas** le service `systemd`.

## Sécurité du dépôt

Ne jamais versionner les secrets, clés SSH, fichiers `.env`, bases SQLite de production, caches Python ou environnements virtuels. Vérifier `.gitignore` avant `git add`.

## Déploiement du 18 octobre 2026

- Mise à jour de `main.py` pour centraliser les notifications publiques dans les fils Raid-Helper. Aucun changement de `database/database.py`.
- Sauvegarde de l’ancien `main.py`, arrêt du service, transfert du nouveau fichier, contrôle de syntaxe, redémarrage du service.
- Journaux du 18 octobre à 09:50 (heure affichée par le serveur) : connexion `Uriel#4477`, détection des événements, surveillance Raid-Helper temps réel activée, **2 salons de production** et contrôle de sécurité **chaque minute**.
- Aucun message d’erreur dans l’extrait de journaux communiqué après redémarrage. Vérifier le comportement des notifications sur un prochain événement réel.

Pour restaurer la sauvegarde locale du serveur, si elle a été réalisée sous `main.py.backup` :

```bash
cd /home/ubuntu/uriel
sudo systemctl stop uriel
cp main.py.backup main.py
sudo systemctl start uriel
sudo systemctl status uriel --no-pager
```

## Déploiement et migration du 24 octobre 2026

Cette opération a été réalisée sur le serveur Oracle avec sauvegarde préalable de la base SQLite et des deux fichiers Python. Le script de migration est un **outil de maintenance ponctuel**, distinct de l'exécution normale du bot.

### Sauvegardes

Sauvegarde SQLite réalisée avec l'API de sauvegarde de SQLite :

```text
/home/ubuntu/uriel/database/uriel_backup_20261024_192130.db
```

Les anciens fichiers de production ont également été sauvegardés sous `main.py.backup_20261024` et `database/database.py.backup_20261024` avant leur remplacement.

### Migration des messages existants

Depuis `/home/ubuntu/uriel`, avec le script `uriel_migration_messages.py` et la configuration de production :

```bash
.venv/bin/python uriel_migration_messages.py preview
sudo systemctl stop uriel
sudo systemctl is-active uriel
.venv/bin/python uriel_migration_messages.py apply
```

**Important :** la sauvegarde SQLite doit précéder `apply`. Vérifier les références proposées par `preview`, puis effectuer `apply` uniquement avec le service arrêté pour éviter des écritures concurrentes. Ne pas réexécuter aveuglément une migration déjà appliquée ; contrôler au préalable les références et le résultat. Conserver une copie de la base avant toute nouvelle opération.

Résultat observé le 24 octobre :

```text
Mode : preview | Références à examiner : 1
Bilan : {'already': 0, 'moved': 0, 'preview': 1, 'skipped': 0, 'errors': 0}
Mode : apply | Références à examiner : 1
Bilan : {'already': 0, 'moved': 1, 'preview': 0, 'skipped': 0, 'errors': 0}
```

Le message concernait **Canalisations silidiennes extrême — Boss 3 : Zeless Gah**. Sa présence dans le fil de l'événement a été confirmée sur Discord après migration.

### Mise à jour du code et redémarrage

Après transfert des versions compatibles de `main.py` et `database/database.py` :

```bash
cd /home/ubuntu/uriel
.venv/bin/python -m py_compile main.py database/database.py
grep -n -A 5 'MONITORED_CHANNEL_IDS =' main.py
sudo systemctl start uriel
sudo systemctl is-active uriel
sudo journalctl -u uriel -n 40 --no-pager
```

Le contrôle a confirmé `MONITORED_CHANNEL_IDS = {HIGH_LEVEL_CHANNEL_ID, GENERAL_CHANNEL_ID}`. Le service a renvoyé `active` ; les journaux ont confirmé `Uriel est prêt`, la détection de cinq événements, la surveillance de deux salons et le contrôle de sécurité chaque minute. Aucune erreur n'était visible dans les lignes de démarrage consultées.

### Validation restante

- **Validé en développement :** accueil/épinglage du fil, H-24, H-30, changements d'horaire, absences et réinscriptions dans les fils.
- **Validé en production :** migration d'un message historique, présence dans le fil, démarrage et surveillance.
- **À observer :** une nouvelle notification automatique publiée dans le bon fil en production, sans message parasite dans le salon parent.

### Sécurité GitHub

Ce dépôt documentaire ne doit contenir ni `database/uriel.db`, ni sauvegardes SQLite, ni `.env`, ni clé SSH, ni token Discord. Le script de migration ne doit être publié que s'il a été relu et débarrassé de toute donnée sensible ; il **n'est pas inclus dans cette archive documentaire**.
