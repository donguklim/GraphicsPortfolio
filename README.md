# 🎮 Grass Motion with Multi-Body Dynamics in Unreal Engine 5

> A physically-based grass animation system inspired by *Ghost of Tsushima* and implemented using Unreal Engine 5, Niagara, PCG, and multi-body dynamics.

## 📽️ Demo Video
[🔗 Watch on YouTube](https://youtu.be/5h7HZT5iuCI?si=WpGUy6z84sb_mj0Y)

![Grass Motion Sample 01](./resources/sample_01.gif "Grass Motion Sample 01")
![Grass Motion Sample 01](./resources/sample_02.gif "Grass Motion Sample 02")
![Grass Motion Sample 01](./resources/sample_03.gif "Grass Motion Sample 03")

## 🔗 Source Code
[GitHub Repository](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics) - includes README file with algorithmic details.


---
## Project Goal
- To implement Ghost of Tsushima grass with Unreal Engine 5
- To demonstrate capability of  Niagara Data Channel and PCG combination on 
runtime gpgpu simulation of a large number of instances.

---

## 🧠 Project Summary

- Based on the *Ghost of Tsushima* GDC presentation.
- Fully real-time grass generation and motion system using hierarchical PCG + Niagara + Physics.
- Enhanced physical realism with **Articulated Body Algorithm (ABA)**.
- Implemented unique improvements, including:
    - Fixed-length grass via **Quadratic Bézier Curve**
    - Physics based Dynamics motion with ABA
    - Physical constraints for joint angles
    - Added grass blade twist based on angular displacement

---

## 🛠️ Key Features

### 🌱 Grass Modeling & Optimization
- **Hierarchical PCG grid** for runtime spawn/cleanup.
- **Quadratic Bézier Curve** used for grass blades.
    - Fixed length maintained, unlike cubic version used in *Ghost of Tsushima*.

![Bezier Curve Grass Example](./resources/bezier_curve_example.jpg "Bezier Curve Grass Example")

**Bezier Curve Grass Example:** P0, P1 Bezier points works as rotational joints.

### 💨 Wind & Motion Simulation
- Wind simulated via noise functions.
- Skeleton-based articulation:
    - P₀: Ball joint (3 DOF)
    - P₁: Hinge joint (1 DOF)
- Elastic recovery toward initial pose, with randomized stiffness.
- Forces
  - Wind force (adjustable by UI)
  - Air friction force (adjustable by UI), works as damping force
  - Resotration force from grass joints

### ⚙️ Physics System
- Referenced but improved upon [SIGGRAPH paper](https://dl.acm.org/doi/10.1145/2856400.2876008)
- Discarded inaccurate or unstable parts of the original paper.
- Replaced with:
    - **Forward Dynamics via Articulated Body Algorithm (ABA)**
    - Angle limit constraints
    - Ground collision handling

---

## 🔌 Niagara Data Channel + PCG Integration

- Niagara emitters dynamically spawned based on PCG grid.
- ABA-based motion data (angular velocity, acceleration, displacement) calculated in Niagara.
- Emitters and particles cleaned up based on camera position.
- Custom workflow:
    - PCG writes to **Niagara Data Channel**
    - Niagara interprets & visualizes in real-time
    - Niagara emitters instances kill particles or kill itself based on camera position and direction.

> 📘 *Created a custom tutorial for using PCG with Niagara Data Channel Interface, since no existing guides were available.*
[PCG + Niagar Data Channel Tutorial Video](https://youtu.be/C1LmzQKNnzI)


---
## 🦾 Motion Comparison
### Motion At Strong Wind force

**Before Angular Limit and Collision**

![Strong Wind Motion Without Limit](./resources/no_limit.gif "Strong Wind Motion Without Limit")

[🔗 Watch on YouTube](https://youtu.be/sHjHLRHukEs)

[🔗 Watch on YouTube(start at the scene with wind force 40)](https://youtu.be/sHjHLRHukEs?si=raVWfqdE0HeZyLcM&t=68)


**After Angular Limit and Collision**

![Strong Wind Motion](./resources/after_limit.gif "Strong Wind Motion")

[🔗 Watch on YouTube (The Demo video starting at the scene with wind force 40)](https://youtu.be/5h7HZT5iuCI?si=dYmNk5WoUefEqJj9&t=36)

Without the total angular displacement limit and collision,
1. grass can store unlimited restoration force 
2. resulting motion inconsistent with wind movement


### Reference Study Motion

Motion result of the reference paper after applying several modifications including
- calculation error fix
- removal of additional inconsistent algorithms, and substitute those with
  - Angle limit constraints
  - Ground collision handling
- fixed grass length

[🔗 Watch on YouTube](https://youtu.be/qu_WTiCiIrc)

[![Watch the video](https://img.youtube.com/vi/qu_WTiCiIrc/hqdefault.jpg)]

The grasses tend to form a straight line with presence of strong wind force.

---

## 🧪 Procedural Variation with Voronoi
![Voronoi Diagram Example](./resources/voronoi_example.jpg "An example of Voronoi Diagram")

**Voronoi Diagram Example:** Locations sharing the same nearest points belong to the same region.

- PCG-generated Voronoi regions used to assign:
    - Grass length
    - Grass width
    - Stiffness
    - Initial direction
    - Color noise
    - population density
- Smooth transition between regions via linear interpolation on some characters.
  - population density
  - Grass length

![Grass Region Example](./resources/grass_voronoi_regions.jpg "Grass Region Example")
**Grass Voronoi Region Example:** The box meshes are located at Voronoi points. Three regions showing different colors, shapes and populations.

![Grass Region Interpolation Example](./resources/voronoi_region_linear_intp.jpg "Grass Region Interpolation Example")
**Region Linear Interpolation Example:** Gradual change of grass length from the region center(White box) to the region boundary can be observed.

---

## ⚔️ Comparison with *Ghost of Tsushima*

| Feature                         | Ghost of Tsushima    | My Implementation                         |
|---------------------------------|----------------------|-------------------------------------------|
| Bézier Curve Type               | Cubic (4-point)      | Quadratic (3-point)                       |
| Grass Length                    | Uhcotrolled (varies) | Fixed                                     |
| Motion                          | Unkown               | Physics Dynamics simulation based on  ABA |
| Hierarchical Runtime Generation | Custom Engine        | UE5 PCG + Niagara Data Channel Interface  |

---

## 📚 References

- [GDC Presentation – Procedural Grass in 'Ghost of Tsushima'](https://youtu.be/Ibe1JBF5i5Y?si=EbGqmGS29uNdBPUn)
- [SIGGRAPH Paper – Grass Swaying with Dynamic Wind Force](https://link.springer.com/article/10.1007/s00371-016-1263-7)
- [Unreal Engine Documentation – Niagara Data Channels Intro](https://dev.epicgames.com/community/learning/tutorials/RJbm/unreal-engine-niagara-data-channels-intro)

---


