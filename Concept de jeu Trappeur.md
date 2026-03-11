# ***PROJET : TRAPPEUR - SIMULATION XVIIIe SIÈCLE***

---

# ***1. VISION DU GAME DESIGN***
Le jeu simule la vie d'un trappeur indépendant et de son groupe (famille/recrues), naviguant dans la rivalité commerciale entre la **HBC** (Hudson's Bay Company) et la **NWC** (North West Company).

* **Moteur :** Godot Engine.
* **Système :** Grille hexagonale tactique.
* **Contexte :** Amérique du Nord, 1750-1821.

---

# ***2. SPÉCIFICATIONS DE LA GRILLE (HEX-GRID)***
L'échelle de l'hexagone est calibrée pour la survie et la navigation tactique.

| Paramètre | Valeur / Type | Justification Technique |
| :--- | :--- | :--- |
| **Taille d'un Hex** | **500 mètres** | Équilibre visibilité (FOV) et micro-gestion du terrain. |
| **Coordonnées** | `Axial` (q, r) | Optimise le pathfinding (A*) sous Godot. |
| **Vitesse de Base** | 8 hexagones / heure | Basé sur une marche de 4 km/h en terrain plat. |

---

# ***3. SYSTÈME DE TEMPS (LOGIQUE DU TOUR)***
Le jeu utilise une échelle horaire pour renforcer la tension liée à la survie.

* **1 Tour = 1 Heure.**
* **Cycle Jour/Nuit :** Impacte la visibilité et la température.
* **Saisonnalité :** Le nombre de tours de "jour" (action sécurisée) varie selon le mois (ex: 16h en été, 8h en hiver).
* **Actions Temporelles :** * *Déplacement :* 1 tour (distance selon biome).
    * *Poser des pièges :* 2 tours.
    * *Établir un campement :* 1 tour.
    * *Portage :* Très coûteux en tours et en énergie.

---

# ***4. BIOMES ET MÉCANIQUES DE DÉPLACEMENT***
Le coût total d'entrée dans un hexagone ($C_{total}$) est calculé dynamiquement :
$$C_{total} = (C_{base} \times M_{saison}) + M_{meteo}$$

| Biome | Coût de Base ($C_{base}$) | Note Historique |
| :--- | :--- | :--- |
| **Plaine** | 1.0 | Déplacement facile. |
| **Forêt Boréale** | 2.5 | Riche en ressources, progression lente. |
| **Marécage** | 4.0 | Danger élevé, quasi-impraticable hors gel. |
| **Rivière** | 0.5 (Canoë) | Autoroute fluviale (Navigation). |

---

# ***5. MÉLÉE ÉCONOMIQUE ET SURVIE***

### ***L'Économie du "Made Beaver" (MB)***
* Le castor est l'unité de compte. Tout équipement (fusils, poudre, raquettes) s'achète en peaux de castor.
* **Sur-trappe :** Une variable `beaver_population` diminue sur l'hexagone à chaque prise.

### ***Météo et Saisons***
* **Hiver :** Les rivières deviennent des routes de glace (marche possible, canoë impossible).
* **Raquettes :** Objet indispensable pour annuler le malus de mouvement sur la neige.
* **Énergie :** Chaque tour consomme des calories. Le froid multiplie cette consommation.

---

# ***6. ARCHITECTURE GODOT SUGGÉRÉE***

```gdscript
# hex_data.gd
class_name HexData extends Resource

@export var biome_type: String = "Forest"
@export var base_friction: float = 2.5
@export var beaver_population: float = 100.0
@export var has_river: bool = false

# time_manager.gd
var current_hour: int = 8
var season: String = "Autumn"

func end_turn(action_cost: int):
    current_hour += action_cost
    if current_hour >= 24:
        # Logique de changement de jour