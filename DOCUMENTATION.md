# 📘 Documentation complète — Uriel

[← Accueil](README.md) · [Historique des versions](VERSIONS.md) · [Confidentialité](PRIVACY.md) · [Conditions](TERMS.md)

**Version actuelle documentée : 2.0.0**

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
- Demander aux inscriptions Provisoire de régulariser leur situation à H-3.
- Fournir au Leader et aux Co-leaders un récapitulatif opérationnel à H-2.
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

La surveillance Discord en temps réel est complétée par un **contrôle de sécurité toutes les cinq minutes** sur les derniers messages des salons surveillés.

## 5. Statuts Raid-Helper pris en charge

| Valeur Raid-Helper | Affichage Uriel | Signification |
|---|---|---|
| `Tank` | Tank | Participant confirmé |
| `Dps` | DPS | Participant confirmé |
| `Healer` | Heal | Participant confirmé |
| `Allrounder` | Tout | Participant confirmé sans rôle de combat imposé |
| `Bench` | Banc | Remplaçant disponible |
| `Late` | Tard | Participant annoncé en retard |
| `Tentative` | Provisoire | Participation possible mais non confirmée |
| `Absence` | Absence | Ne participe pas |

**Principe :** Provisoire n'est pas Absence. Uriel ne transforme pas automatiquement une inscription Provisoire en Absence.

## 6. Règles d'inscription actuelles

| Transition | Comportement Uriel |
|---|---|
| Nouvelle inscription Tank / DPS / Heal / Tout / Banc / Tard | MP de confirmation + fichier `.ics` |
| Nouvelle inscription Provisoire | MP spécifique d'inscription provisoire + fichier `.ics` |
| Nouvelle inscription Absence | Silencieux |
| Tank / DPS / Heal / Tout → Absence | Désistement ; MP + message public si la sortie est dans la fenêtre H-24 |
| Banc / Tard → Absence | Silencieux ; pas de place de combat libérée |
| Tank / DPS / Heal / Tout → Provisoire | Silencieux |
| Banc / Tard → Provisoire | Silencieux |
| Provisoire → Absence | Silencieux |
| Absence → Provisoire | MP spécifique d'inscription provisoire |
| Absence / Provisoire → Tank / DPS / Heal / Tout | Inscription ou réinscription ; mise à jour d'un éventuel message public existant |
| Changement de job uniquement | Silencieux |

## 7. Désistements

Lorsqu'un participant confirmé passe en Absence pendant les 24 dernières heures avant la sortie, Uriel peut envoyer un MP de confirmation et publier un message dans le salon. Le message public ne ping pas le membre. Le dernier rôle de combat est mémorisé afin d'indiquer correctement la place libérée : Tank, DPS, Heal ou Tout.

Un même utilisateur ne doit avoir qu'**un seul message public de suivi par événement**. En cas de réinscription, ce message est modifié plutôt qu'un nouveau message créé.

## 8. Rappels

L'ordre fonctionnel des rappels est le suivant :

- **H-24 :** rappel général dans le salon de la sortie.
- **H-3 :** MP à chaque membre encore en Provisoire.
- **H-2 :** MP récapitulatif au Leader et aux Co-leaders.
- **H-30 :** rappel de départ dans le salon de la sortie.

Les rappels publics H-24 et H-30 concernent **Tank, DPS, Heal, Tout, Banc, Tard et Provisoire**. **Absence** est exclu.

### H-3 — Provisoire

À H-3, chaque membre encore en Provisoire reçoit un MP lui demandant de mettre à jour son inscription dans l'heure. Uriel rappelle les choix possibles et précise que le statut Tard doit être accompagné d'une heure d'arrivée communiquée au Leader ou à un Co-leader. Le retard est limité à 30 minutes maximum ; au-delà, le membre doit passer en Absence.

Le Leader et les Co-leaders sont récupérés directement depuis les métadonnées Raid-Helper. L'envoi est mémorisé avec `tentative_reminder_sent`.

### H-2 — Leader et Co-leaders

À H-2, Uriel récupère le Leader et les Co-leaders depuis Raid-Helper et leur envoie individuellement le même récapitulatif. Les identifiants sont dédupliqués.

Le récapitulatif contient :

- les inscrits confirmés avec leur **nom et leur job** ;
- les membres en **Banc** ;
- les membres **En retard** ;
- les membres **Toujours en Provisoire**.

Les catégories vides sont masquées. Les Absences ne sont pas affichées. L'envoi est mémorisé avec `leader_summary_sent`.

## 9. Modification d'une sortie

Lorsque la date ou l'heure d'un événement change, Uriel détecte la modification et informe les participants concernés. Les indicateurs de rappel de l'événement sont réinitialisés afin que les rappels correspondant au nouvel horaire puissent être réévalués. Les changements de job d'un participant restent silencieux.

## 10. Suppression et nettoyage

Uriel gère la suppression directe d'un événement sur Discord et les suppressions détectées via Raid-Helper. Le contrôle périodique permet notamment de retrouver un événement connu qui n'est plus découvert normalement ; une réponse API `404` peut alors déclencher le nettoyage.

| Situation | Nettoyage | MP d'annulation |
|---|---|---|
| Événement futur supprimé | Oui | Oui, pour les participants concernés |
| Événement passé supprimé | Oui | Non |

Les messages Uriel associés à l'événement et les données SQLite correspondantes sont supprimés.

## 11. Calendriers `.ics`

Uriel génère un fichier standard `.ics` lors des inscriptions concernées. Le bot n'accède pas au calendrier personnel du membre : celui-ci choisit lui-même d'importer ou non le fichier dans son application.

Le MP contient le fichier `.ics`, puis Uriel ajoute un bouton **« 📅 Ajouter à mon calendrier »** pointant vers la pièce jointe Discord.

## 12. Persistance SQLite

La base conserve l'état nécessaire au fonctionnement entre les redémarrages : événements, inscriptions, rappels et références de messages. Une table dédiée référence notamment les messages créés pour un événement afin de permettre leur nettoyage.

```sql
CREATE TABLE IF NOT EXISTS uriel_event_messages (
    message_id TEXT PRIMARY KEY,
    event_id TEXT NOT NULL,
    message_type TEXT NOT NULL
)
```

Les indicateurs de rappel incluent notamment :

```text
general_reminder_sent
departure_reminder_sent
tentative_reminder_sent
leader_summary_sent
```

La base de production est un état vivant : **elle ne doit pas être remplacée par une base locale lors d'un déploiement**.

## 13. Démarrage et synchronisation

Au démarrage, Uriel reconstruit son suivi à partir des événements existants. Cette synchronisation reste silencieuse pour les inscriptions déjà présentes afin d'éviter des MP artificiels. Les rappels restent évalués à partir de l'état persistant de chaque événement.

## 14. Robustesse Raid-Helper

Les appels API utilisent un délai d'attente et plusieurs tentatives contrôlées pour les erreurs réseau ou serveur. Les erreurs `404` sont traitées spécifiquement pour les événements connus. Les limitations temporaires de l'API sont également prises en compte.

## 15. Infrastructure

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
./.venv/bin/python -m py_compile main.py database/database.py
sudo systemctl restart uriel
sudo systemctl status uriel
sudo journalctl -u uriel -f
```

## 16. Messages envoyés par Uriel

Cette section reproduit **les textes générés par la v2.0.0**. Les éléments entre `<...>` représentent uniquement les valeurs dynamiques insérées au moment de l'envoi.

### 16.1 Messages privés

#### Inscription confirmée

```text
✅ **Inscription confirmée — <nom de la sortie>**
Tu es bien inscrit en **<Tank / DPS / Heal / Tout / Banc / Tard>**.

📅 <date et heure Discord>

Tu peux ajouter cette sortie à ton calendrier avec le bouton ci-dessous.
```

Le MP contient également le fichier `.ics` et le bouton **« 📅 Ajouter à mon calendrier »**.

#### Inscription provisoire

```text
🟠 **Inscription provisoire — <nom de la sortie>**
Tu es maintenant inscrit en **Provisoire** pour cette sortie.

⚠️ Si tu sais finalement que tu ne pourras pas participer, pense impérativement à te remettre en **Absence** sur Raid-Helper afin que l'organisation de la sortie reste claire.

📅 <date et heure Discord>

Tu peux ajouter cette sortie à ton calendrier avec le bouton ci-dessous.
```

Le MP contient également le fichier `.ics` et le bouton **« 📅 Ajouter à mon calendrier »**.

#### Confirmation d'absence

```text
Ton passage en **Absent** pour **<nom de la sortie>** a bien été pris en compte.

📅 <date et heure Discord>
```

#### H-3 — membre toujours Provisoire

```text
⏳ **Inscription provisoire — <nom de la sortie>**

La sortie commence dans **3 heures** et tu es toujours inscrit en **Provisoire**.

Merci de mettre à jour ton inscription sur **Raid-Helper dans l'heure**.

⚔️ **Tu participes ?**
Passe-toi en **Tank, Heal, DPS ou Tout**.

🪑 **Tu souhaites rester disponible en remplacement ?**
Passe-toi en **Banc**.

🕒 **Tu seras en retard ?**
Passe-toi en **Tard** et préviens <@Leader> (Leader) ou <@Co-leader> (Co-leader) de ton heure d'arrivée.

Le retard est limité à **30 minutes maximum**. Au-delà, passe-toi en **Absence**.

❌ **Tu ne seras pas disponible ?**
Passe-toi en **Absence**.

⚠️ **Merci de ne pas rester en Provisoire une fois ta situation connue.**
```

S'il y a plusieurs Co-leaders, ils sont ajoutés au texte. Si Raid-Helper ne fournit exceptionnellement aucun responsable, Uriel écrit **« un responsable de la sortie »**.

#### H-2 — récapitulatif Leader / Co-leader

```text
📋 **Récap de la sortie — <nom de la sortie>**

La sortie commence dans **2 heures**.

⚔️ **Inscrits**
- <membre> — <job>

🪑 **Banc**
- <membre>

🕒 **En retard**
- <membre>

⚠️ **Toujours en Provisoire**
- <membre>
```

Chaque bloc vide est omis du message. Le MP est envoyé individuellement au Leader et à chaque Co-leader.

#### Annulation d'une sortie future

```text
❌ **Sortie annulée — <nom de la sortie>**
La sortie prévue le <date et heure Discord> a été annulée.
```

### 16.2 Messages dans les salons Discord

#### H-24 — rappel général

```text
🔔 **Rappel — <nom de la sortie>**
La sortie commence dans 24 heures !

<mentions des participants concernés>

⚠️ **Si vous ne pouvez finalement pas participer, merci de vous passer en `Absence` sur Raid-Helper afin de libérer votre place.**
```

#### H-30 — rappel de départ

```text
⏰ **<nom de la sortie> commence dans 30 minutes !**
<mentions des participants concernés>
```

#### Modification de l'horaire

```text
📅 **Modification — <nom de la sortie>**
L'horaire de la sortie a été modifié.

Ancien horaire : <ancienne date et heure Discord>
Nouvel horaire : <nouvelle date et heure Discord>

<mentions des participants concernés>
```

#### Désistement tardif — place libérée

```text
⚠️ **<membre>** vient de se désinscrire de **<nom de la sortie>**.
Une place de **<Tank / DPS / Heal / Tout>** vient de se libérer !
```

#### Réinscription — modification du message de désistement existant

```text
~~⚠️ **<membre>** s'était désinscrit de **<nom de la sortie>**.~~
~~Une place de **<Tank / DPS / Heal / Tout>** s'était libérée.~~

✅ **Mise à jour :** **<membre>** s'est finalement réinscrit en **<Tank / DPS / Heal / Tout>**.
```

### 16.3 Règles de mentions

- H-24 et H-30 peuvent mentionner Tank, DPS, Heal, Tout, Banc, Tard et Provisoire.
- Absence est exclu.
- Le message public de désistement utilise le nom du membre et ne le ping pas.
- Le H-3 mentionne directement le Leader et les Co-leaders fournis par Raid-Helper.

### 16.4 Cas volontairement silencieux

- inscription initiale en Absence ;
- changement de job uniquement ;
- Banc / Tard → Absence ;
- rôle confirmé → Provisoire ;
- Banc / Tard → Provisoire ;
- Provisoire → Absence ;
- suppression d'une sortie déjà passée : nettoyage sans MP d'annulation.

## 17. Points de vigilance

- Uriel dépend de Discord et de Raid-Helper.
- Les secrets et tokens ne doivent jamais être publiés.
- La base SQLite de production doit être conservée lors des mises à jour.
- Des messages orphelins issus d'anciens bugs peuvent ne pas être récupérés si leur événement n'est plus connu du suivi actuel.
- Les textes H-24, H-3, H-2 et H-30 annoncent volontairement un délai fixe correspondant au rappel concerné.

## 18. Mise à jour du projet

À chaque évolution : tester localement, valider les scénarios concernés, déployer uniquement les fichiers nécessaires, vérifier la compilation et le service, contrôler les logs, puis mettre à jour cette documentation et l'[historique des versions](VERSIONS.md).

---

[← Retour à l'accueil](README.md)
