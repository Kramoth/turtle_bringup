# turtle_bringup
![ROS2](https://img.shields.io/badge/ROS_2-jazzy-green?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square)
![License](https://img.shields.io/badge/Licence-MIT-lightgrey?style=flat-square)

Package ROS 2 (Jazzy) de régulation de la tortue TurtleSim. Il implémente une chaîne complète **bruit → filtrage → régulation** : une pose bruitée est publiée, filtrée par un filtre passe-bas, puis un nœud de régulation guide la tortue vers un waypoint.

---

## Architecture des nœuds

```
turtlesim_node
      │  /turtle1/pose (Pose brute)
      ▼
noisy_pose_publisher_node  ──►  /turtle1/pose_noisy
                                         │
                              filtered_pose_publisher_node
                                         │
                                /turtle1/pose_filtered
                                         │
                              set_way_point_node
                                         │
                              /turtle1/cmd_vel ──► turtlesim_node
```

---

## Nœuds

### `turtlesim_node` *(pkg: turtlesim)*
Simulateur de tortue standard de ROS 2.

---

### `filtered_pose_publisher_node`
Filtre passe-bas exponentiel appliqué sur la pose de la tortue.

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `alpha` | `double` | `0.1` | Coefficient du filtre (0 = très lissé, 1 = pas de filtrage) |

---

### `noisy_pose_publisher_node`
Ajoute un bruit uniforme à la pose publiée par TurtleSim, pour simuler des mesures imparfaites.

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `noise_min` | `double` | `-0.5` | Borne inférieure du bruit ajouté |
| `noise_max` | `double` | `0.5` | Borne supérieure du bruit ajouté |

---

### `set_way_point_node`
Régulateur proportionnel qui dirige la tortue vers un waypoint cible. Utilise la pose **filtrée** via un remap.

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `Kpl` | `double` | `1.5` | Gain proportionnel sur la vitesse linéaire |
| `Kp` | `double` | `20.0` | Gain proportionnel sur la vitesse angulaire |

> **Remap :** `/turtle1/pose` → `/turtle1/pose_filtered`

---

## Prérequis

- ROS 2 **Jazzy Jalisco**
- Package `turtlesim` installé :
  ```bash
  sudo apt install ros-jazzy-turtlesim
  ```

---

## Installation

```bash
# Cloner le dépôt dans votre workspace
cd ~/ros2_ws/src
git clone https://github.com/Kramoth/turtle_bringup.git

# Compiler
cd ~/ros2_ws
colcon build --packages-select turtle_bringup

# Sourcer
source install/setup.bash
```

---

## Lancement

```bash
ros2 ros2 launch turtle_bringup turtle_regulation_bringup.launch.xml
```

---

## Topics

| Topic | Type | Description |
|-------|------|-------------|
| `/turtle1/pose` | `turtlesim/msg/Pose` | Pose brute publiée par TurtleSim |
| `/turtle1/pose_noisy` | `turtlesim/msg/Pose` | Pose avec bruit ajouté |
| `/turtle1/pose_filtered` | `turtlesim/msg/Pose` | Pose filtrée (filtre passe-bas) |
| `/turtle1/cmd_vel` | `geometry_msgs/msg/Twist` | Commande en vitesse envoyée à la tortue |