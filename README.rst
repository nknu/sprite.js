===========
sprite.js
===========

The sprite.js framework lets you create animations and games
using sprites in an efficient way. The goal is to have common
framework for Desktop and mobile browsers.

sprite.js has been tested on Chromium, Firefox, Android emulator, Opera and IE9.

For an example of the what the framework offers, have a look at test_sprites.html.

You can see the tests demos online here:

http://batiste.dosimple.ch/sprite.js/tests/

Example usage
=================

Example of a basic use::

    <script src="sprite.js"></script>

    <script>
    // set the viewport size (default 480x320)
    sjs.w = 640;
    sjs.h = 480;

    // load the images that gonna be used in parallel. When all the images are
    // ready, the callback function is called.
    sjs.loadImages(['character.png'], function() {

        // create the Sprite object;
        var sp = new sjs.Sprite('character.png');

        // change the visible size of the sprite
        sp.h=55;
        sp.w=30;
        // apply the latest visual changes to the sprite;
        sp.update();

        // change the offset of the image in the sprite (this works the opposite way of a CSS background)
        sp.xoffset=50;
        sp.yoffset=50;

        // diverse transformations
        sp.move(100, 100);
        sp.rotate(3.14 / 4);
        sp.scale(2);
        sp.opacity = 0.8;

        sp.update();

    });
    </script>



Performance and different ways to draw
=======================================

This library provides 2 rendering backends: HTML and canvas.

By default the HTML backend is used. The HTML backend displays sprites using DOM elements when the canvas
backend draw the sprites on the canvas. Each layer of the application can have a different backend.
This enable you to mix the 2 technics if needed.

Switching sprites.js to use the canvas backend is done by setting the 'useCanvas' flag before
creating sprites::

    sjs.useCanvas = true;

You can also add a canvas GET parameter in the URL of any sprite.js application::

    index.html?canvas=1

Finaly, you can always overrides those defaults by using the useCanvas options when you create a layer.

    var background = Layer('background', {useCanvas:true});

Tests show that the canvas backend can be somewhat slower on Firefox, Opera and Chrome.
Especially with a high number of sprites and a large canvas. Clearing and redrawing the whole canvas can expensive if you have a lot of sprites.
Canvas seems faster when there is a lot of transformations applied to the sprite.

The canvas will be automaticaly cleared by the game ticker. If you don't need it you can set the autoClear to false when building a layer::

    var background = Layer('background', {useCanvas:true, autoClear:false});

Performances on particle test can be quite different depending on the device and browser and plateform:

+------------------------+---------------+-------------+---------------+---------------------+-------+
| Browsers               | Chrome linux  | Opera linux | Firefox linux | HTC Desire (webkit) | IE9   |
+========================+===============+=============+===============+=====================+=======+
| HTML backend           | 2000          | 60          | 500           | 120                 | 30    |
+------------------------+---------------+-------------+---------------+---------------------+-------+
| Canvas backend         | 1300          | 100         | 300           | 80                  | 600   |
+------------------------+---------------+-------------+---------------+---------------------+-------+



Sprite object public methods and attributes
===========================================



To create a sprite you should use the sjs.Sprite constructor helper::

    var sprite = sjs.Sprite(<src>, <layer>)

Both parameters are optionnal. If you want to set the layer but not any image::

    var sprite = sjs.Sprite(false, <layer>)

For performance reasons *there have been an API change* on the way the attributes can be set, please read the following.
Sprites have those *read only* attributes::

    sprite.y
    sprite.x
    sprite.w        // Controls the visible surface of the image. To have repeating sprites
                    // set the width or height value bigger than the size of the image.
    sprite.h

    sprite.xoffset  // offset in the image to start painting in the view surface
    sprite.yoffset
    sprite.xscale
    sprite.yscale
    sprite.angle    // use radians
    sprite.opacity  // use float in the range 0-1
    sprite.color    // Background color of the sprite. Use the rgb/hexadecimal CSS notation.

If you want to change any of those attributes use the following setters::

    sprite.setX(5);
    sprite.setY(5);
    sprite.setXOffset(10) // offset in the image to start painting in the view surface
    sprite.setXScale(2)

Or one of the helper methods::

    sprite.rotate(radians)
    sprite.scale(x, y)   // if y is not defined, y take the same value as x
    sprite.move(x, y)
    sprite.offset(x, y)
    sprite.size(w, h)    // set the width and height of the visible sprite

To appy handle simple physic with the sprites you can use those helpers::

    sprite.xv                // horizontal velocity
    sprite.yv                // vertical velocity
    sprite.rv                // radial velocity
    sprite.applyVelocity()   // apply all velocities on the current Sprite
    sprite.reverseVelocity() // apply all the negative velocities on the current Sprite

    sprite.applyXVelocity()    // apply the horizontal xv velocity
    sprite.applyYVelocity()    // apply the vertical yv velocity
    sprite.reverseXVelocity()  // apply the horizontal xv velocity negatively
    sprite.reverseYVelocity()  // apply the vertical yv velocity negatively

    sprite.isPointIn(x, y) // return true if the point (x, y) is within
                           // the sprite surface (angles don't affect this function)

    sprite.collidesWith(sprite) // return true if the sprite is in
                                // collision with the other sprite (angles don't affect this function).

    sprite.collidesWithArray([sprites]) // Same as collidesWith but you need to pass an array of sprite.
                                        // The current sprite is filtered out of the test loop.

    sprite.distance(x, y)       // return the distance between the sprite center and the point (x, y)

Other important methods::

    sprite.onload(callback)     // call the function "callback" when the sprite's image is loaded.
                                // If the image is already loaded the function is called immediatly.


    sprite.loadImg(src, bool resetSize)    // change the image sprite. The size of the sprite will be rested by
                                           // the new image if resetSize is true.

    sprite.remove // Remove the dom element if the HTML backend is used and facilite the garbage collection of the object.


    Sprite.canvasUpdate(layer)  // draw the sprite on a given layer, even if the sprite's layer use a HTML backend


To update the view after modifying the sprite, call "update"::

    Sprite.update()

With a canvas backend, the surface will be automaticaly cleared before each game tick. You will need to call update
to draw the sprite on the canvas again. If you don't want to do this you can set the layer autoClear attribute to false.


Ticker object
==============

Keeping track of time in javascript is tricky. Sprite.js provides a Ticker object to deal with
this issue.

A ticker is an object that keeps track of time properly, so it's straight
forward to render the changes in the scene. The ticker gives accurate ticks.
A game tick is the time between every Sprites/Physics update in your engine.
To setup a ticker::

    function paint() {

        myCycles.next(ticker.lastTicksElapsed);
        // do your stuff

    }
    var ticker = new sjs.Ticker(35, paint); // we want a tick every 35ms
    ticker.run();

    ticker.pause();
    ticker.resume();

lastTicksElapsed is the number of ticks elapsed during 2 runs of the paint
function. If performances are good the value should be 1. If the number
is higher than 1, it means that there have been more game ticks than calls
to the paint function since the last time paint was called. In essence,
there were dropped frames. The game loop can use the tick count to make
sure it's physics end up in the right state, regardless of what has been
rendered.

Cycle object
============

A cycle object handles sprite animations. A cycle is defined by list of
tuples: (x offset, y offset, game tick duration), and the sprites the
cycle applies to. this is a cycle with 3 position, each lasting 5 game ticks::

    var cycle = new sjs.Cycle([[0, 2, 5],
                              [30, 2, 5],
                              [60, 2, 5]);
    var sprite = sjs.Sprite("walk.png")
    cycle.sprites = [sprite];

    cycle.next()  // apply the next cycle to the sprite
    cycle.next(2) // apply the second next cycle to the sprite
    cycle.goto(1) // go to the second cycle triplet
    cycle.reset() // reset the cycle to the original position
    cycle.repeat = false // if set to false, the animation will stop automaticaly after one run


Input object
=============

The input object deals with user input. There are a number of flags for keys
that will be true if the key is pressed::

    var input  = new sjs.Input();
    if(input.keyboard.right) {
        sprite.move(5, 0);
    }

    // arrows is true if any directionnal keyboard arrows are pressed
    if(input.arrows())
        cycle.next();
    else
        cycle.reset();

    // input.keyboard is a memory of which key is down and up. If you need to know which key
    // has just been pressed or released you can use those functions

    input.keyPressed('up')
    input.keyReleased('up')

Layer object
=============

If you need to separate you sprites into logical layers, you can use the Layer
object::

    var background = new sjs.Layer('background', options);

You should then pass the layer as the second argument of the contructor of your sprites::

    var sprite = new sjs.Sprite('bg.png', background);

The layer object can take those options::

    var options = {
        useCanvas:true,   // force the use of the canvas on this layer, that enable you to mix HTML and canvas
        autoClear:false   // disable the automatic clearing of the canvas before every paint call.
    }

