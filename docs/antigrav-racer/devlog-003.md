# Devlog 003 – AI Movement Prototype

<video controls width="100%">
  <source src="videos/ai_no_collision_logic_race.mp4" type="video/mp4">
</video>

In this short video, simple AI agents are racing along the track, using the center line for guidance. The movement is updated in a local coordinate space $(l,w)$ rather than in world space. There is no collision logic yet, and if vehicles were to leave the track, they would simply become stuck. To prevent the later, they can strafe and will do so when the notice that the lateral parameter $w$, measuring the distance to the center line, is becoming to large.

## How do AI agents move

<img src="figures/track_tester.jpg" width="100%">

<div style="display: flex; gap: 10px;">
  <div style="text-align: center;">
    <img src="figures/straight_not_corrected.jpg" width="100%">
    <p><em>Before correction</em></p>
  </div>
  <div style="text-align: center;">
    <img src="figures/straight_corrected.jpg" width="100%">
    <p><em>After correction</em></p>
  </div>
</div>

```csharp
for (int i=0; i<N; i++)
{
    for (int j=0; j<substeps; j++)
    {
        float dti = dt / (float)substeps;
        (l1, w1, onTrack) = segment.UpdatePosition((l0, w0), (vl , vw), speed * dti);

        f = segment.Forward(l1, w1);
        r = segment.Right(l1, w1);
        u = segment.Up(l1, w1);

        wp = segment.WorldPosition(l1, w1);

        vl = Vector3.Dot(f, transform.forward);
        vw = Vector3.Dot(r, transform.forward);

        l0 = l1;
        w0 = w1;
    }
    // line renderer
    fwdLR.SetPosition(i+1, wp + 0.15f * u);
}
```

```csharp
public (float l1, float w1, bool onTrack) UpdatePosition((float, float) pos0, (float, float) dir, float dist)
{
    (float l0, float w0) = pos0;
    (float ml, float mw) = dir;

    float l1 = l0 + dist * ml;
    float w1 = w0 + dist * mw;

    float arc = ApproxCurveLength((l0, w0), (l1, w1), 1);
    float corr = arc / dist;
    corr = float.IsNaN(corr) ? 0f : corr;

    l1 = l0 + (l1 - l0) / corr;
    w1 = w0 + (w1 - w0) / corr;

    return (l1, w1, Mathf.Abs(w1) < 0.5f * Width(l1) && l1 <= length && l1 >= 0f);
}
```

