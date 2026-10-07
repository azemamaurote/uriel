# 🆕 Dernières mises à jour — Uriel

[← Accueil](README.md) · [Documentation complète](DOCUMENTATION.md) · [Historique détaillé](VERSIONS.md)

Cette page présente les nouveautés d'Uriel en version courte. Pour le détail du fonctionnement, consulter la [documentation complète](DOCUMENTATION.md).

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
