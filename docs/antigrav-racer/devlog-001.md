# Devlog 001 – Track Segment Conceptualization

There is a moment in F-Zero X, where the track suddenly ends and your vehicle drops onto a tube. Then, your vehicle sticks to this tube, and you can move around its curved surface area freely. This means you can drive on the downward facing surface, as if it was magnetic. At the same time, the speed of the vehicles stays at this insane level, and I kept wondering how this is achieved. I kept thinking of this, and decided to try out something:

My idea was that if I have a well-behaved parametrization of the track segments, the position of the vehicles on them could be updated in a sort-of analytical fashion i.e. the parametrization should allow for a movement scheme in which based on the current position and the movement direction, a closed-form expression is used to compute the next position. The vehicles should thus move on a 2D parameterized manifold in 3D space that defines the track, without relying on mesh information like vertex positions. The mesh would only be the rendered representation of the parametrized track. The movement shall be constrained to the track, as long as the vehicle is "on the track". As soon as a vehicle leaves the track (e.g. by sudden jumps or by going over an edge without a railing), the movement shall be determined from the last velocity on the track and the global gravity, with some steering allowed.  

## Key ideas:
- Vehicle's movement is confined on track segments of length $L$ and a variable width $W$
- While on a track segment, "down" is defined locally $\Rightarrow$ vehicles stick to the track
- A track segment shall be defined by its endpoints positions and rotations. 
- The track segments are parameterized by a longitudinal $l\in[0,L]$ and a transversal parameter $w\in[-W(l)/2,W(l)/2]$
- The movement is updated in the $(l,w)$ space while vehicles are on the track

## Initial ideas and conceptual problems
My core motivation was the mathematical problem of updating the vehicles positions. When I made the decision to build a prototype, I thought it would be possible to have an analytical solution for the movement on the track segments, if their parametrization is chosen properly. I aim to avoid barycentric coordinates, and while I am pretty certain that on segments defined by width, curve radius and global rotation an closed-form solution for movement without collisions exists, elements like this are most likely pretty restrictive in terms of geometry that can be achieved.

 I figured that a nice editor workflow would be to place waypoints, rotate them, and then have them connected by an automized track generation routine. From these waypoints, I want to use the position $\mathbf{X}_i$ and their forward direction $\mathbf{f}_i$, which gives me four vectors. Thus, the model for the track's center line needs 4 vectorial degrees of freedoms, which means two waypoints can be connected by the curve 


$$ \mathbf{X}(t)= \mathbf{X}_0 + t\mathbf{A} + t^2\mathbf{B} + t^3\mathbf{C}$$

so that $\mathbf{X}(0)$ and $\mathbf{X}(1)$ lie on the waypoints. Now, instead of using only the forward direction of the waypoints to match the first derivative, a scaling parameter can be introduced that allows to change the curvature of the track segments. The tangent line $\mathbf{T}(t) = \mathbf{X}'(t)$ shall then match the scaled forward directions $\mathbf{T}_i=\zeta\mathbf{f}_i$ at $t=0$ and $t=1$. Then, we end up with 

$$\mathbf{A}=\mathbf{T}_0$$

$$\mathbf{B}=3(\mathbf{X}_1-\mathbf{X}_0)-2\mathbf{T}_0-\mathbf{T}_1$$

$$\mathbf{C}=\mathbf{T}_0 + \mathbf{T}_1 - 2(\mathbf{X}_1-\mathbf{X}_0).$$


While this seemed nice enough, there is no obvious way how the right and up direction shall be interpolated between the two waypoints. The only real condition they need to satisfy is that they shall be orthogonal to the tangent line $\mathbf{T}(t)$. My initial idea was to use Unity's built-in ´Quaternion.Slerp´ to interpolate the rotations as quaternions between the points, to maintain a mathematical expression I can compute. But this does not guarantee the right and up directions to be orthogonal to the local forward direction defined by the center line's first derivative. The other way around, if I would compute the track segments such that their center line follows the forward obtained from ´Quaternion.Slerp´, the second waypoint cannot be placed freely (at least not easily, the system is over determined if not additional scaling is introduced). I decided that it is better to give up with the idea of having mathematical expressions for the up and right directions, and an analytical way to update the 3D movement on the track. Instead, I opted for interpolation from sampled values of up and right directions along the track. This means that I had to give up on the idea of having analytical expressions to update the vehicles position (although I soon realized that this idea wasn't all that well considered anyways).

This also means that there is no analytical expression for distances between two points defined by the local coordinates $(l,w)$. What surprised me a bit more though was that there seems to be no closed-form expression to transform between $l$, the position along the center line of the track segment measured in meters and the modelling parameter $t$. Since I had already given up on the idea of having an almost-analytical racing game, I decided that interpolation from lookup tables would do just fine.

## How I roughly expect (AI) vehicle control to work
First, if a vehicle is currently not on a track segment, it will try to find one using a raycast. Assuming that there is a segment underneath it, the local coordinates $(l,w)$ need to be determined. From them, the local forward and right direction of the track shall be computed. Thus, the track segments will implement `Vector3 Right(float l, float w)` and `Vector3 Forward(float l, float w)`. For orientation, the AI needs a guiding line on the segments, that can be the center line for now. From it, they can decide on a movement direction, that is projected on the local forward and right direction to update the parameter pair $(l,w)$ The vehicle is then moved to this new position, obtained by the `WorldPosition(float l, float w)` method of the track segment.

The new position might not be on the track segment anymore. If the segment has railing to prevent vehicles from overshooting, the movement needs to contain some bounce logic. Also, collisions with other vehicles need to be accounted for. 

## The Track Segment Interface

With these things considered, the interface for track segments I went with for now is

```csharp
public interface ITrackSegment
{
    float length { get; } // Approximated length of the center line
    float dl { get; } // Approximated sampling distance on the center line

    // Sampling
    Vector3 WorldPosition(float l, float w); // The position of the point represented by the parameters (l, w)
    Vector3 Right(float l, float w); // The right direction at (l, w)
    Vector3 Up(float l, float w); // The up direction at (l, w)
    Vector3 Forward(float l, float w); // The forward direction at (l, w)

    // Movement on the Segment
    (float l1, float w1, bool onTrack) UpdatePosition((float, float) pos0, (float, float) dir, float dist);

    // Arc Length Approximation between two points
    float ApproxCurveLength((float l, float w) start, (float l, float w) end, int steps = 1);

    // Get local coordinates from raycast hit
    (float l, float w) GetTrackWSCoordinates(RaycastHit hit);

    // Generate the mesh
    void Generate();
}
```