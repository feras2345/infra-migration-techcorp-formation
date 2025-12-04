# Plan de Migration - TechCorp Formation

## 1. Analyse de l'existant

### 1.1 Composants à migrer

| Composant | Description | Taille estimée | Criticité |
|-----------|-------------|----------------|-----------|
| Base PostgreSQL | Comptes, progressions, résultats d'examen | ~50-100 Go | Critique |
| Serveur de fichiers | PDF, images, vidéos compressées | ~200-500 Go | Haute |
| VM Authentification | Service d'authentification utilisateurs | ~20 Go | Critique |
| VM Plateforme Cours | Gestion des formations en ligne | ~30 Go | Critique |
| Dashboard Interne | Service web accessible via VPN | ~5 Go | Moyenne |

### 1.2 Dépendances identifiées

- VM Authentification dépend de PostgreSQL pour la validation des identifiants.
- VM Plateforme Cours dépend de PostgreSQL et du serveur de fichiers.
- Le dashboard interne dépend des deux VM et de l'accès VPN.

### 1.3 Contraintes

- Disponibilité : fenêtre de coupure maximale de 2h (week-end).
- Bande passante : liaison dédiée 200 Mbps.
- Sécurité : données utilisateurs sensibles (RGPD), accès VPN à reconfigurer.
- Performance : plus de 5000 utilisateurs actifs, reprise lundi matin impérative.

## 2. Choix de la stratégie de migration

### 2.1 Comparaison des stratégies

| Stratégie | Avantages | Inconvénients | Adapté au contexte |
|-----------|-----------|---------------|-------------------|
| Big Bang | Simple, une seule bascule | Risque élevé, rollback difficile | Non |
| Migration progressive | Risque réduit, tests continus | Complexité, synchronisation | Partiellement |
| Réplication + bascule | Downtime minimal, rollback plus simple | Nécessite préparation et réplication | Recommandé |
| Lift and Shift | Conservation de l'existant | Ne corrige pas les limites actuelles | Partiellement |

### 2.2 Stratégie retenue : réplication anticipée + bascule rapide

La stratégie retenue repose sur :

- Pré-réplication des données (fichiers, DB, snapshots) plusieurs jours avant la migration.
- Synchronisation delta finale durant la fenêtre de 2h.
- Possibilité de rollback en réactivant l'ancien site de Montpellier.

## 3. Étapes du plan de migration

### 3.1 Préparation (J-14 à J-1)

Semaine 1 (J-14 à J-7) :

- Audit complet de l'infrastructure de Montpellier.
- Inventaire des volumes de données.
- Test de bande passante réelle entre les deux sites.
- Configuration réseau de Toulouse (VLANs, firewall, VPN).
- Installation d'une infrastructure Proxmox équivalente à Toulouse.
- Mise en place de la réplication PostgreSQL (streaming replication).

Semaine 2 (J-7 à J-1) :

- Première synchronisation complète des fichiers via rsync.
- Export et transfert des snapshots VM Proxmox.
- Configuration du VPN côté Toulouse.
- Tests de connectivité et de latence.
- Synchronisations delta quotidiennes.
- Préparation des scripts de bascule DNS/IP.
- Communication de la maintenance planifiée aux utilisateurs.

### 3.2 Migration (Jour J - samedi)

Exemple de chronologie :

- 22h00 : Activation du mode maintenance sur la plateforme.
- 22h05 : Arrêt des connexions utilisateurs.
- 22h10 : Arrêt propre des VM (shutdown).
- 22h15 : Dernière sauvegarde PostgreSQL (pg_dump, format custom).
- 22h20 : Synchronisation delta finale des fichiers via rsync.
- 22h30 : Transfert du dump PostgreSQL vers Toulouse.
- 22h45 : Restauration PostgreSQL à Toulouse (pg_restore).
- 23h15 : Démarrage des VM à Toulouse.
- 23h30 : Tests de validation rapides.
- 23h45 : Bascule DNS/IP.
- 00h00 : Fin de la fenêtre de maintenance.

### 3.3 Bascule (cutover)

- Modification des enregistrements DNS (TTL réduit en amont).
- Reconfiguration du VPN pour pointer vers Toulouse.
- Activation du monitoring sur la nouvelle infrastructure.
- Tests de fumée sur chaque service.

### 3.4 Post-migration (J+1 à J+7)

- Monitoring intensif (erreurs, performances, charge).
- Vérification régulière des logs applicatifs et systèmes.
- Tests de performance sous charge contrôlée.
- Validation métier avec les formateurs et le support.
- Documentation de la nouvelle architecture.
- Désactivation progressive du data center de Montpellier après validation.

## 4. Plan de tests

### 4.1 Tests pré-migration

| Test | Objectif | Critère de succès |
|------|----------|-------------------|
| Test bande passante | Valider le débit réel | ≥ 150 Mbps stable |
| Test réplication PostgreSQL | Vérifier le lag | Lag < 1 minute |
| Test rsync | Valider la synchronisation initiale | Delta < 5 Go |
| Test VPN Toulouse | Vérifier la connectivité | Accès dashboard OK |
| Test restauration snapshot | Valider la restauration des VM | VM fonctionnelle |

### 4.2 Tests pendant la migration

| Test | Fréquence | Action si échec |
|------|-----------|-----------------|
| Monitoring transfert | Continu | Pause et analyse |
| Vérification intégrité (checksums) | À chaque étape clé | Retransfert |
| Test connectivité DB | Après restauration | Rollback si problème persistant |

### 4.3 Tests post-migration

| Test | Description | Validation |
|------|-------------|------------|
| Intégrité données | Comparaison des enregistrements DB | 100 % concordants |
| Connexion utilisateur | Tests d'authentification | Connexion réussie |
| Accès formations | Navigation sur la plateforme | Toutes les formations accessibles |
| Lecture fichiers | Téléchargement PDF/vidéo | Fichiers lisibles |
| Performance | Test de charge (100 utilisateurs simultanés) | Temps de réponse < 2 s |
| VPN formateurs | Accès dashboard | Fonctionnel |

## 5. Plan de rollback

### 5.1 Critères de déclenchement

- Échec de restauration PostgreSQL après un délai défini.
- VM critiques ne démarrant pas après plusieurs tentatives.
- Perte de données ou incohérences détectées.
- Dépassement du temps de migration prévu.

### 5.2 Procédure de rollback

1. Arrêter immédiatement les services à Toulouse.
2. Vérifier l'intégrité des données à Montpellier.
3. Redémarrer les services sur le site de Montpellier.
4. Annuler les modifications DNS et réseau.
5. Informer les utilisateurs du report de la migration.
6. Analyser les causes et replanifier une nouvelle fenêtre.

## 6. Planning prévisionnel et ressources

### 6.1 Équipe projet

| Rôle | Responsabilité | Disponibilité |
|------|----------------|---------------|
| Chef de projet | Coordination, GO/NO-GO | J-14 à J+7 |
| Admin Système | Migration VM, Proxmox | J-7 à J+3 |
| DBA | PostgreSQL, réplication, restauration | J-7 à J+3 |
| Admin Réseau | VPN, DNS, firewall | J-3 à J+1 |
| Testeur | Validation fonctionnelle | J-1 à J+3 |

### 6.2 Planning global

- Semaine 1 (J-14) : Audit et préparation de l'infra Toulouse.
- Semaine 2 (J-7) : Mise en place de la réplication et synchronisations.
- Jour J (samedi) : Migration effective (22h00 - 00h00).
- Jour J+1 (dimanche) : Monitoring et tests supplémentaires.
- Jour J+2 (lundi) : Reprise du service avec support renforcé.
- Semaine J+7 : Clôture du projet et décommissionnement Montpellier.

## 7. Annexes

### 7.1 Outils envisagés

| Outil | Usage |
|-------|-------|
| rsync | Synchronisation des fichiers |
| pg_dump / pg_restore | Sauvegarde et restauration PostgreSQL |
| pg_basebackup | Mise en place de la réplication PostgreSQL |
| vzdump (Proxmox) | Snapshot et export des VM |
| scp / sftp | Transfert sécurisé de fichiers |
| Outils de monitoring | Surveillance post-migration |

### 7.2 Schéma d'architecture cible (simplifié)

Diagramme texte :

Data Center Toulouse :

- Proxmox (VM Authentification)
- Proxmox (VM Plateforme Cours)
- Serveur de fichiers (PDF, vidéos, images)
- Base PostgreSQL (primary)
- Firewall + VPN Gateway

Les utilisateurs, formateurs et équipes support accèdent aux services via Internet et VPN, avec une connectivité fibre et un backup 4G côté Toulouse.
