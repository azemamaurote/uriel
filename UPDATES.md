# 🆕 Dernières mises à jour — Uriel

[← Accueil](README.md) · [Documentation complète](DOCUMENTATION.md) · [Historique détaillé](VERSIONS.md)

Cette page présente les nouveautés d'Uriel en version courte. Pour le détail du fonctionnement, consulter la [documentation complète](DOCUMENTATION.md).

## v2.1.1 — Déploiement du 18 octobre 2026

### 💬 Notifications publiques regroupées dans les fils

Les rappels H-24/H-30, modifications d’horaire, désistements et réinscriptions sont désormais publiés ou mis à jour dans le fil Raid-Helper de chaque événement. Les salons parents restent réservés aux messages d’événement, sans déplacer les anciens messages.

### 🛠️ Correctif et validation

Correction du paramètre `event` dans les deux appels de gestion des messages publics (absence et réinscription). Tests locaux validés avant mise en production. Service Oracle Cloud redémarré avec succès le 18 octobre : connexion et surveillance de deux salons confirmées dans les logs. Le comportement en production des prochaines notifications reste à observer. MP et nettoyage H+24 inchangés.

---

## v2.1.0 — Déploiement du 12 octobre 2026

### 💬 Fils automatiques

Un fil public est créé pour chaque événement Raid-Helper suivi. Il reste associé à l'événement après modification et est supprimé lors du nettoyage.

### ⏰ Rappels Endormi H-15 et H-5

Les inscrits éligibles possédant le rôle Endormi reçoivent deux rappels privés supplémentaires, sans doublon.

### 🧹 Nettoyage automatique H+24

Vingt-quatre heures après l'heure de début, Uriel nettoie automatiquement le message Raid-Helper, le fil et les messages associés, sans envoyer de MP d'annulation. Vérification toutes les minutes et au redémarrage.

### 🚀 Production

Déploiement sur Oracle Cloud : salons HL et Général surveillés ; connexion du bot et création d'un fil vérifiées sur le serveur réel. Les autres nouvelles fonctions avaient été validées en environnement de test.

---

## v2.0.1

### 📝 Description dans les MP d’inscription

La description de la sortie renseignée dans **Raid-Helper** est désormais reprise dans les MP d’inscription confirmée et Provisoire. Les liens et mentions de salons Discord présents dans cette description restent directement utilisables par les membres.

### ❌ Motif d’annulation

Lorsqu’une sortie future est supprimée, Uriel ajoute au MP d’annulation la **dernière description connue** sous le titre « Message de l’organisateur ». L’organisateur peut ainsi modifier la description avant de supprimer l’événement afin d’expliquer la raison de l’annulation.

### 💾 Conservation de la description

Uriel mémorise la dernière description Raid-Helper dans SQLite. Elle reste donc disponible pour le MP d’annulation même lorsque l’événement n’est plus accessible via Raid-Helper. Si la description est vide, aucun bloc supplémentaire n’est affiché.

---

## v2.0.0

### ⏳ Rappel H-3 — Provisoire

Trois heures avant une sortie, Uriel contacte en privé les membres encore inscrits en **Provisoire** afin qu'ils mettent à jour leur situation : participation confirmée, Banc, Tard ou Absence.

### 📋 Récap H-2 — Responsables

Deux heures avant la sortie, le **Leader** et les **Co-leaders** reçoivent en privé un récapitulatif des inscrits, du Banc, des retardataires et des membres toujours en Provisoire.

### 🕒 Nouveau statut Tard

Uriel prend désormais en charge le statut **Tard** pour identifier les membres qui participeront avec un retard annoncé.

### ⚔️ Rôle Tout

Le rôle **Tout** est désormais reconnu par Uriel au même titre que Tank, Heal et DPS pour les inscriptions confirmées.

---

## Versions précédentes

### v1.0.3

- Stabilisation du statut **Provisoire**.
- Rappels H-24 et H-30 étendus à Banc et Provisoire.
- Renforcement de la détection et du nettoyage des sorties supprimées.

### v1.0.2

- Première prise en charge spécifique du statut **Provisoire**.

### v1.0.1

- Passage du rappel général et de la fenêtre de désistement tardif de **H-8 à H-24**.
- Sécurisation de la synchronisation au démarrage.

### v1.0.0

- Première version de production d'Uriel.
- Suivi Raid-Helper, inscriptions, rappels, désistements, calendriers `.ics` et persistance SQLite.

---

Pour le détail technique de chaque version, consulter **[l'historique complet](VERSIONS.md)**.
