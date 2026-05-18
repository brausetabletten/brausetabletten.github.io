# Welcome

On this website, I will blog a bit about some of my hobby game dev projects.

The focus will be, for now, on a small anti-grav racer, that I am currently working on. 
I was playing F-Zero X and F-Zero GX with friends last summer and was fascinated by the physics of these games. This motivated me to try to implement a few things on my own. 

## Most Recent

- [Devlog 004 - Collision Handling Prototype](antigrav-racer/devlog-004.md)

<video controls width="100%">
  <source src="antigrav-racer/devlog-004/videos/ai_collision_race.mp4" type="video/mp4">
</video>

Building on the simple AI movement prototype, vehicle collisions were introduced to the game logic. Instead of relying on Unity's general purpose physics engine, I started to implement my own (more simplistic) physics logic. The vehicles are now equipped with a custom box-collider and a collision handler checks for overlap between these boxes in the LateUpdate cycle. Overlapping vehicles are separated by an iterative scheme. 