# ***PROJET : TRAPPEUR - SIMULATION XVIIIe SIÈCLE***

---

# ***1. VISION DU GAME DESIGN***
Le jeu simule la vie d'un trappeur indépendant, de sa famille et de ses recrues, naviguant dans la rivalité commerciale entre la **HBC** (Hudson's Bay Company) et la **NWC** (North West Company).

* **Moteur :** Godot Engine.
* **Système :** Grille hexagonale tactique.
* **Contexte :** Amérique du Nord, 1750-1821.

---

# ***2. SPÉCIFICATIONS DE LA GRILLE (HEX-GRID)***
L'unité de mesure de l'hexagone est le pilier de la simulation.

| Paramètre | Valeur / Type | Justification Technique |
| :--- | :--- | :--- |
| **Taille d'un Hex** | **500 mètres** | Équilibre visibilité (FOV) et temps de trajet réaliste. |
| **Coordonnées** | `Axial` (q, r) | Optimise le pathfinding (A*) et le stockage en mémoire. |
| **Vitesse de Base** | 8 à 10 Hex / Heure | Basé sur la marche avec charge lourde (4 km/h). |
| **Attributs par Hex** | `Resource` (.tres) | Biome, humidité, population de castors, influence politique. |

---

# ***3. MÉCANIQUES DE SIMULATION***

### ***L'Économie du "Made Beaver" (MB)***
Tout le système économique repose sur la peau de castor séchée, l'unité de compte historique.
* **Sur-trappe :** Une variable `float beaver_population` sur chaque hexagone qui diminue à chaque prise. Si elle tombe à 0, l'hexagone est stérile pour 3 cycles (années).
* **Commerce :** Les prix fluctuent selon la distance du Fort de traite le plus proche et l'allégeance du joueur.

### ***Logistique et Portage***
Le relief et les saisons dictent le gameplay via des multiplicateurs de coût de mouvement ($PM$) :
* **Navigation :** Coût de mouvement réduit sur les hexagones "Rivière".
* **Portage :** Action spécifique augmentant drastiquement la consommation d'énergie pour traverser un hexagone "Terre" avec un canoë.

---

# ***4. ALERTES ET ÉQUILIBRAGE***

### ***Risques de Design***
> [!IMPORTANT]
> **Alerte Micro-management :** Gérer chaque membre de la famille individuellement sur une grille de 500m est trop lourd.
> * **Solution :** Utiliser des "Ordres de groupe". La famille suit le trappeur par défaut, sauf lors de l'établissement du campement.

### ***Rigueur Historique***
* **Saisonnalité :** En hiver, les rivières gèlent. Le canoë devient inutile, les raquettes deviennent indispensables pour maintenir un coût de mouvement décent.

---

# ***5. ARCHITECTURE GODOT SUGGÉRÉE***

Pour tes scripts, structure tes hexagones comme des objets `Resource` pour optimiser la mémoire :

```gdscript
# Exemple de structure de donnée pour un Hexagone (hex_data.gd)
class_name HexData extends Resource

@export var biome_type: String = "Forest"
@export var has_river: bool = false
@export var beaver_density: float = 1.0 # De 0.0 à 1.0
@export var owner_company: String = "None" # "HBC" or "NWC"