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

## 16. Catalogue exhaustif des messages envoyés par Uriel

Cette section recense **tous les messages utilisateur envoyés par la version 2.0.1 de `main.py`**.

Les valeurs dynamiques sont représentées entre chevrons :

- `<nom de la sortie>` : titre Raid-Helper de l'événement ;
- `<date et heure>` : horodatage Discord au format complet ;
- `<membre>` : nom du membre ;
- `<job>` : job Raid-Helper ;
- `<mentions>` : mentions Discord des membres concernés ;
- `<Leader>` / `<Co-leader>` : mentions des responsables de la sortie ;
- `<description Raid-Helper>` : contenu de la description de l’événement, conservé tel quel.

Les messages ci-dessous conservent le texte, les emojis, le gras, les retours à la ligne et les libellés utilisés par Uriel.

### 16.1 MP — inscription confirmée

**Déclenchement :** inscription dans un statut confirmé ou actif pris en charge par Uriel.

Le rôle affiché peut être **Tank**, **DPS**, **Heal**, **Tout**, **Banc** ou **Tard**.

```text
✅ **Inscription confirmée — <nom de la sortie>**
Tu es bien inscrit en **<rôle>**.

📅 <date et heure>

📝 **Informations sur la sortie**
<description Raid-Helper>

Tu peux ajouter cette sortie à ton calendrier avec le bouton ci-dessous.
```

Si la description Raid-Helper est vide, le bloc `📝 Informations sur la sortie` est entièrement omis.

Le MP contient également :

- le fichier calendrier `.ics` en pièce jointe ;
- un bouton lien Discord avec l'emoji `📅` ;
- le libellé exact du bouton :

```text
Ajouter à mon calendrier
```

### 16.2 MP — inscription en Provisoire

**Déclenchement :** inscription en statut Raid-Helper `Tentative`, affiché par Uriel comme **Provisoire**.

```text
🟠 **Inscription provisoire — <nom de la sortie>**
Tu es maintenant inscrit en **Provisoire** pour cette sortie.

⚠️ Si tu sais finalement que tu ne pourras pas participer, pense impérativement à te remettre en **Absence** sur Raid-Helper afin que l'organisation de la sortie reste claire.

📅 <date et heure>

📝 **Informations sur la sortie**
<description Raid-Helper>

Tu peux ajouter cette sortie à ton calendrier avec le bouton ci-dessous.
```

Si la description Raid-Helper est vide, le bloc `📝 Informations sur la sortie` est entièrement omis.

Le MP contient également le fichier `.ics` et le même bouton :

```text
Ajouter à mon calendrier
```

### 16.3 MP — confirmation de passage en Absence

**Déclenchement :** désistement d'un membre confirmé lorsque la logique de confirmation d'absence s'applique.

```text
Ton passage en **Absent** pour **<nom de la sortie>** a bien été pris en compte.

📅 <date et heure>
```

### 16.4 Salon — désistement tardif

**Déclenchement :** un membre confirmé passe en Absence dans les 24 heures précédant la sortie.

```text
⚠️ **<membre>** vient de se désinscrire de **<nom de la sortie>**.
Une place de **<rôle>** vient de se libérer !
```

Les valeurs possibles de `<rôle>` sont celles retournées par Uriel pour les rôles de combat :

```text
Tank
DPS
Heal
Tout
```

### 16.5 Salon — réinscription après désistement

Uriel **modifie le message public de désistement existant** au lieu d'en créer un nouveau.

```text
~~⚠️ **<membre>** s'était désinscrit de **<nom de la sortie>**.~~
~~Une place de **<rôle>** s'était libérée.~~

✅ **Mise à jour :** **<membre>** s'est finalement réinscrit en **<rôle>**.
```

Les valeurs possibles de `<rôle>` sont :

```text
Tank
DPS
Heal
Tout
```

### 16.6 Salon — modification de l'horaire

**Déclenchement :** modification de la date ou de l'heure Raid-Helper d'un événement déjà connu.

```text
📅 **Modification — <nom de la sortie>**
L'horaire de la sortie a été modifié.

Ancien horaire : <ancienne date et heure>
Nouvel horaire : <nouvelle date et heure>

<mentions>
```

La ligne `<mentions>` n'est ajoutée que lorsqu'Uriel a des membres à mentionner.

### 16.7 MP H-3 — membre toujours en Provisoire

**Déclenchement :** trois heures avant la sortie pour chaque membre encore en **Provisoire**.

```text
⏳ **Inscription provisoire — <nom de la sortie>**

La sortie commence dans **3 heures** et tu es toujours inscrit en **Provisoire**.

Merci de mettre à jour ton inscription sur **Raid-Helper dans l'heure**.

⚔️ **Tu participes ?**
Passe-toi en **Tank, Heal, DPS ou Tout**.

🪑 **Tu souhaites rester disponible en remplacement ?**
Passe-toi en **Banc**.

🕒 **Tu seras en retard ?**
Passe-toi en **Tard** et préviens <Leader> (Leader) ou <Co-leader> (Co-leader) de ton heure d'arrivée.

Le retard est limité à **30 minutes maximum**. Au-delà, passe-toi en **Absence**.

❌ **Tu ne seras pas disponible ?**
Passe-toi en **Absence**.

⚠️ **Merci de ne pas rester en Provisoire une fois ta situation connue.**
```

S'il existe plusieurs Co-leaders, Uriel ajoute chaque responsable à la partie correspondante.

Si Raid-Helper ne fournit exceptionnellement aucun responsable, la phrase devient exactement :

```text
Passe-toi en **Tard** et préviens un responsable de la sortie de ton heure d'arrivée.
```

### 16.8 MP H-2 — récapitulatif Leader / Co-leaders

**Déclenchement :** deux heures avant la sortie.

Le même récapitulatif est envoyé individuellement au Leader et à chaque Co-leader.

La structure complète possible est :

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

Chaque catégorie vide est totalement absente du MP.

Ainsi, les variantes réellement possibles sont composées uniquement des blocs qui contiennent au moins un membre :

#### Bloc Inscrits

```text
⚔️ **Inscrits**
- <membre> — <job>
```

Si Raid-Helper ne fournit pas de job pour un inscrit confirmé, Uriel affiche exactement :

```text
- <membre> — Job inconnu
```

#### Bloc Banc

```text
🪑 **Banc**
- <membre>
```

#### Bloc En retard

```text
🕒 **En retard**
- <membre>
```

#### Bloc Toujours en Provisoire

```text
⚠️ **Toujours en Provisoire**
- <membre>
```

Les membres en **Absence** ne figurent pas dans ce récapitulatif.

### 16.9 Salon H-24 — rappel général

**Déclenchement :** 24 heures avant la sortie.

```text
🔔 **Rappel — <nom de la sortie>**
La sortie commence dans 24 heures !

<mentions>

⚠️ **Si vous ne pouvez finalement pas participer, merci de vous passer en `Absence` sur Raid-Helper afin de libérer votre place.**
```

Si aucune mention n'est disponible, Uriel n'ajoute pas le bloc `<mentions>`.

Les statuts pouvant être mentionnés sont :

```text
Tank
DPS
Heal
Tout
Banc
Tard
Provisoire
```

Les membres en **Absence** sont exclus.

### 16.10 Salon H-30 — rappel de départ

**Déclenchement :** 30 minutes avant la sortie.

```text
⏰ **<nom de la sortie> commence dans 30 minutes !**
<mentions>
```

Si aucune mention n'est disponible, seule la première ligne est envoyée.

Les statuts pouvant être mentionnés sont :

```text
Tank
DPS
Heal
Tout
Banc
Tard
Provisoire
```

Les membres en **Absence** sont exclus.

### 16.11 MP — annulation d'une sortie future

**Déclenchement :** suppression d'une sortie future suivie par Uriel.

```text
❌ **Sortie annulée — <nom de la sortie>**
La sortie prévue le <date et heure> a été annulée.

📝 **Message de l'organisateur**
<description Raid-Helper>
```

Ce MP concerne les inscriptions encore susceptibles de participer :

```text
Tank
DPS
Heal
Tout
Banc
Tard
Provisoire
```

Si la dernière description connue est vide, le bloc `📝 Message de l'organisateur` est entièrement omis. Uriel conserve cette description en SQLite afin de pouvoir l'utiliser même après la suppression de l'événement.

Un membre déjà en **Absence** ne reçoit pas ce MP.

Une sortie déjà passée ne déclenche pas ce MP d'annulation.

## 17. Cas où Uriel reste volontairement silencieux

L'absence de message fait partie du comportement attendu dans plusieurs transitions.

| Situation | Message envoyé |
|---|---|
| Inscription initiale en Absence | Aucun |
| Changement de job uniquement | Aucun |
| Banc → Absence | Aucun |
| Tard → Absence | Aucun |
| Confirmé → Provisoire | Aucun |
| Banc → Provisoire | Aucun |
| Tard → Provisoire | Aucun |
| Provisoire → Absence | Aucun |
| Suppression d'une sortie déjà passée | Aucun MP d'annulation |
| Provisoire encore présent à H-3 | MP H-3 |
| Leader / Co-leader à H-2 | MP récapitulatif H-2 |

## 18. Résumé chronologique des messages autour d'une sortie

Pour une sortie normale, Uriel peut produire la séquence suivante :

1. **À l'inscription** : MP de confirmation ou MP Provisoire avec `.ics` et bouton calendrier.
2. **Lors d'un changement d'horaire** : message public de modification.
3. **À H-24** : rappel public avec mentions.
4. **À H-3** : MP individuel aux membres encore Provisoire.
5. **À H-2** : MP de récapitulatif au Leader et aux Co-leaders.
6. **À H-30** : rappel public de départ.
7. **En cas de désistement tardif** : MP de confirmation d'Absence et message public de place libérée.
8. **En cas de réinscription** : modification du message public de désistement.
9. **En cas d'annulation d'une sortie future** : MP individuel aux membres concernés.

---

Fin de la documentation — état **v2.0.1**
