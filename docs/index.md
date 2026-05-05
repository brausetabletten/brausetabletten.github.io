# Welcome

On this website, I will blog a bit about some of my hobby game dev projects.

The focus will be, for now, on a small anti-grav racer, that I am currently working on. 
I was playing F-Zero X and F-Zero GX with friends last summer and was fascinated by the physics of these games. This motivated me to try to implement a few things on my own. 

## Most Recent

- [Devlog 003 - AI Movement Prototype](antigrav-racer/devlog-003.md)

<video controls width="100%">
  <source src="antigrav-racer/devlog-003/videos/ai_no_collision_logic_race.mp4" type="video/mp4">
</video>

In this short video, simple AI agents are racing along the track, using the center line for guidance. The movement is updated in local coordinate space $(l,w)$ rather than in world space. There is no collision logic yet, and if vehicles were to leave the track, they would simply become stuck. To prevent the latter, they can strafe and will do so when they notice that the lateral parameter $w$, measuring the distance to the center line, is becoming too large.