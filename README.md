# Bezier-Noise
Bezier Noise is a new type of noise algorithm I made, to explain shorly how it works, connect a bunch of bezier curves together, and every bezier curve is connected by Anchor points and control points with random heights.


##How it works?
The way the algorithm works is pretty simple, first, just create points, you can space them horizontally however you want, but at first i spaced them evenly so each point has en equal distance from the previous point and the next point. Second step is to just apply a bezier curve for every 3 points, a bezier curve starts from the last point of the previous bezier curve, except for the first one, then you have a control point and the last point, all of which have random heights. If you ha done it like this, there would be most likely alot of sharp turns, to fix this we need the control point of every curve to have the same slope as the line that the last point of the previous bezier curve and the control point of the previous bezier curve makes. After this change we can draw bezier curves and there won't be any sharp turns.
