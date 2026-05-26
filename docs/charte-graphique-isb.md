# Charte Graphique ISB

Extraite du projet `isb-dashboard` — à appliquer à toutes les applications intranet ISB.

---

## 1. Logo

Le logo ISB est un carré aux coins arrondis contenant les initiales **ISB** en blanc.

| Propriété | Valeur |
|---|---|
| Taille | 48 × 48 px |
| Border-radius | 10px |
| Fond | `#A0722A` (bois) |
| Texte | `ISB` — blanc, 17px, gras (700), letter-spacing 1.5px |

**Usage mobile :** réduire à 40 × 40 px, police 15px.

**Élément décoratif associé :** motif de polygones hexagonaux en SVG, placé en haut à droite du header, en couleur bois avec opacité décroissante (0.28 → 0.03). Renforce l'identité sans surcharger.

---

## 2. Palette de couleurs

### Couleurs fondamentales

| Nom | Variable | Hex | Usage |
|---|---|---|---|
| Fond principal | `--bg` | `#FFFFFF` | Arrière-plan de page, cartes |
| Fond doux | `--bg-soft` | `#F8F9FA` | Header admin, zones secondaires, footer |
| Texte principal | `--text` | `#1A1A1A` | Corps de texte, titres |
| Texte secondaire | `--muted` | `#6B7280` | Labels, métadonnées, texte désactivé |
| Bordure | `--border` | `#E8E8E8` | Séparateurs, contours de cartes |

### Couleurs de marque

| Nom | Variable | Hex | Usage |
|---|---|---|---|
| Bois *(couleur principale)* | `--wood` | `#A0722A` | Accents, onglet actif, logo, hover |
| Bleu | `--blue` | `#2B5BA8` | Liens, boutons info, panneaux détail |
| Vert | `--green` | `#3D7A3D` | Données environnementales / bois |

### Couleurs sémantiques

| Nom | Variable | Hex | Usage |
|---|---|---|---|
| Positif | `--positive` | `#27A365` | Hausse, OK, livraison reçue |
| Négatif | `--negative` | `#E53935` | Baisse, erreur, retard |

### Badges (fond teinté)

| Type | Fond | Texte |
|---|---|---|
| Positif | `rgba(39, 163, 101, 0.11)` | `#27A365` |
| Négatif | `rgba(229, 57, 53, 0.11)` | `#E53935` |
| Neutre | `#F8F9FA` | `#6B7280` |

---

## 3. Typographie

| Propriété | Valeur |
|---|---|
| Police principale | **Inter** (Google Fonts) |
| Fallback | `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` |
| Taille de base | 14px |
| Line-height | 1.5 |
| Graisses utilisées | 400 (normal), 500 (medium), 600 (semibold), 700 (bold) |

### Hiérarchie typographique

| Niveau | Taille | Graisse | Autre |
|---|---|---|---|
| Titre de page | 20px | 700 | — |
| Sous-titre / subtitle | 12px | 400 | couleur `--muted` |
| Titre de section | 11px | 600 | uppercase, letter-spacing 0.5px |
| Titre de carte/graphe | 12px | 600 | uppercase, letter-spacing 0.4px |
| Valeur KPI | 26px | 700 | tabular-nums |
| Unité KPI | 12px | 400 | couleur `--muted` |
| Corps de texte | 13–14px | 400–500 | — |
| Label petit | 10–11px | 600 | uppercase, letter-spacing 0.5px |

---

## 4. Espacements & mise en page

| Propriété | Valeur |
|---|---|
| Largeur max du contenu | 1440px, centré |
| Padding horizontal (desktop) | 32px |
| Padding horizontal (mobile) | 16px |
| Border-radius des cartes | 8px (`--radius`) |
| Gap grille KPI | 16px |
| Gap grille graphiques | 20px |
| Padding interne des cartes | 18–22px |

---

## 5. Composants

### Cartes (KPI, graphiques)

- Fond blanc, bordure `1px solid #E8E8E8`, radius 8px
- Ombre : `0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.04)`
- **Cartes KPI :** bande d'accent verticale gauche de 3px en couleur `--wood`

### Header

- Fond blanc, sticky, bordure inférieure `1px solid #E8E8E8`
- Zone `--bg-soft` utilisée pour les zones secondaires (admin, footer)

### Navigation par onglets

- Fond blanc, bordure inférieure globale
- Onglet inactif : texte `--muted`, bordure inférieure transparente
- Onglet actif : texte `--wood`, bordure inférieure `3px solid #A0722A`, graisse 600
- Numéro d'onglet : `10px`, opacité 0.55, masqué sur mobile

### Boutons

- Style minimal : pas de fond, bordure `1px solid #E8E8E8`, radius 8px
- Hover : bordure et texte passent en `--wood`
- État désactivé : opacité 0.5

### Sélecteurs (select)

- Fond `--bg-soft`, bordure `--border`, radius 5px
- Police 11px, 500
- Flèche SVG personnalisée (`#6B7280`)
- Focus/hover : bordure `--wood`

### Panneau latéral (drawer)

- Largeur 360px, fixe à droite, hauteur pleine page
- Fond blanc, bordure gauche `1px solid --border`
- Ombre : `-4px 0 24px rgba(0,0,0,0.10)`
- Animation : slide depuis la droite, `0.22s cubic-bezier(0.4, 0, 0.2, 1)`
- Header interne sur fond `--bg-soft`

### Skeleton / chargement

- Gradient shimmer : `linear-gradient(90deg, --border 25%, --bg-soft 50%, --border 75%)`
- Animation 1.4s infinie
- Radius 4px

---

## 6. Animations & transitions

| Élément | Durée | Easing |
|---|---|---|
| Hover (couleurs, bordures) | 0.15s | linéaire |
| Panneau latéral (slide) | 0.22s | `cubic-bezier(0.4, 0, 0.2, 1)` |
| Icône refresh (rotation) | 0.75s | linéaire infini |
| Skeleton shimmer | 1.4s | infini |

---

## 7. Responsive

| Breakpoint | Changements |
|---|---|
| ≤ 900px | Grille graphiques passe à 1 colonne |
| ≤ 768px | Padding 16px, KPI en 2 colonnes, motif hex réduit et semi-transparent, numéros d'onglets masqués |
| ≤ 480px | KPI en 1 colonne, logo réduit (40px), bloc "mis à jour" masqué |

---

## 8. Alertes — couleurs spécifiques aux applications métier

Pour les apps de relance / suivi fournisseurs, utiliser ce code couleur cohérent avec la palette ISB :

| Niveau | Couleur | Hex | Usage |
|---|---|---|---|
| 🔴 Critique / Dépassé | Négatif | `#E53935` | Livraison en retard |
| 🟠 Attention | Orange | `#F57C00` | Sans confirmation depuis X jours |
| 🟡 Vigilance | Ambre | `#F9A825` | Délai < 7 jours sans confirmation |
| 🟢 OK | Positif | `#27A365` | Commande confirmée / livrée |

---

## 9. Variables CSS — copier-coller

```css
:root {
  --bg:       #FFFFFF;
  --bg-soft:  #F8F9FA;
  --text:     #1A1A1A;
  --muted:    #6B7280;
  --wood:     #A0722A;
  --blue:     #2B5BA8;
  --green:    #3D7A3D;
  --positive: #27A365;
  --negative: #E53935;
  --border:   #E8E8E8;
  --shadow:   0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.04);
  --radius:   8px;
  --font:     'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}
```
