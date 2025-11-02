# Creating your very own Halloween kitty game!

## HTML basics

Creating web games requires knowing 3 computer languages:
- HTML: the language that web pages are written in
- CSS: the language that makes things look pretty on the screen
- JS (javascript): the language that complex logic is written in

We start out by writing a very simple web page

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Halloween Kitty! 🎃 </title>
    <script src="//cdn.jsdelivr.net/npm/phaser@3.11.0/dist/phaser.js"></script>
</head>
<body>
    <p>Welcome to the Halloween Kitty game!</p>
</body>
</html>
```

HTML uses `tags` to display different parts of the webpage. Each tag has a `<tagname>` tag that begins the code block and a `</tagname>` that ends the code block for that tag.

In the above code the `<head>` tag contains the header, or basic information about the page. The `<title>` tag is the title that is diplayed in the web browser. The `script` tag is used to load various javascript libraries. In this case we're loading the `Phaser` library that was created for making web based games!

The `<body>` tag contains the part of the web page that wil be displayed on the screen. In this case it contains a single paragraph `<p>` tag that prints the message `Welcome to the Halloween Kitty game!` on the screen.

There's a lot more to learn about HTML but we won't need much else for this tutorial so we'll move on to styling our web page.

### Veiwing the web page

To actually view the web page we need to first run a web server on our computer. Open up a new terminal and navigate to the directory that contains your `index.html` file. In that directory run `python -m http.server 8080`. This creates a local web server that you can access from your preferred web browser.

Open your web browser and in the address bar type `localhost:8080/index.html`. Congratulations, you've just created your first webpage!

## Using CSS to style web pages

Notice that this web page is *boring*. To make it more interesting we can change the styles of things.

Underneath the `<script>...<s/cript>` code block in your index file paste the code

```css
body {
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background-color: #1a1a2e;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
}
```

We won't go into the details here but this tells the webpage to set the background color, font color, and center the web page in the middle of the screen. Now refresh your web page in the browser and you'll notice that the background changed to a dark color with light colored font.

## Creating the game element

The game itself is going to be written in javascript. You can delete the `<p>Welcome to the Halloween Kitty game!</p>` line from the `<body>` of your code. In it's place, between the `<body>` and `</body>` tags past the following code

```html
<script type="text/javascript">
    // global game options
    let gameOptions = {
        screenWidth: 800,
        screenHeight: 600,
    }

    // Phaser game configuration
    let config = {
        type: Phaser.AUTO,
        width: gameOptions.screenWidth,
        height: gameOptions.screenHeight,
        physics: {
            default: 'arcade',
            arcade: {
                gravity: { y: 500 },
                debug: false
            }
        },
        scene: {
            preload: preload,
            create: create,
            update: update
        }
    };

    // create the Phaser game instance
    let game = new Phaser.Game(config);

    // Load all assets before the game starts
    function preload () {
    }

    // Create the game world
    function create () {
    }

    // Update the game state
    function update (time, delta) {
    }
</script>
```

Let's go through all of this new code. In javascript the `let` command just tells the compiler that you are defining a new `variable` (older code may use the `var` command but it's good practice now to use `let` for reasons that are outside of the scope of this tutorial). This might be new to you if you've used languages like `python` before that don't require you to define new variables, so
```javascript
let gameOptions = {
    screenWidth: 800,
    screenHeight: 600,
    gravity: {y: 500},
}
```
just defines a javascript object `gameOptions` that behaves like a mix between a python dict and a python class. Notice that unlike in python, you don't have to put quotes around `screenWidth` and `screenHeight`, since the compiler knows that those fields *must* be strings. This is different in python where the keys can be any immutable object. We also define the strength of `gravity` here. Modifying this value will make gravity stronger or weaker, changing the affect of gravity on objects in our game.

The next bit of code is for Phaser, the game engine that we're using to create our game

```javascript
let config = {
    type: Phaser.AUTO,
    width: gameOptions.screenWidth,
    height: gameOptions.screenHeight,
    physics: {
        default: 'arcade',
        arcade: {
            gravity: gameOptions.gravity,
            debug: false
        }
    },
    scene: {
        preload: preload,
        create: create,
        update: update
    }
};
```

As you can imagine `width` and `height` set the size of the playing field, `physics` defines the physics that are used. We won't go into details here but note that this is where we plugin our value for `gravity` that we defined in the game options.

The `scene` is the gameplay arena. We define 3 functions that are used to
- preload: Used to load sounds and graphics before the game begins
- create: Used to create a new game instance
- update: run every frame to update the objects used in the game

Right now those are empty functions but we'll be filling them in soon.

The next line `let game = new Phaser.Game(config);` is what actually creates the game that you are going to be writing. Notice that it is passed the `config` variable that you just defined. Refresh your web browser to see what the web page looks like now.

It's still the same!

## Adding a background

The first thing that we need to do is load the image into memmory. We do this by uplading the `preload` function

```javascript
function preload () {
    this.load.image('sky', 'assets/sky.png');
}
```

where `sky.png` is an actual image file. But the image still won't be displayed, we have to actually draw it in the game. We're going to need to create multiple objects in the game so lets make a new function underneath `update` that we can use to create all of our objects.

```javascript
function createObjects(scene){
    scene.add.image(400, 300, 'sky');

    // The platforms group contains the ground.
    // If we want to add more ground pieces later we
    // can use this same group
    scene.platforms = scene.physics.add.staticGroup();

    //  Here we create the ground.
    //  Scale it to fit the width of the game (the original sprite is 400x32 in size)
    scene.platforms.create(400, 568, 'ground').setScale(2).refreshBody();
}
```

Now update the `create` function to call our new `createObejcts` function, passing the scene so that new objects will be added to our Phaser game scene:

```javascript
function create () {
    createObjects(this);
}
```

This adds the image to the game at the location `(gameOptions.screenWidth/2, gameOptions.screenHeight/2)`. Notice that it is placed in the center of the screen. Any time images are added to our Phaser game they will be placed at the location of their center. Also notice that `sky` in the `add.image` function matches the first argument that we passed to `load.image`. This is required, as that keyword is what identifies the image that we loaded earlier and tells the game which image to use. Refresh your browser now to see the sky.

The last thing that we'll do is put a border around our game board to make it stand out a little more using CSS. So in your `<head><style>` section add

```css
canvas {
    border: 2px solid #ff6600;
    border-radius: 10px;
    box-shadow: 0 0 20px rgba(255, 102, 0, 0.3);
}
```

underneath your existing `body{...}` code. Refresh your browser now to see the outlined game board.

## Adding the ground

Now it's time to add the ground. For now we're going to just have a single "platform" but in the future, or other games, we might want to have multiple levels that the player can jump to. So we'll write our code so that it will be easy for us to add more than one platform for the user to stand on.

Like our background, we first need to load the image in the `preload` function. Underneath your code to load the sky write
```javascript
this.load.image('ground', 'assets/platform.png');
```

Because we might have more than one platform in the future we first create a group by adding this code to `createObjects`

```javascript
scene.platforms = scene.physics.add.staticGroup();
```

This is a `staticGroup`, meaning the platforms won't fall due to gravity or be affected by the other game pieces. Now we add a single `ground` platform just below that

```javascript
scene.platforms.create(400, 568, 'ground').setScale(2).refreshBody();
```

The `setScale` is used to double the ize of the platform. This makes it thicker, since the `platform.png` image is designed for thin platforms whereas for the main ground we want it to be thicker. Notice the `ground` argument that tells the code to use the `ground` image that we loaded in `preload`.

Refresh your browser to see the ground!

## Adding the player

Before we draw our player on the screen lets create some game options to tell us where it will be placed on the screen by adding `playerStartX: 100` and `playerStartY: 516` to `gameOptions`. Like the sky and the ground we first need to load the images for our player. In this case, just under your `preload` code to add the gournd add the slightly different
```javascript
this.load.spritesheet('cat-run', 'assets/RunCatb.png', { frameWidth: 32, frameHeight: 32 });
```
To see why look at the file `RunCatb.png`. It isn't just a single cat, it's 6 images that are all 32x32 pixels. By defining `frameWidth` and `frameHeight` to 32 it tells the code how to extract each individual frame of the cat by selecting a 32x32 box that contains it.

To create the player in the game we add the following code to the `createObjects` function beneath the code to create the platforms

```javascript
scene.player = scene.physics.add.sprite(
    gameOptions.playerStartX,
    gameOptions.playerStartY,
    'cat-run'
);
```

A `sprite` is a class-like structure used to contain all of the relevant information about an object in the game including it's position, image, health, etc. In this case we initialize it at the starting position that we defined above and with the first frame in the `cat-run` set of images that we created earlier. Refresh your browser to see your cat.

Oh no! The cat fell right off the bottom of the screen! What happened? We never told the game that the cat needs to interact with the ground!

We'll have to setup other interactions in our game so let's create a new function to handle all of those.

```javascript
function setCollisions(scene){
    scene.physics.add.collider(scene.player, scene.platforms);
}
```

Now we can just add `setCollisions(this);` to our `create` function just beneath our call to `createObjects`. Refresh now to see our kitty player resting peacefully on the ground.

## Adding animation

In our game the kitty is going to be running the entire time so we want to animate our cat to make it look like it's running. Remember how we saw that our `cat-run` was a set of images of a cat running? Now is where we animate between those different frames!

First we'll create a function to add all of our animations to

```javascript
function createAnimations(scene) {
    scene.anims.create({
        key: 'run',
        frames: this.anims.generateFrameNumbers('cat-run', { frames: [0,1,2,3,4,5,6,5,4,3,2,1] }),
        frameRate: 40,
        repeat: -1
    });
}
```

and then in `creat` we add `createAnimations(this);`. Make sure to add it before `createObejcts` since we also need to start the animation by adding

```javascript
scene.player.anims.play('run', true);
```
to our `createObjects` function.

Let's unpack this a bit. Notice that the key is `run`, which is how we'll identify this animation sequence in the future. The `frames` argument is for the frames in `cat-run` that are used for running (sometimes a single spritesheet might have more than one set of actions). In this case there are 7 frames, ranging from indices 0 through 6. But to fully animate our cat we also need to go in reverse for it to land and jump again, so we give it a `frames` keyword to show that we'll go through all of the frame forwards, then backwards. The frame rate just tells us how quickly to switch between frames. Making it larger will make the cat appear to run faster. Try changing the value a few times and refresh your browser to see for yourself.

## Adding user interactions

It's time to make our cat jump! Since the game already has gravity built in we just need to add a y-velocity to our cat when the user presses up or clicks the screen. Gravity will automatically slow the cat's y-velocity to a stop and then reverse the direction until it collides with the ground.

First let's add `jumpVelocity: -200` to our `gameOptions`. In our `create` function add the line

```javascript
cursors = this.input.keyboard.createCursorKeys();
```

which allows the game to capture key presses. Now let's add a few properties to the player in `createObjects` that will be useful for jumping

```javascript
scene.player.jumpVelocity = gameOptions.jumpVelocity;
```

Finally we'll modify the `update` function

```javascript
function update (time, delta) {
    let pointer = this.input.activePointer;
    let pressJump = pointer.isDown || cursors.up.isDown;

    if (pressJump && this.player.body.touching.down) {
        // start jump
        this.player.setVelocityY(this.player.jumpVelocity);
    }
}
```

The line `let pointer = this.input.activePointer;` just sets the variable `pointer` equal to the mouse or touchpad. The next line `let pressJump = pointer.isDown || cursors.up.isDown;` sets the value of the variable `pressJump` to `true` if the pointer is being held down or the `up` arrow is being held down.

The next code bloack says that if the user is pressing a jump key and their body is currently touching the ground below (`touching.down`) then it will set their y-velocity and their state to jumping. Refresh your browser and see that now you can jump by either pushing the mouse/touchpad or the up arrow.

### Fixing the jumping animation

Currently when the cat is jumping in the air it is still running. This isn't very realistic so we'll want to update our code to show the cat stretching out as it jumps through the air. To do that we first need to add this to the `createAnimations` function

```javascript
scene.anims.create({
    key: 'jump',
    frames: scene.anims.generateFrameNumbers('cat-run', { frames: [6] }),
    frameRate: 20,
    repeat: -1
});
```
This uses the same spritesheet that running uses but notice that we only use frame `6`, the image of the cat stretched out. In `createObjects` we add the line `scene.player.state = 'running';` so that the initial state of the player is running. Now we can add some logic to our `update` method to make it play the `jump` animation while jumping and return to `running` once the user lands on the ground

```javascript
if (pressJump && this.player.body.touching.down) {
    // start jump
    this.player.setVelocityY(this.player.jumpVelocity);
    this.player.anims.play('jump', true);
    this.player.state = 'jumping';
} else if (this.player.body.touching.down) {
    // landed
    this.player.anims.play('run', true);
    this.player.state = 'running';
}
```

## Adding prizes

Like many side based scrolling games we're going to have prizes that the user can collect, in this case a cool glowing pumpkin. Since we're going to have a collection of prizes that will all behave the same way we'll initialize them in the same way that we initialized the groud by creating a group in `createObjects`

```javascript
scene.prizes = scene.physics.add.group({
    allowGravity: false,
    immovable: true
});
```

Notice that we have gravity turned off for our prizes. This will allow them to stay at the same height that we place them instead of falling once the game starts.

Next we need to load the spritesheet for our pumpkins in `preload`

```javascript
this.load.spritesheet('pumpkin', 'assets/pumpkin.png', { frameWidth: 32, frameHeight: 32 });
```

Like the player (cat), the pumpkins are an animated sprite so we need to define the width and height of each frame. Now we can add the animation to `createAnimations`

```javascript
scene.anims.create({
    key: 'pumpkin-animation',
    frames: scene.anims.generateFrameNumbers('pumpkin', { start: 0, end: 16 }),
    frameRate: 8,
    repeat: -1
});
```

To create new prizes we add a new `addPrize` function that adds a prize at a given `x, y` position

```javascript
function addPrize(scene, x, y, name){
    // Add a prize to the prizes group
    let prize = scene.physics.add.sprite(x, y, name);
    // Don't prevent the prize from going out of the world bounds
    prize.setCollideWorldBounds(false);
    // Add the prize to the prizes group
    scene.prizes.add(prize);
    // Play the prize animation
    prize.play(name+'-animation');
    // Randomize the animation speed a bit
    prize.anims.setTimeScale(1 + Math.random() * 0.5);
}
```

Finally we add a single prize to the game in the `create` method with

```javascript
addPrize(this, 600, 475, 'pumpkin');
```

Refresh your browser to see a floating pumpkin

## Making the scene scroll

This part is surprisingly simple! First we'll create a new option in `gameOptions` for `scrollSpeed: 350`. Then we just add a line to the end of `addPrize` that will set the velocity of the prizes in the negative x-direction

```javascript
prize.setVelocityX(-gameOptions.scrollSpeed);
```
##