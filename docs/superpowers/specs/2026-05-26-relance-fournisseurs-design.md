# Design — Application de Relance Fournisseurs

**Date :** 2026-05-26  
**Statut :** Approuvé  

---

## Contexte

Application intranet permettant aux acheteurs de relancer automatiquement les fournisseurs en retard ou sans confirmation de livraison, à partir des données de l'ERP AS400. L'app envoie des emails pré-remplis modifiables et permet de mettre à jour les dates confirmées via une table de staging synchronisée avec l'AS400 par un job IT.

---

## Architecture globale

```
┌─────────────────────────────────────────────────────┐
│                SERVEUR LINUX (intranet)              │
│                                                     │
│  ┌─────────────┐    ┌──────────────┐               │
│  │  Django App  │────│  PostgreSQL  │               │
│  │  (Gunicorn)  │    │  (staging)   │               │
│  └──────┬──────┘    └──────────────┘               │
│         │ Nginx (reverse proxy)                     │
└─────────┼───────────────────────────────────────────┘
          │
          ├──── ODBC (lecture seule) ──────► AS400 / DB2
          ├──── LDAP ──────────────────────► Active Directory
          └──── SMTP ──────────────────────► Exchange Server

                    ┌──────────────────┐
                    │  Job AS400 (IT)  │◄── lit staging PostgreSQL
                    │  (synchronisation│    écrit dans ERP
                    │   planifiée)     │
                    └──────────────────┘
```

**Flux principal :**
1. L'utilisateur se connecte avec ses identifiants Windows (LDAP/AD)
2. Django lit les commandes en alerte depuis l'AS400 (ODBC read-only)
3. L'utilisateur sélectionne les fournisseurs à relancer, édite le mail, envoie
4. Après réponse du fournisseur, l'utilisateur saisit la nouvelle date → écrite dans PostgreSQL (staging)
5. Le job AS400 (piloté par l'IT) intègre les mises à jour dans l'ERP

---

## Stack technique

| Composant | Choix |
|---|---|
| Framework | Django 5.x |
| Base locale | PostgreSQL 16 |
| Serveur web | Gunicorn + Nginx |
| Auth AD | `django-auth-ldap` |
| Connexion AS400 | `pyodbc` + driver IBM iSeries Access ODBC |
| Envoi email | `django.core.mail` → SMTP Exchange |
| Interface | Django Templates + Bootstrap 5 |
| Admin | Django Admin natif personnalisé |

---

## Structure du projet

```
relance/
├── core/          # Connexion AS400, lecture commandes
├── relances/      # Logique métier, tableau de bord, envoi mails
├── fournisseurs/  # Gestion contacts fournisseurs
├── staging/       # Table tampon, interface synchro AS400
├── accounts/      # Auth LDAP/AD, rôles utilisateurs
└── config/        # Settings Django, admin personnalisé
```

---

## Règles métier

### 3 déclencheurs de relance

| Règle | Condition | Indicateur |
|---|---|---|
| **Livraison dépassée** | Date de livraison confirmée < aujourd'hui ET aucune réception enregistrée | 🔴 Rouge |
| **Délai critique** | Date de livraison prévue dans ≤ N jours ET pas de confirmation fournisseur (N paramétrable, défaut : 7) | 🟡 Jaune |
| **Sans confirmation** | Commande passée depuis ≥ X jours sans confirmation fournisseur (X paramétrable, défaut : 14) | 🟠 Orange |

Les trois règles sont activables/désactivables indépendamment depuis le panel admin.

---

## Interface utilisateur

### Écran 1 — Tableau de bord

- Liste des commandes en alerte, groupées par type (🔴 🟡 🟠)
- Colonnes : Fournisseur, Référence commande, Produit, Date prévue, Statut
- Cases à cocher pour sélection multiple
- Bouton « Relancer la sélection »
- Bouton « Mise à jour date » par ligne (après réponse fournisseur)

### Écran 2 — Rédaction et envoi de relance

- Un formulaire par commande sélectionnée (navigation par onglets si plusieurs)
- Champs : À (email contact fournisseur), Objet, Corps
- Pré-rempli depuis le template actif avec les variables : `{ref_commande}`, `{fournisseur}`, `{date_prevue}`, `{produit}`, `{type_alerte}`
- Tous les champs sont modifiables avant envoi
- Boutons : « Envoyer » / « Ignorer »

### Écran 3 — Saisie nouvelle date confirmée

- Accessible depuis le tableau de bord, par commande
- Champs : Nouvelle date confirmée (date picker), Note interne (texte libre)
- Enregistrement dans la table staging PostgreSQL
- Statut visible dans le tableau de bord : « En attente synchro AS400 »

### Écran 4 — Historique

- Liste de toutes les relances envoyées
- Colonnes : Date envoi, Utilisateur, Fournisseur, Référence, Type de relance, Suite donnée
- Filtres par fournisseur, période, utilisateur

---

## Panel d'administration

Accessible aux utilisateurs avec le rôle `Admin`, basé sur Django Admin.

### Paramètres métier
- Seuil alerte orange : nombre de jours sans confirmation (défaut : 14)
- Seuil alerte jaune : nombre de jours avant livraison (défaut : 7)
- Activation/désactivation de chaque règle de déclenchement

### Templates email
- Éditeur de templates avec variables dynamiques
- Plusieurs templates nommés possibles (ex. « 1ère relance », « Relance urgente »)
- Objet et corps configurables

### Fournisseurs
- Email(s) de contact par fournisseur
- Nom du contact
- Activation/désactivation d'un fournisseur (l'exclure des alertes)

### Utilisateurs
- Rôles : `Acheteur` (envoi de relances), `Admin` (accès panel complet)
- Les comptes proviennent de l'AD, les rôles sont assignés dans l'app

### Logs
- Consultation de toutes les actions depuis le panel admin

---

## Sécurité

| Couche | Mesure |
|---|---|
| **Authentification** | LDAP/AD uniquement — aucun compte local Django |
| **AS400** | Compte ODBC dédié en lecture seule, accès limité aux tables nécessaires |
| **Écriture ERP** | Jamais depuis l'app — uniquement vers PostgreSQL staging |
| **Synchronisation** | Job AS400 côté IT, l'ERP contrôle ce qui est intégré |
| **Réseau** | App accessible uniquement sur l'intranet |
| **Emails** | SMTP interne Exchange, pas de relay externe |
| **Sessions** | Expiration automatique après inactivité |
| **Logs** | Toute action tracée avec l'identifiant utilisateur AD |

---

## Table de staging (PostgreSQL)

```sql
CREATE TABLE staging_date_update (
    id              SERIAL PRIMARY KEY,
    ref_commande    VARCHAR(50) NOT NULL,
    nouvelle_date   DATE NOT NULL,
    note_interne    TEXT,
    created_by      VARCHAR(100),  -- identifiant AD
    created_at      TIMESTAMP DEFAULT NOW(),
    synced_at       TIMESTAMP,     -- rempli par le job AS400 après synchro
    status          VARCHAR(20) DEFAULT 'pending'  -- pending / synced / error
);
```

---

## Déploiement Linux

- Nginx en reverse proxy (port 80/443 intranet)
- Gunicorn comme serveur WSGI
- PostgreSQL local
- Service systemd pour l'app Django
- Driver ODBC IBM iSeries Access installé sur le serveur
- Variables sensibles (credentials AS400, LDAP, SMTP) dans un fichier `.env` hors dépôt

---

## Ce qui est hors périmètre

- Application mobile
- Exposition sur internet
- Écriture directe dans l'AS400 depuis l'app
- Module de gestion des commandes (lecture seule de l'ERP)
