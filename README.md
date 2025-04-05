# 🎮 Grass Motion with Multi-Body Dynamics in Unreal Engine 5

> A physically-based grass animation system inspired by *Ghost of Tsushima* and implemented using Unreal Engine 5, Niagara, PCG, and multi-body dynamics.

## 📽️ Demo Video
[🔗 Watch on YouTube](https://youtu.be/5h7HZT5iuCI?si=WpGUy6z84sb_mj0Y)

![Demo Screenshot](https://img.youtube.com/vi/5h7HZT5iuCI/hqdefault.jpg)

## 🔗 Source Code
[GitHub Repository](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics) - includes README file with algorithmic details.

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

<!-- 예시 이미지: 베지어 커브 비교 -->
<!-- ![Bezier Curve Comparison](link_to_image.jpg) -->

### 💨 Wind & Motion Simulation
- Wind simulated via noise functions.
- Skeleton-based articulation:
    - P₀: Ball joint (3 DOF)
    - P₁: Hinge joint (1 DOF)
- Elastic recovery toward initial pose, with randomized stiffness.

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

> 📘 *Created a custom tutorial for using PCG with Niagara Data Channel interface, since no existing guides were available.*

<!-- 튜토리얼 영상 위치 -->
<!-- ![Niagara Tutorial Video](link_to_video.jpg) -->

---

## 🧪 Procedural Variation with Voronoi

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

<!-- 예시 이미지: Voronoi Diagram -->
<!-- ![Voronoi Regions](link_to_image.jpg) -->

---

## ⚔️ Comparison with *Ghost of Tsushima*

| Feature                         | Ghost of Tsushima | My Implementation                         |
|---------------------------------|-------------------|-------------------------------------------|
| Bézier Curve Type               | Cubic (4-point)   | Quadratic (3-point)                       |
| Grass Length                    | Dynamic (varies)  | Fixed                                     |
| Motion                          | Unkown            | Physics Dynamics simulation based on  ABA |
| Hierarchical Runtime Generation | Custom Engine     | UE5 PCG + Niagara Data Channel Interface  |

---

## 📚 References

- [GDC Talk – Procedural Grass in 'Ghost of Tsushima'](https://youtu.be/Ibe1JBF5i5Y?si=EbGqmGS29uNdBPUn)
- [SIGGRAPH Paper – Grass Swaying with Dynamic Wind Force](https://link.springer.com/article/10.1007/s00371-016-1263-7)
- [Unreal Engine Documentation – Niagara Data Channels Intro](https://dev.epicgames.com/community/learning/tutorials/RJbm/unreal-engine-niagara-data-channels-intro)

---


