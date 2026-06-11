# Relance Fournisseurs — ISB

Application intranet de relance automatique des fournisseurs, connectée à l'ERP AS400.

## Problème

Les acheteurs ISB gèrent les commandes fournisseurs depuis l'AS400. Quand une livraison est en retard ou sans confirmation, ils doivent manuellement relancer par email et mettre à jour les dates dans l'ERP — processus chronophage et sans traçabilité.

## Solution

Une application Django qui :
1. **Détecte les alertes** — lit les commandes depuis l'AS400 et applique 3 règles (livraison dépassée, délai critique, sans confirmation)
2. **Automatise les relances** — sélection groupée, templates personnalisables, envoi SMTP avec anti-doublon
3. **Synchronise l'ERP** — les nouvelles dates fournisseur passent par une table staging, un service push les écrit dans l'AS400 (via ODBC) toutes les 5 minutes

## Architecture

```
AS400 (DB2) ← ODBC → Django App ←→ PostgreSQL ← Sync Push Service → AS400
                        ↑
                   AD / LDAPS (auth)
                        ↑
                   SMTP Exchange (envoi emails)
```

## Stack

| Composant | Technologie |
|---|---|
| Backend | Django 5.x + Gunicorn |
| Base locale | PostgreSQL 16 |
| ERP | IBM i (AS400) via ODBC + SQLAlchemy |
| Auth | LDAPS / Active Directory |
| Email | SMTP STARTTLS Exchange |
| Serveur | Nginx (reverse proxy) + systemd |

## Fonctionnalités

- **Dashboard** — commandes en alerte classées par priorité 🔴🟡🟠 avec sélection, filtres, badges de statut
- **Rédaction groupée** — un email par fournisseur avec templates et variables dynamiques
- **Staging AS400** — saisie de date → table staging → push automatique vers l'ERP
- **Historique** — traçabilité complète des relances envoyées
- **Administration** — seuils, templates, fournisseurs, utilisateurs, audit

## Démarrage

```bash
# Prérequis : Python 3.12+, PostgreSQL 16, driver ODBC IBM i Access (ACS)
git clone https://github.com/ISB-France/relance-fournisseur.git
cd relance-fournisseur
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # configurer les accès
python manage.py migrate
python manage.py runserver
```

## Structure

```
relance/
├── core/          # Connexion AS400, lecture commandes
├── relances/      # Dashboard, envoi emails groupés
├── fournisseurs/  # Gestion contacts fournisseurs
├── staging/       # Table tampon, suivi synchronisation
├── sync/          # Service push : staging → AS400
├── accounts/      # Auth LDAP/AD, rôles
└── config/        # Settings Django
```

## Projet

Développé par **ISB-France** — usage interne uniquement.
