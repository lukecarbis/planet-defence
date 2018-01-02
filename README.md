# Planet Defence

[Open](https://lukecarbis.github.io/planet-defence)

Arrow keys to move the turret, space to shoot.

## To-do
- [ ] Add controller support for moving and firing the turret
- [ ] Add enemy spacecraft, flying toward the planet for you to shoot at
- [ ] Add collision between beam and enemies
- [ ] Explosions

## Debrief

This turned out to be much harder than I anticipated!

One of the first issues I ran into was rotating the turret. Because of the shape of the model, the rotation point was totally off, and there's not way that I can tell in A-Frame or three.js to fix this.

What I ended up doing was pretty neat, I created a box, and placed it at the rotation point that I wanted. Then I made the turret model a child of that box, and positioned it relative to it's parent. That way I can apply rotation to the box and the turret rotates from this point too.

There were lots of animations happening here. The turret needs to rotate, the beam grows in scale and position, the light surrounding the beam grows in intensity, and finally, the beam shoots off into the distance. I found that using A-Frame's `<a-animation>` was messy and unwieldy. In my [last](https://github.com/lukecarbis/drone-attack/) experiment, I found myself having to clean up the DOM once the animation had completed. Instead, I opted to use `TWEEN`, which is part of three.js, and hence part of A-Frame.

Another issue I ran into was positioning the beam. There are two states for the beam: loading and firing. When it's loading, it really needs to be a child of the turret, so that it can be positioned and animated in exactly the right place, and continue to move with the turret, before it's fired. However, after it's fired, it should not be linked to or follow the turret rotation in any way.

To solve this, I use two different beams. The loading beam is positioned as a child of the turret. When it's ready to fire, I need it's position and rotation, so I can apply that to the second "firing" beam. The problem here is that the "loading" beam's position is relative to it's parent.

To solve this, I was able to grab it's world position by creating a new `THREE.Vector3`, and use the `setFromMatrixPosition` method with the "loading" beam's `beam.object3D.matrixWorld` property. I then apply the world position to the "firing" beam, as well as the rotation of the turret.

Once the firing beam was in place, I had a lot of difficulty with the released beam actually firing. `TWEEN` uses the variables as they were set when _defining_ the tween, not as they are set when the tween starts. Even changing the value of a variable during a tween's `onStart` method won't have any effect on the value during `onUpdate`.

In the end I resolved this by calculating the position (end position and current position as a percentage between start and end) during the `onUpdate` method, which isn't an optimal use of resources, but the best I could manage.

The next major challenge I faced was figuring out the end point that I wanted the beam to fire to. It's no good just animating the beam's `position.z`, because this doesn't take into account rotation (`z` is always in the same place, no matter where the turret is pointing).

After looking into some complicated solutions (such as creating a new `Matrix4` with the turret's quaternion, and translating the `z` position of the matrix) I finally discovered three.js's very handy `translateZ` method, which did all the heavy lifting!
