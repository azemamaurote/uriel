# 🪽 Uriel

**Uriel** est un assistant Discord privé développé pour la compagnie libre **Anges – Apocalypse** sur FINAL FANTASY XIV.

Il complète **Raid-Helper** afin de faciliter l'organisation et le suivi des sorties de la compagnie libre.

> **Version documentée : 2.1.2**

## 🆕 Dernières mises à jour

### v2.1.2 — Migration des messages et validation Oracle (24 octobre 2026)

- Déploiement coordonné de `main.py` et `database/database.py` sur Oracle Cloud, après sauvegarde du code et de SQLite.
- Migration des références de messages publics existants vers les fils Raid-Helper avec `uriel_migration_messages.py` (`preview` puis `apply`).
- Migration confirmée d'un message de **Canalisations silidiennes extrême — Boss 3 : Zeless Gah** vers son fil Discord.
- Redémarrage du service `uriel` réussi : connexion Discord, découverte des événements, surveillance des deux salons et contrôle de sécurité chaque minute confirmés dans les journaux.
- **Reste à confirmer en production :** le prochain envoi automatique d'une nouvelle notification dans le fil. Les scénarios correspondants ont été testés en développement.

### v2.1.1 — Notifications publiques dans les fils (18 octobre 2026)

- Les rappels publics **H-24 et H-30**, les changements de date/heure et les notifications de désistement/réinscription sont publiés dans le **fil Raid-Helper de la sortie**, plutôt que dans le salon parent.
- Correction des appels de suivi des absences et réinscriptions pour transmettre le contexte de l’événement au gestionnaire de messages publics.
- Tests de recette validés ; mise en production sur Oracle Cloud le 18 octobre 2026, connexion et surveillance confirmées par les journaux.
- Les **MP** et le nettoyage automatique **H+24** ne changent pas.


### v2.1.0 — Fils, rappels Endormi et nettoyage H+24

- Création automatique d'un **fil public unique** pour chaque événement Raid-Helper suivi.
- Conservation du fil lorsque l'événement est modifié ; suppression du fil lorsque l'événement est supprimé.
- **MP H-15 et H-5** aux inscrits éligibles disposant du rôle Discord **Endormi**.
- **Nettoyage automatique H+24**, calculé depuis l'heure de début de l'événement : message Raid-Helper, fil, messages Uriel et données de suivi ; sans MP d'annulation.
- Contrôle périodique **toutes les minutes** et rattrapage au démarrage.
- Mise en production sur Oracle Cloud, avec surveillance des salons **HL et Général**.

### v2.0.1

- 📝 La **description Raid-Helper** est ajoutée aux MP d’inscription.
- 🔗 Les liens et mentions de salons présents dans la description restent utilisables dans Discord.
- ❌ Lors d’une annulation, la dernière description connue devient le **message de l’organisateur**.
- 💾 La description est mémorisée dans SQLite pour rester disponible après la suppression de l’événement.

**[Voir toutes les nouveautés →](UPDATES.md)**

## 📚 Documentation

- **[Documentation complète](DOCUMENTATION.md)** — fonctionnement, règles métier, messages envoyés, architecture, stockage et exploitation.
- **[Historique des versions](VERSIONS.md)** — base v1.0.0 puis évolutions jusqu'à la v2.1.2.
- **[Politique de confidentialité](PRIVACY.md)**
- **[Conditions d'utilisation](TERMS.md)**

## ⚙️ Fonctionnalités principales

Uriel assure notamment :

- les confirmations d'inscription en message privé avec fichier calendrier `.ics` et informations de la description Raid-Helper ;
- la prise en charge des statuts Tank, DPS, Heal, Tout, Banc, Tard, Provisoire et Absence ;
- le rappel général H-24 et le rappel de départ H-30 dans le fil de chaque sortie ;
- le rappel privé H-3 aux membres encore inscrits en Provisoire ;
- le récapitulatif privé H-2 au Leader et aux Co-leaders ;
- le suivi des absences, inscriptions provisoires et désistements ;
- les notifications en cas de modification ou d'annulation d'une sortie ;
- le nettoyage de ses messages et données lorsqu'une sortie est supprimée.

**[Guide de déploiement et exploitation →](DEPLOIEMENT.md)**

## 💡 Évolution du projet

Uriel est un projet évolutif et participatif. La documentation publique est mise à jour avec les nouvelles versions afin de conserver l'historique du fonctionnement du bot.

Les membres d'**Anges – Apocalypse** peuvent proposer des idées et améliorations. Le développement et la maintenance étant réalisés bénévolement sur du temps personnel, les suggestions peuvent nécessiter du temps avant d'être étudiées ou mises en place.

## 👤 Projet

Uriel appartient à **Azem**, qui assure son administration, sa configuration et sa maintenance.

Le bot a été développé avec l'assistance de **ChatGPT**, à partir des besoins et règles définis par Azem. L'intelligence artificielle n'intervient pas dans le fonctionnement quotidien d'Uriel et n'analyse pas les inscriptions.

Uriel est un projet indépendant et n'est affilié ni à Discord, ni à Raid-Helper, ni à Square Enix ou FINAL FANTASY XIV.

---

**© 2026 Azem — Uriel**  
*Développé pour Anges – Apocalypse avec l'assistance de ChatGPT.*
