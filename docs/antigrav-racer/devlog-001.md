# Devlog 001 – Track Segment Prototype

### Key ideas:
- Vehicle's' movement is confined on track segments
- While on a track segment, "down" is defined locally => vehicles stick to the track
- The first type of track segments is parametrized by a longitudinal and a transversal parameter

```csharp
private (Vector3 A, Vector3 B, Vector3 C) GetCubicHermiteCoefficients(Vector3 X0, Vector3 X1, Vector3 T0, Vector3 T1)
{
    Vector3 A = T0;
    Vector3 B = 3 * (X1 - X0) - 2 * T0 - T1;
    Vector3 C = T0 + T1 - 2 * (X1 - X0);

    return (A, B, C);
}

public Vector3 CenterLine(float t)
{
    // The center line is given by c(t)=X1+t*A+t**2*B+t**3*C, t in [0,1]
    // with A, B, C so that c(1)=X2, c'(0)=T0, c'(1)=T1

    // This is equivalent to the Cubic Hermite spline

    var tt = t * t;
    var ttt = tt * t;

    return X0 + t*A + tt*B + ttt*C;
}

public Vector3 TangentLine(float t)
{
    // The derivative of the center line
    var tt = t * t;

    return A + 2*t*B + 3*tt*C;
}
```

