# 🧾 Historique des versions — Uriel

[← Accueil](README.md) · [Documentation complète](DOCUMENTATION.md)

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

### État

La v1.0.3 est la version actuellement documentée en production.

---

Pour le fonctionnement complet actuel, consulter la **[documentation](DOCUMENTATION.md)**.
