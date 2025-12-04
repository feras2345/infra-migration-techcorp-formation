# Architecture et Schémas - Migration TechCorp Formation

## 1. Architecture Actuelle - Data Center Montpellier

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DATA CENTER MONTPELLIER (ACTUEL)                     │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                     Hyperviseur Proxmox                           │  │
│  │  ┌──────────────────────┐         ┌──────────────────────┐       │  │
│  │  │  VM Authentification │         │  VM Plateforme Cours │       │  │
│  │  │                      │         │                      │       │  │
│  │  │  - Auth Users        │         │  - Gestion formations│       │  │
│  │  │  - LDAP/AD           │         │  - Interface web     │       │  │
│  │  │  - Session mgmt      │         │  - API REST          │       │  │
│  │  │  (~20 Go)            │         │  (~30 Go)            │       │  │
│  │  └──────────┬───────────┘         └──────────┬───────────┘       │  │
│  │             │                                 │                   │  │
│  └─────────────┼─────────────────────────────────┼───────────────────┘  │
│                │                                 │                      │
│  ┌─────────────┴─────────────────────────────────┴───────────────────┐  │
│  │                      Réseau Interne - VLAN 10                      │  │
│  └─────────────┬─────────────────────────────────┬───────────────────┘  │
│                │                                 │                      │
│  ┌─────────────┴──────────┐        ┌────────────┴──────────────────┐  │
│  │  PostgreSQL Database   │        │   Serveur de Fichiers         │  │
│  │                        │        │                               │  │
│  │  - Users (comptes)     │        │   - PDF (supports cours)      │  │
│  │  - Progressions        │        │   - Images (illustrations)    │  │
│  │  - Résultats examens   │        │   - Vidéos (compressées)      │  │
│  │  - Logs                │        │   - Documents pédagogiques    │  │
│  │  (~50-100 Go)          │        │   (~200-500 Go)               │  │
│  └────────────────────────┘        └───────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    Dashboard Interne (VPN)                        │  │
│  │  - Monitoring formateurs                                          │  │
│  │  - Support technique                                              │  │
│  │  - Stats plateformes (~5 Go)                                      │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                     Firewall + VPN Gateway                        │  │
│  └─────────────────────────────┬─────────────────────────────────────┘  │
└────────────────────────────────┼──────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │   Connexion Internet    │
                    │   (Bande passante       │
                    │    limitée)             │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
        ┌─────┴─────┐      ┌────┴─────┐      ┌────┴─────┐
        │  5000+    │      │Formateurs│      │ Support  │
        │   Users   │      │  (VPN)   │      │Technique │
        └───────────┘      └──────────┘      │  (VPN)   │
                                             └──────────┘
```

## 2. Architecture Cible - Data Center Toulouse

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DATA CENTER TOULOUSE (CIBLE)                         │
│                          Infrastructure Modernisée                      │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │              Hyperviseur Proxmox (Nouvelle Version)               │  │
│  │  ┌──────────────────────┐         ┌──────────────────────┐       │  │
│  │  │  VM Authentification │         │  VM Plateforme Cours │       │  │
│  │  │    (Migrée)          │         │      (Migrée)        │       │  │
│  │  │                      │         │                      │       │  │
│  │  │  - Auth Users        │         │  - Gestion formations│       │  │
│  │  │  - LDAP/AD           │         │  - Interface web     │       │  │
│  │  │  - Session mgmt      │         │  - API REST          │       │  │
│  │  │  (~20 Go)            │         │  (~30 Go)            │       │  │
│  │  └──────────┬───────────┘         └──────────┬───────────┘       │  │
│  │             │                                 │                   │  │
│  └─────────────┼─────────────────────────────────┼───────────────────┘  │
│                │                                 │                      │
│  ┌─────────────┴─────────────────────────────────┴───────────────────┐  │
│  │               Réseau Interne - VLAN 10 (Reconfiguré)              │  │
│  └─────────────┬─────────────────────────────────┬───────────────────┘  │
│                │                                 │                      │
│  ┌─────────────┴──────────┐        ┌────────────┴──────────────────┐  │
│  │  PostgreSQL Database   │        │   Serveur de Fichiers         │  │
│  │    (Primary - Migré)   │        │      (Synchronisé)            │  │
│  │                        │        │                               │  │
│  │  - Users (comptes)     │        │   - PDF (supports cours)      │  │
│  │  - Progressions        │        │   - Images (illustrations)    │  │
│  │  - Résultats examens   │        │   - Vidéos (compressées)      │  │
│  │  - Logs                │        │   - Documents pédagogiques    │  │
│  │  (~50-100 Go)          │        │   (~200-500 Go)               │  │
│  │                        │        │                               │  │
│  │  + Réplication         │        │   + Checksum validation       │  │
│  │  + Backup auto         │        │   + Versioning                │  │
│  └────────────────────────┘        └───────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                Dashboard Interne (VPN - Migré)                    │  │
│  │  - Monitoring formateurs                                          │  │
│  │  - Support technique                                              │  │
│  │  - Stats plateformes (~5 Go)                                      │  │
│  │  + Dynatrace/Grafana                                              │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │              Firewall + VPN Gateway (Reconfiguré)                 │  │
│  │              + IDS/IPS + Logs centralisés                         │  │
│  └─────────────────────────────┬─────────────────────────────────────┘  │
└────────────────────────────────┼──────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │  Connectivité Redondée  │
                    │  - Fibre Principale     │
                    │  - 4G Failover          │
                    │  (Haute disponibilité)  │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
        ┌─────┴─────┐      ┌────┴─────┐      ┌────┴─────┐
        │  5000+    │      │Formateurs│      │ Support  │
        │   Users   │      │  (VPN)   │      │Technique │
        └───────────┘      └──────────┘      │  (VPN)   │
                                             └──────────┘
```

## 3. Schéma de Migration - Vue d'ensemble du Processus

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        PROCESSUS DE MIGRATION                           │
└─────────────────────────────────────────────────────────────────────────┘

    MONTPELLIER                    LIAISON 200 Mbps                TOULOUSE
┌─────────────────┐                                          ┌──────────────────┐
│                 │                                          │                  │
│  ┌───────────┐  │    PHASE 1: Préparation (J-14 à J-7)   │  ┌────────────┐  │
│  │PostgreSQL │──┼──────────────────────────────────────────▶│ PostgreSQL │  │
│  │  (Prod)   │  │  Streaming Replication (pg_basebackup) │  │  (Standby) │  │
│  └───────────┘  │                                          │  └────────────┘  │
│                 │                                          │                  │
│  ┌───────────┐  │    PHASE 2: Sync Fichiers (J-7 à J-1)  │  ┌────────────┐  │
│  │ Fichiers  │──┼──────────────────────────────────────────▶│  Fichiers  │  │
│  │(200-500Go)│  │       rsync --progress --stats          │  │  (Copie)   │  │
│  └───────────┘  │                                          │  └────────────┘  │
│                 │                                          │                  │
│  ┌───────────┐  │    PHASE 3: Export VM (J-3 à J-1)      │  ┌────────────┐  │
│  │ VM Auth + │──┼──────────────────────────────────────────▶│ VM Import  │  │
│  │ VM Cours  │  │    Proxmox vzdump + restore             │  │  Proxmox   │  │
│  └───────────┘  │                                          │  └────────────┘  │
│                 │                                          │                  │
│       ▼         │    JOUR J: Migration Finale (2h max)    │       ▲          │
│  ┌───────────┐  │                                          │  ┌────────────┐  │
│  │  ARRÊT    │  │  - Sync delta final                     │  │  DÉMARRAGE │  │
│  │ Services  │  │  - Dump PostgreSQL final                │  │  Services  │  │
│  └───────────┘  │  - Restore + Bascule DNS                │  └────────────┘  │
│                 │                                          │                  │
│  ┌───────────┐  │                                          │  ┌────────────┐  │
│  │ Standby   │◀─┼────────── Rollback possible ────────────┤  │ Production │  │
│  │(Backup 7j)│  │        (si échec critique)              │  │   Active   │  │
│  └───────────┘  │                                          │  └────────────┘  │
└─────────────────┘                                          └──────────────────┘
```

## 4. Timeline de Migration (Jour J)

```
22h00 ├─────────────────────────────────────────────────────────┤ 00h00
      │                                                         │
      │  T0: Mode Maintenance                                   │
      ├──┐                                                      │
      │  ├─ Arrêt connexions utilisateurs                       │
22h05 │  ├─ Shutdown VM graceful                                │
      │  │                                                      │
22h15 │  ├─ pg_dump final (15 min)                              │
      │  │                                                      │
22h20 │  ├─ rsync delta final (10 min)                          │
      │  │                                                      │
22h30 │  ├─ Transfert dump vers Toulouse (15 min)               │
      │  │                                                      │
22h45 │  ├─ pg_restore Toulouse (30 min)                        │
      │  │                                                      │
23h15 │  ├─ Start VM Toulouse (15 min)                          │
      │  │                                                      │
23h30 │  ├─ Tests validation (15 min)                           │
      │  │   • DB connectivity                                  │
      │  │   • VM health checks                                 │
      │  │   • Files integrity                                  │
      │  │                                                      │
23h45 │  ├─ Bascule DNS + VPN (10 min)                          │
      │  │                                                      │
00h00 │  └─ FIN - Services en ligne à Toulouse                  │
      │                                                         │
      └─────────────────────────────────────────────────────────┘
       ◄───────────── Fenêtre de 2h maximum ──────────────────▶
```

## 5. Flux de Données pendant la Migration

```
┌──────────────────────────────────────────────────────────────────────┐
│                    FLUX DE SYNCHRONISATION                           │
└──────────────────────────────────────────────────────────────────────┘

  AVANT MIGRATION (J-7 à J-1):                 
  
  Montpellier                                    Toulouse
  ┌──────────┐                                  ┌──────────┐
  │PostgreSQL│─────── Streaming Repl. ─────────▶│PostgreSQL│
  │ PRIMARY  │         (Continue)               │ STANDBY  │
  └──────────┘                                  └──────────┘
       │                                              ▲
       │ Write                                        │ Lag < 1 min
       ▼                                              │
  ┌──────────┐                                  ┌──────────┐
  │  Users   │                                  │ Readonly │
  └──────────┘                                  └──────────┘
  
  ┌──────────┐                                  ┌──────────┐
  │ Fichiers │────── rsync nightly ─────────────▶│ Fichiers │
  │  Source  │       (Delta sync)               │  Mirror  │
  └──────────┘                                  └──────────┘


  JOUR J (Bascule):                            
  
  Montpellier                                    Toulouse
  ┌──────────┐                                  ┌──────────┐
  │PostgreSQL│─────── Final dump ──────────────▶│PostgreSQL│
  │  STOPPED │       pg_restore                 │ PROMOTED │
  └──────────┘                                  └─────┬────┘
       ✗                                              │
    No Write                                          │ Write
                                                      ▼
  ┌──────────┐                                  ┌──────────┐
  │  Users   │────── DNS Cutover ──────────────▶│  Users   │
  │  OFFLINE │                                  │  ONLINE  │
  └──────────┘                                  └──────────┘
```

## 6. Architecture Réseau Détaillée

```
┌────────────────────────────────────────────────────────────────────┐
│                         TOULOUSE DC                                │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    Zone DMZ - VLAN 100                       │ │
│  │  ┌──────────────┐              ┌──────────────┐             │ │
│  │  │  Reverse     │              │  VPN Gateway │             │ │
│  │  │  Proxy       │              │  OpenVPN     │             │ │
│  │  │  (nginx)     │              │              │             │ │
│  │  └──────┬───────┘              └──────┬───────┘             │ │
│  └─────────┼────────────────────────────┼───────────────────────┘ │
│            │                            │                         │
│  ┌─────────┴────────────────────────────┴───────────────────────┐ │
│  │               Firewall Principal (pfSense)                   │ │
│  │  Rules: HTTP/HTTPS (80,443) + VPN (1194) + SSH (22)         │ │
│  └─────────┬────────────────────────────┬───────────────────────┘ │
│            │                            │                         │
│  ┌─────────┴────────────────────────────┴───────────────────────┐ │
│  │                    Zone App - VLAN 10                        │ │
│  │                                                              │ │
│  │  ┌──────────────┐              ┌──────────────┐             │ │
│  │  │ VM Auth      │              │ VM Cours     │             │ │
│  │  │ 192.168.10.5 │◀────────────▶│ 192.168.10.6 │             │ │
│  │  └──────┬───────┘              └──────┬───────┘             │ │
│  │         │                             │                     │ │
│  └─────────┼─────────────────────────────┼─────────────────────┘ │
│            │                             │                       │
│  ┌─────────┴─────────────────────────────┴─────────────────────┐ │
│  │                    Zone Data - VLAN 20                      │ │
│  │                                                             │ │
│  │  ┌──────────────┐              ┌──────────────┐            │ │
│  │  │ PostgreSQL   │              │ File Server  │            │ │
│  │  │ 192.168.20.5 │              │ 192.168.20.6 │            │ │
│  │  │ Port: 5432   │              │ NFS/SMB      │            │ │
│  │  └──────────────┘              └──────────────┘            │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                  Zone Management - VLAN 99                  │ │
│  │  ┌──────────────┐              ┌──────────────┐            │ │
│  │  │ Monitoring   │              │ Backup Server│            │ │
│  │  │ Grafana      │              │              │            │ │
│  │  └──────────────┘              └──────────────┘            │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Connectivité Redondée                          │ │
│  │  [Fibre Primary] ──┬──▶ Internet                           │ │
│  │  [4G Backup]    ───┘                                        │ │
│  └─────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────┘
```
