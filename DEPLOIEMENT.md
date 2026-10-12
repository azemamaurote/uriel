# Déploiement et exploitation — Uriel

[← Accueil](README.md) · [Documentation](DOCUMENTATION.md) · [Versions](VERSIONS.md)

**État observé : 12 octobre 2026.** Uriel est hébergé sur Oracle Cloud (Ubuntu) et exécuté par le service `uriel` de `systemd`. Les commandes ci-dessous s'exécutent sur une machine autorisée à administrer cette instance.

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

1. Préparer les nouveaux `main.py` **et** `database/database.py` ensemble ; tester leur compatibilité en local.
2. Faire une sauvegarde des fichiers de production et de la **base SQLite** avant tout remplacement. Ne jamais remplacer la base de production par une base locale.
3. Transférer les deux fichiers Python avec un accès SSH/SCP autorisé. Ne pas envoyer `.env`, la base SQLite, les clés SSH ou `.venv` dans le dépôt GitHub.
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
