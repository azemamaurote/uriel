# 🧾 Historique des versions — Uriel

[← Accueil](README.md) · [Documentation complète](DOCUMENTATION.md)

## v2.1.2 — Migration des messages publics existants (24 octobre 2026)

- Déploiement conjoint de `main.py` et `database/database.py` sur Oracle Cloud.
- Sauvegarde préalable de la base SQLite et des fichiers Python de production.
- Ajout d'une procédure ponctuelle de migration des messages publics historiques vers les fils Raid-Helper avec `uriel_migration_messages.py`.
- Exécution `preview` : **1 référence** à migrer ; exécution `apply` : **1 message migré, 0 erreur**.
- Présence du message migré dans le fil Discord confirmée manuellement.
- Service `uriel` actif après redémarrage ; logs de connexion et de surveillance sans erreur visible au démarrage.
- Tests de notifications dans les fils validés sur le Discord de développement ; **nouvelle notification automatique en production encore à observer**.

> Numéro de version documentaire ; ne présume pas de la création d'un tag Git ni d'un numéro de version embarqué dans le bot.

## v2.1.1 — Notifications publiques dans les fils (18 octobre 2026)

- Rappels publics H-24 et H-30 publiés dans le fil associé à l’événement Raid-Helper.
- Notifications publiques de changement de date/heure et de désistement dans ce même fil.
- Réinscription : mise à jour du message public existant dans le fil, sans nouveau message superflu.
- Correctif des deux appels à `set_public_message` : transmission de l’argument `event`.
- Tests de recette réussis, puis déploiement Oracle Cloud ; démarrage et connexion confirmés le 18 octobre 2026.
- Aucun changement prévu aux MP ni au nettoyage H+24.

> Version de documentation ; ne présume pas de la création d’un tag Git.

## v2.1.0 — Fils automatiques, Endormi et nettoyage H+24 (12 octobre 2026)

- Création automatique d'un fil public unique pour chaque événement Raid-Helper surveillé.
- Conservation du fil lors des modifications ; suppression lors du nettoyage de l'événement.
- MP H-15 et H-5 réservés aux inscrits éligibles portant le rôle Discord **Endormi** ; déduplication persistante.
- Suppression automatique **24 heures après l'heure de début** : message Raid-Helper, fil, messages Uriel et état SQLite ; **sans MP d'annulation**.
- Nettoyage au démarrage et contrôle périodique toutes les minutes.
- Activation des salons HL et Général de la CL, avec rôle Endormi du serveur de production.
- Déploiement Oracle Cloud confirmé : connexion Discord et création de fil sur un raid en cours vérifiées en production.
- Rappels Endormi et nettoyage H+24 validés lors des tests locaux ; leur comportement en conditions réelles reste à observer.

> Cette entrée décrit l'état du code déployé. Elle ne suppose pas qu'un tag Git `v2.1.0` a déjà été créé.

## v1.0.0 — Première version de production

La v1.0.0 pose les fondations d'Uriel :

- découverte et surveillance des événements Raid-Helper ;
- suivi des inscriptions ;
- MP de confirmation avec fichier calendrier `.ics` ;
- gestion des passages en Absence ;
- mémorisation du rôle précédant un désistement ;
- message public de place Tank / DPS / Heal libérée dans la fenêtre H-8 ;
- réutilisation du même message public en cas de réinscription ;
- rappel général H-8 ;
- rappel de départ H-30 ;
- détection des changements de date/heure ;
- annulation et nettoyage des messages Uriel ;
- persistance SQLite ;
- hébergement permanent sur Oracle Cloud avec `systemd`.

## v1.0.1 — H-24 et sécurisation du démarrage

- Le rappel général passe de **H-8 à H-24**.
- La fenêtre de désistement tardif passe également à **24 heures**.
- Les noms techniques des rappels sont généralisés pour ne plus dépendre d'un délai précis.
- La migration SQLite conserve l'état des anciens rappels afin d'éviter leur renvoi.
- La synchronisation de démarrage devient silencieuse pour les inscriptions déjà présentes.

## v1.0.2 — Introduction de Tentative / Provisoire

- Première prise en charge spécifique du statut **Tentative**.
- Protection contre un faux message de place libérée lors d'un passage `Tentative → Absence`.
- Ajout des scénarios `Tentative ↔ Absence` et `Tentative ↔ rôle confirmé`.
- Cette première logique sera affinée en v1.0.3 afin de ne pas assimiler Tentative à Absence.

## v1.0.3 — Stabilisation des statuts, rappels et suppressions

### Rappels

- Remplacement des timestamps relatifs Discord par des textes statiques.
- **Bench** et **Tentative** sont inclus dans les rappels H-24 et H-30.
- **Absence** reste exclu.

### Tentative / Provisoire

- `Tank/DPS/Heal → Tentative` : silencieux.
- `Bench → Tentative` : silencieux.
- `Tentative → Absence` : silencieux.
- `Absence → Tentative` : MP spécifique d'inscription provisoire.
- `Tentative → rôle confirmé` : inscription/réinscription.
- Tentative est désormais clairement distinct d'Absence.

### Suppressions

- Renforcement de la détection des suppressions effectuées depuis le dashboard Raid-Helper.
- Les événements connus sont vérifiés même lorsqu'ils disparaissent de la découverte Discord.
- Une réponse API `404` peut déclencher le nettoyage.
- Une sortie déjà passée reste suffisamment suivie pour pouvoir être nettoyée lorsqu'elle est supprimée.
- Une sortie passée supprimée ne génère pas de MP d'annulation.

## v2.0.0 — Suivi opérationnel avant sortie

### Statuts

- Prise en charge du choix **Tout** (`Allrounder`) comme participation confirmée.
- Prise en charge du statut **Tard** (`Late`).
- Les rappels H-24 et H-30 concernent désormais Tank, DPS, Heal, Tout, Banc, Tard et Provisoire.
- Absence reste exclu des rappels.

### H-3 — Régularisation des Provisoires

- Trois heures avant la sortie, chaque membre encore en **Provisoire** reçoit un MP.
- Le message demande de mettre à jour Raid-Helper dans l'heure : Tank, Heal, DPS, Tout, Banc, Tard ou Absence.
- En cas de retard, le membre doit prévenir le Leader ou un Co-leader de son heure d'arrivée.
- Le retard annoncé est limité à **30 minutes maximum** ; au-delà, le membre doit passer en Absence.
- Uriel ne transforme jamais automatiquement un Provisoire en Absence.
- L'envoi est mémorisé avec `tentative_reminder_sent`.

### H-2 — Récapitulatif des responsables

- Deux heures avant la sortie, Uriel envoie un MP au **Leader** et à chaque **Co-leader**.
- Le récapitulatif contient les inscrits confirmés avec leur job, puis les Banc, En retard et Toujours en Provisoire.
- Les catégories vides sont masquées.
- Les Absences sont exclues.
- L'envoi est mémorisé avec `leader_summary_sent`.

### Documentation

- La documentation contient désormais le **texte exact des messages générés par Uriel**, pour les MP comme pour les salons Discord.
- Les valeurs variables (nom de sortie, membre, rôle, job, date et mentions) sont représentées par des marqueurs explicites.

### État

La **v2.0.0** introduit le suivi opérationnel H-3 / H-2, le statut Tard et le rôle Tout.


## v2.0.1 — Exploitation de la description Raid-Helper

### Inscriptions

- Uriel récupère le champ `description` de l’événement Raid-Helper.
- La description est ajoutée aux MP d’inscription confirmée et Provisoire sous **« 📝 Informations sur la sortie »**.
- Les liens et mentions Discord présents dans la description sont conservés tels quels.
- Si la description est vide, le bloc n’est pas affiché.

### Annulations

- Uriel mémorise en permanence la dernière description connue de chaque événement.
- Lorsqu’une sortie future est supprimée, cette description est ajoutée au MP d’annulation sous **« 📝 Message de l'organisateur »**.
- L’organisateur peut donc modifier la description avant la suppression pour communiquer le motif de l’annulation.
- Une description vide n’ajoute aucun bloc au MP.

### Stockage

- Ajout de `event_description` dans la table SQLite `raid_helper_events`.
- La migration de la base existante est automatique ; le fichier `uriel.db` n’a pas besoin d’être remplacé.

### État

La **v2.0.1** est la version actuellement documentée en production.

---

Pour le fonctionnement complet actuel, consulter la **[documentation](DOCUMENTATION.md)**.
