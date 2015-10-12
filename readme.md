# Tutorial: Phaser.io Splash Screen, Main Menu, and more with State Management

This aims to be an exhaustive tutorial and example/boilerplate with hopes of teaching others the usage of phaser's state management system, including:
* Creating a cool splash screen
* Loading assets into your game with a progress bar.
* How to load Custom Fonts into your game
* Game state management
* Navigation between Main Menu, Options Screen, and more.

## Chapter 1 - The Splash Screen

When finishing this chapter we'll create a cool splash screen with a preloading progress bar.  The progress bar
will also be available very early, as it will load all dependent scripts as well.  We'll also load and play music,
and load a custom font!  Additionally, we'll touch the surface of the game state management system.


## Part 1: Orientation

### Directory Structure

My directory structure follows a kind-of hybrid of scaffolds I've used in the past for web application development as well
as game development with different engines.  I am not saying it's the best one, but please follow it this time so you
can easily follow the tutorial.  If you get stuck at any time.  The source code is available to you on this github page.

```
game/
    assets/
        bgm/
        fonts/
        images/
        style
    states/
    vendor/
```

### The HTML file

For this HTML file we're going to do something a little different then you might have seen across the web.  Rather
then loading all of our assets in `script` tags, I want you to just load the minimum.  So it looks like we only need
`phaser.js` and `main.js` - Now if you are wondering why, and how we are going to load everything I look forward to
explaining further below.  Don't worry, we're not going to put everything in one monolithic file either :)

The [index.html](./game/index.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    <main id="game"></main>
    <script src="vendor/phaser.js"></script>
    <script src="main.js"></script>
</body>
</html>
```

The benefit is that we can actually have the engine start up much sooner.  This way, the users don't have to wait
as long before they can see your cool splash screen with a nice progress bar.  I don't know about you but, I have
closed my browser before fully loading a game before simply because I saw a blank screen.

So what do we need to load before we can show our cool splash screen??

* A background image - I chose space because I love space.
* A logo - I chose my personal logo because I made it a long time ago and I dont want to remake one
* The progress bar - I created a simple one you are free to use.  All you need is a horizontal bar and make it 100% full

### Exercise 1:

Create a simple logo, a progress bar, and a background image (800x600)

Or you can just use the stuff found in (./game/assets/images)


## Part 2 - Bootstrapping

So, did you create a logo, progress bar, and background image?  Or use mine? Either way, I hope you had fun!

So now it's time to create the splash screen, but before we get started with that we need to have phaser go ahead
and load our assets that we need to actually show it.

Now to create the main.js file that html file was pointing at, and we're going to use it to load everything we need
to show our cool splash screen, and then start the splash screen once it is ready.

game/main.js

```
var game = new Phaser.Game(800, 600, Phaser.AUTO, 'game'), main = {};

main.prototype = {
  // Our preload function loads our newly created images
  preload: function () {
    game.load.image('stars',    'assets/images/stars.jpg');
    game.load.image('loading',  'assets/images/loading.png');
    game.load.image('brand',    'assets/images/logo.png');
    // states/splash.js will be used to show our cool splash screen.
    game.load.script('splash',  'states/splash.js');
  },
  // phaser will run create function as soon as the preload function finishes loading all of the file assets.
  create: function () {
    // The game will create a new phaser object called Splash from the splash object.
    // Notice we didn't create a splash object?  no worries, the splash object will exist because we
    // will create it in states/splash.js - which always happens before the create function calls.
    game.state.add('Splash', splash);
    game.state.start('Splash');
  }

};

// Now to load our main script, and start!
game.state.add('Main', main);
game.state.start('Main');
```

### Preloading assets with progress

states/splash.js is used to load everything else, including javascript files and out webfont loader (thank you, google)

The benefit is the preload function will be used to load all of the assets and instantiate the progress bar.
Once the preload function completes, our "create" function follows.

To keep things organized, I prefer to create specific functions that have single responsibilities.

So here is how my splash object starts out:

Note, all we really need to do here, is load all of the game assets (at least for this tutorial) and show our
cool splash screen.
```javascript
splash.prototype = {

  loadScripts: function () {
  },

  loadBgm: function () {
  },

  loadImages: function () {
  },

  loadFonts: function () {
  },

  // The preload function then will call all of the previously defined functions:
  preload: function () {
    this.loadScripts();
    this.loadImages();
    this.loadFonts();
    this.loadBgm();
  },
```

#### Adding the progress bar

To add the progress bar, we add the following code to our preload function:

```javascript
    // Add the loadingbar to the scene:
    var loadingBar = game.add.sprite(game.world.centerX, 400, "loading");
    // set the anchor to 0.5, which anchors the center pixel to the stage, allowing centerX to align the sprite
    // to the center.  Otherwise the left-most pixel would be pinned to the centerX position, which is not what we want.
    loadingBar.anchor.setTo(0.5);
    // Tell phaser to use laodingBar as our preload progess bar
    this.load.setPreloadSprite(loadingBar);
```

Now with our preload progress bar in full effect, we will be able to let the game start up more quickly because the rest of
the game assets including our game scripts can be loaded and show the user a loading progress bar at the same time.

#### Designing the splash screen:

So all of the assets loaded from `main.js` are available to you to create your splash screen.  As such, we can add them
to our `preload` function:

```javascript
  preload: function () {
    // object z-index is set to when object was added.
    // background
    game.add.sprite(0, 0, 'stars');
    // my logo
    var myLogo = game.add.sprite(game.world.centerX-120, 100, 'brand');
    // loading bar
    var loadingBar = game.add.sprite(game.world.centerX-(387/2), 400, "loading");
    // my logo was too big, so i use scale to set it down, not optimal but it works for this.
    myLogo.scale.setTo(0.5);
    // add text that says its loading
    var status = game.add.text(game.world.centerX, 380, 'Loading...', {fill: 'white'});
    this.load.setPreloadSprite(loadingBar);
  }
```


That's nice and all, but we could do better.  In fact there are a couple things I do not like about it.

for one: I don't like that we are using game.add.sprite before we use var, because the javascript engine will then
create the var for me at the top of the function and it will be undefined.  This is known as hoisting.  I dont want
to get into too much detail about it.  But it might be better to do it this way:

```
  preload: function () {
    var myLogo, loadingBar, status;
    game.add.sprite(0, 0, 'stars');
    myLogo = game.add.sprite(game.world.centerX-120, 100, 'brand');
    myLogo.scale.setTo(0.5);
    // loading bar
    loadingBar = game.add.sprite(game.world.centerX-(387/2), 400, "loading");
    // my logo was too big, so i use scale to set it down, not optimal but it works for this.
    status = game.add.text(game.world.centerX, 380, 'Loading...', {fill: 'white'});
    this.load.setPreloadSprite(loadingBar);

```

Well, that's essentially the same thing, except we're saving the run-time compiler a step and we're reminding ourselves
how it works.  You don't have to do it this way, but it surely is a style that I prefer.

BUT.. there's another problem with this...  I don't have to use maths to center the logo, instead all I have to do
is anchor.setTo(0.5)

```
  preload: function () {
    var myLogo, loadingBar, status;
    game.add.sprite(0, 0, 'stars');

    myLogo = game.add.sprite(game.world.centerX, 100, 'brand');
    myLogo.anchor.setTo(0.5);
    myLogo.scale.setTo(0.5);

    status = game.add.text(game.world.centerX, 380, 'Loading...', {fill: 'white'});
    status.anchor.setTo(0.5);

    loadingBar = game.add.sprite(game.world.centerX, 400, "loading");
    loadingBar.anchor.setTo(0.5);
    this.load.setPreloadSprite(loadingBar);
```

Ok, that's looking better, but you know I still don't like it.  Also, DID you notice the loading bar loads from the center?
We'll have to avoid using anchor for the loading bar unless you would prefer to keep it that way of course.

Still, making multiple UI elements have the similar properties, like anchoring etc doesnt seem DRY.  I prefer
to be DRY (Don't Repeat Yourself), and really, I'd rather declare my variables at the top and have them defined as well.

So I endeed up creating some utility or helper functions and gave them the utils namespace.
Since you're following along, here's another challenge for you, if you get stuck just head on over to the next chapter:

#### Exercise 2:

1. create a utils.js file and put it in a new lib folder
2. add a helper function that will center all of the sprites you pass into it
3. make sure to load the utils.js file by putting it in main.js
4. use your new helper function with the your splash screen

## Part 3: DRY: Don't repeat yourself!

How did you do?  Here's what I came up with...

utils.js
```javascript
var utils = {
  centerGameObjects: function (objects) {
    objects.forEach(function (object) {
      object.anchor.setTo(0.5);
    })
  }
};
```

main.js (preload function)
```javascript
  preload: function () {
    game.load.image('stars',    'assets/images/stars.jpg');
    game.load.image('loading',  'assets/images/loading.png');
    game.load.image('brand',    'assets/images/logo.png');
    game.load.script('utils',   'lib/utils.js');
    game.load.script('splash',  'states/splash.js');
  },

```

My new splash screen (now using the handy init function!)


```
  init: function () {
    this.loadingBar = game.make.sprite(game.world.centerX-(387/2), 400, "loading");
    this.logo       = game.make.sprite(game.world.centerX, 200, 'brand');
    this.status     = game.make.text(game.world.centerX, 380, 'Loading...', {fill: 'white'});
    utils.centerGameObjects([this.logo, this.status]);
  },

  preload: function () {
    game.add.sprite(0, 0, 'stars');
    game.add.existing(this.logo).scale.setTo(0.5);
    game.add.existing(this.loadingBar);
    game.add.existing(this.status);
    this.load.setPreloadSprite(this.loadingBar);

    this.loadScripts();
    this.loadImages();
    this.loadFonts();
    this.loadBgm();

  },
```

The init function calls before preload, so we can go ahead and add our sprites there, in any order we want, without
having to worry about z-order, many thanks to the `make` function.

Then we can simply use `game.add` to add them to the stage.  You can notice, I also `chain` off of the addition
of the logo and scale it right then and there.

It's now time to add the scripts, images, fonts, and background game music.

## Part 4: Finishing up the splash screen/preloading assets.
```javascript
  loadScripts: function () {
    // You can download this at: https://github.com/typekit/webfontloader
    // or use npm install webfontloader
    game.load.script('WebFont', 'vendor/webfontloader.js');
    // You can go ahead and make empty versions of these:
    game.load.script('gamemenu','states/gamemenu.js');
    game.load.script('thegame', 'states/thegame.js');
    game.load.script('gameover','states/gameover.js');
    game.load.script('credits', 'states/credits.js');
    game.load.script('options', 'states/options.js');
  },

  loadBgm: function () {
    // thanks Kevin Macleod at http://incompetech.com/
    game.load.audio('dangerous', 'assets/bgm/Dangerous.mp3');
    game.load.audio('exit', 'assets/bgm/Exit the Premises.mp3');
  },
  // various freebies found from google image search
  loadImages: function () {
    game.load.image('menu-bg', 'assets/images/menu-bg.jpg');
    game.load.image('options-bg', 'assets/images/options-bg.jpg');
    game.load.image('gameover-bg', 'assets/images/gameover-bg.jpg');
  },

```
#### Loading a custom font

Phaser works pretty well with google's webfont loader.  What webfont loader does is it allows you to load fonts
with javascript code.  You can load fonts from google's website, or you can load custom fonts you created or downloaded
yourself.  I personally downloaded a custom font, in fact I think its better you keep the font on your filesystem,
in case a 3rd party server goes down.

To add the custom font:
1. Download a font from the web, a good site is http://www.dafont.com
2. Put the font in assets/fonts directory  (I chose theminion.otf)
3. Create a css file that defines the font

style/theminion.css:
```css
@font-face {
    font-family: 'TheMinion';
    src: url('../fonts/theminion.otf');
}
```

We will use the webfont loader script located at https://github.com/typekit/webfontloader to get the font.  This way
we dont put the font in our html file, this means that we can use our cool splash screen while the font is loading.

```javascript
  loadFonts: function () {
    WebFontConfig = {
      custom: {
        families: ['TheMinion'],
        urls: ['assets/style/theminion.css']
      }
    }
  },
```


Now let's create the create function, and have it change the text from "Loading" to "Ready" - then add a 5 second
timer before loading the next screen.

```
  create: function() {
    this.status.setText('Ready!');

    setTimeout(function () {
      // We will load the main menu here
    }, 5000);
  }
```

I decided to make it change the text to ready, so if the user is out of focus the game will not start.  THat way
they can click to continue, otherwise it will transition on its own.  Feel free to adjust the timer, or remove it
if you think it is too much of a delay.


We also want to enqueue the other states of our game like so:

```javascript

    game.state.add("GameMenu",GameMenu);
    game.state.add("Game",Game);
    game.state.add("GameOver",GameOver);
    game.state.add("Credits",Credits);
    game.state.add("Options",Options);

```


#### Playing music

You can play music with phaser's music loader pretty easy, like so:
```javascript
    music = game.add.audio('dangerous');
    music.loop = true;
    music.play();
```

But actually the music will keep playing between states, so it might be better to declare the `music` variable
at the top level of scope, so that way we can access it later in case we want to change it or if we have music
options that turn it off and on.

The final splash screen:
```javascript
var Splash = function () {},
    playSound = true,
    playMusic = true,
    music;

Splash.prototype = {

  loadScripts: function () {
    game.load.script('WebFont', 'vendor/webfontloader.js');
    game.load.script('gamemenu','states/gamemenu.js');
    game.load.script('thegame', 'states/thegame.js');
    game.load.script('gameover','states/gameover.js');
    game.load.script('credits', 'states/credits.js');
    game.load.script('options', 'states/options.js');
  },

  loadBgm: function () {
    // thanks Kevin Macleod at http://incompetech.com/
    game.load.audio('dangerous', 'assets/bgm/Dangerous.mp3');
    game.load.audio('exit', 'assets/bgm/Exit the Premises.mp3');
  },
  // varios freebies found from google image search
  loadImages: function () {
    game.load.image('menu-bg', 'assets/images/menu-bg.jpg');
    game.load.image('options-bg', 'assets/images/options-bg.jpg');
    game.load.image('gameover-bg', 'assets/images/gameover-bg.jpg');
  },

  loadFonts: function () {
    WebFontConfig = {
      custom: {
        families: ['TheMinion'],
        urls: ['assets/style/theminion.css']
      }
    }
  },

  init: function () {
    this.loadingBar = game.make.sprite(game.world.centerX-(387/2), 400, "loading");
    this.logo       = game.make.sprite(game.world.centerX, 200, 'brand');
    this.status     = game.make.text(game.world.centerX, 380, 'Loading...', {fill: 'white'});
    utils.centerGameObjects([this.logo, this.status]);
  },

  preload: function () {
    game.add.sprite(0, 0, 'stars');
    game.add.existing(this.logo).scale.setTo(0.5);
    game.add.existing(this.loadingBar);
    game.add.existing(this.status);
    this.load.setPreloadSprite(this.loadingBar);

    this.loadScripts();
    this.loadImages();
    this.loadFonts();
    this.loadBgm();

  },

  addGameStates: function () {

    game.state.add("GameMenu",GameMenu);
    game.state.add("Game",Game);
    game.state.add("GameOver",GameOver);
    game.state.add("Credits",Credits);
    game.state.add("Options",Options);
  },

  addGameMusic: function () {
    music = game.add.audio('dangerous');
    music.loop = true;
    music.play();
  },

  create: function() {
    this.status.setText('Ready!');
    this.addGameStates();
    this.addGameMusic();

    setTimeout(function () {
      //game.state.start("GameMenu");
    }, 5000);
  }
};
```

### Acknowledgements:
This system is based off of [Understanding Phaser States by Emanuele Feronato](http://www.emanueleferonato.com/2014/08/28/phaser-tutorial-understanding-phaser-states/) and also some other tutorials I've used in the past that were actually for different game engines.  The purpose is to have a re-usable game menu system.

Emanuele brings up some great details about state management that I will not re-iterate.
So if you're interested in learning more please also check out his tutorial as well.

I couldn't have done it without the examples found on phaser's website, you should check them out when you get the chance:  http://phaser.io/docs#loader