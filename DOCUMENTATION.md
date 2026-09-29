# 📘 Documentation complète — Uriel

[← Accueil](README.md) · [Historique des versions](VERSIONS.md) · [Confidentialité](PRIVACY.md) · [Conditions](TERMS.md)

**Version actuelle documentée : 1.0.3**

## 1. Présentation

Uriel est un bot Discord développé en Python pour la compagnie libre FFXIV **Anges – Apocalypse**. Il complète Raid-Helper en automatisant le suivi des sorties, les inscriptions, les rappels, certains changements de statut et le nettoyage des messages liés aux événements.

| Élément | Choix actuel |
|---|---|
| Interface | Discord |
| Source des événements | Raid-Helper |
| Langage | Python |
| Persistance | SQLite |
| Hébergement | Oracle Cloud, Ubuntu |
| Exécution | service `systemd` |
| Surveillance | temps réel + contrôle de sécurité périodique |

## 2. Objectifs

- Découvrir automatiquement les événements Raid-Helper des salons surveillés.
- Suivre les inscriptions et changements de statut.
- Envoyer uniquement les notifications utiles à l'organisation.
- Fournir un fichier calendrier `.ics` lors des inscriptions concernées.
- Envoyer les rappels avant une sortie.
- Signaler les désistements tardifs utiles sans multiplier les messages.
- Réagir aux modifications et suppressions d'événements.
- Fonctionner en permanence sans dépendre du PC personnel de l'administrateur.

## 3. Architecture

```text
Uriel/
├── main.py
├── services/
│   ├── raid_helper.py
│   └── calendar_ics.py
├── database/
│   ├── database.py
│   └── uriel.db
├── .env
└── .gitignore
```

`main.py` orchestre Discord, les événements, les rappels et les règles métier. `raid_helper.py` gère les appels à Raid-Helper. `calendar_ics.py` génère les calendriers. `database.py` gère la persistance SQLite. Les secrets restent dans `.env` et ne doivent pas être publiés.

## 4. Fonctionnement général

1. Uriel découvre un événement Raid-Helper dans un salon surveillé.
2. Il récupère l'état de l'événement et de ses inscriptions.
3. Il compare cet état à celui déjà connu.
4. Il applique les règles correspondant aux changements détectés.
5. Il envoie, modifie ou supprime les messages nécessaires.
6. Il conserve l'état utile pour les prochains contrôles.

La surveillance Discord en temps réel est complétée par un **contrôle de sécurité toutes les cinq minutes**.

## 5. Règles d'inscription actuelles

| Transition | Comportement Uriel |
|---|---|
| Nouvelle inscription Tank / DPS / Heal / Bench | MP de confirmation + fichier `.ics` |
| Nouvelle inscription Tentative | MP spécifique d'inscription provisoire |
| Nouvelle inscription Absence | Silencieux |
| Tank / DPS / Heal → Absence | Désistement ; MP + message public si la sortie est dans la fenêtre H-24 |
| Bench → Absence | Pas de message de place de combat libérée |
| Tank / DPS / Heal → Tentative | Silencieux |
| Bench → Tentative | Silencieux |
| Tentative → Absence | Silencieux |
| Absence → Tentative | MP spécifique d'inscription provisoire |
| Absence / Tentative → Tank / DPS / Heal | Inscription ou réinscription ; mise à jour d'un éventuel message public existant |
| Changement de job uniquement | Silencieux |

**Principe :** `Tentative` n'est pas `Absence`. Tentative représente une participation possible mais non garantie ; Absence signifie que le membre ne participera pas.

## 6. Désistements

Lorsqu'un participant confirmé passe en Absence pendant les 24 dernières heures avant la sortie, Uriel peut envoyer un MP de confirmation et publier un message dans le salon. Le message public ne ping pas le membre. Le dernier rôle de combat est mémorisé afin d'indiquer correctement la place libérée : Tank, DPS ou Heal.

Un même utilisateur ne doit avoir qu'**un seul message public de suivi par événement**. En cas de réinscription, ce message est modifié plutôt qu'un nouveau message créé.

## 7. Rappels

- **H-24 :** rappel général.
- **H-30 :** rappel de départ.
- Sont concernés : **Tank, DPS, Heal, Bench et Tentative**.
- **Absence** est exclu.

Les textes sont volontairement statiques afin d'éviter que les timestamps relatifs Discord se transforment après l'heure de départ.

```text
🔔 Rappel — ZORAAL JA
La sortie commence dans 24 heures !

⏰ ZORAAL JA commence dans 30 minutes !
```

## 8. Modification d'une sortie

Lorsque la date ou l'heure d'un événement change, Uriel détecte la modification et informe les participants concernés. Les changements de job d'un participant restent silencieux.

## 9. Suppression et nettoyage

Uriel gère la suppression directe d'un événement sur Discord et les suppressions détectées via Raid-Helper. Le contrôle périodique permet notamment de retrouver un événement connu qui n'est plus découvert normalement ; une réponse API `404` peut alors déclencher le nettoyage.

| Situation | Nettoyage | MP d'annulation |
|---|---|---|
| Événement futur supprimé | Oui | Oui, pour les participants concernés |
| Événement passé supprimé | Oui | Non |

Les messages Uriel associés à l'événement et les données SQLite correspondantes sont supprimés.

## 10. Calendriers `.ics`

Uriel génère un fichier standard `.ics` lors des inscriptions concernées. Le bot n'accède pas au calendrier personnel du membre : celui-ci choisit lui-même d'importer ou non le fichier dans son application.

## 11. Persistance SQLite

La base conserve l'état nécessaire au fonctionnement entre les redémarrages : événements, inscriptions, rappels et références de messages. Une table dédiée référence notamment les messages créés pour un événement afin de permettre leur nettoyage.

```sql
CREATE TABLE IF NOT EXISTS uriel_event_messages (
    message_id TEXT PRIMARY KEY,
    event_id TEXT NOT NULL,
    message_type TEXT NOT NULL
)
```

La base de production est un état vivant : **elle ne doit pas être remplacée par une base locale lors d'un déploiement**.

## 12. Démarrage et synchronisation

Au démarrage, Uriel reconstruit son suivi à partir des événements existants. Cette synchronisation doit rester silencieuse pour les inscriptions déjà présentes afin d'éviter des MP ou rappels artificiels. Le contrôle de sécurité périodique attend son intervalle avant son premier passage afin d'éviter un scan redondant immédiatement après le démarrage.

## 13. Robustesse Raid-Helper

Les appels API utilisent un délai d'attente et plusieurs tentatives contrôlées pour les erreurs réseau ou serveur. Les erreurs `404` sont traitées spécifiquement pour les événements connus. Les limitations temporaires de l'API sont également prises en compte.

## 14. Infrastructure

### Développement local

- Windows
- Python 3.14.x
- environnement virtuel `.venv`
- principales dépendances : `discord.py`, `python-dotenv`, `aiohttp`

### Production

- Oracle Cloud Always Free
- Ubuntu 24.04 LTS
- VM ARM A1 Flex, 1 OCPU / 2 Go de RAM
- projet dans `/home/ubuntu/uriel`
- exécution via `systemd`

Commandes courantes :

```bash
cd /home/ubuntu/uriel
./.venv/bin/python -m py_compile main.py
sudo systemctl restart uriel
sudo systemctl status uriel
sudo journalctl -u uriel -f
```

## 15. Points de vigilance

- Uriel dépend de Discord et de Raid-Helper.
- Les secrets et tokens ne doivent jamais être publiés.
- La base SQLite de production doit être conservée lors des mises à jour.
- Des messages orphelins issus d'anciens bugs peuvent ne pas être récupérés si leur événement n'est plus connu du suivi actuel.
- Le texte H-24 est volontairement statique ; un éventuel rattrapage tardif conservera donc ce libellé.

## 16. Mise à jour du projet

À chaque évolution : tester localement, valider les scénarios concernés, déployer uniquement les fichiers nécessaires, vérifier la compilation et le service, contrôler les logs, puis mettre à jour cette documentation et l'[historique des versions](VERSIONS.md).

---

[← Retour à l'accueil](README.md)
