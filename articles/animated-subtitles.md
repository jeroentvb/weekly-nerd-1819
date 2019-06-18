# Animating text based on audio amplitude with p5.js

Ever wanted to animate an html element bases on the amplitude of a sound that’s playing? No? I didn’t either, until I actually wanted to.
So here goes.

## Loading p5.js
P5.js is a library for drawing stuff on a canvas using JS, but it also has 2 extensions. One being p5.sound.js, which is what we are going to use.

First, download p5.js and p5.sound.js from [here](https://p5js.org/download/). Place them in your assets folder and add the following 2 lines to your html document.
```html
<script src="assets/js/p5.min.js"></script>
<script src="assets/js/p5.sound.min.js"></script>

<script src="assets/js/script.js"></script>
```

## Preload
P5 uses 2 main functions, which you have to define yourself. These functions being `setup` and `draw`.
In our case, before we can setup anything, we need to preload our sound file using a `preload`  function. P5 will run this function before the `setup`  function.

```js
let audio, amplitude

function preload () {
  audio = loadSound('assets/audio/cars.mp3')
}
```
This loads the audio for us. We make the `audio` variable a global one because we want to acces it in our setup function. We also create the `amplitude` variable for later use.

## Setup
Next we want to setup our canvas and amplitude.
```js
function setup () {
	const someEl = document.getElementById('someId')

  createCanvas(0, 0)
  amplitude = new p5.Amplitude()

  someEl.addEventListener('click', e => {
    audio.play()
  })
}
```
Because p5.js needs a canvas, but we don’t, we still create one but with 0 width and height.
Next we create a new instance of p5.Amplitude, which automatically listens to our previously loaded mp3 file.
Because the sound isn’t loaded via the html `audio`  tag, we don’t get any controls, so we need to manually bind them to an html element and listen for an event. This only sets up the `play` event, but you can add your own `pause`  eventListener if you want to.

## Draw
Finally, we need to create the `draw` function, because otherwise nothing will happen.
```js
function draw () {
  let level = amplitude.getLevel()

  document.getElementById('someId').style.fontSize = '${level}em'
}
```
We create a variable `level`  which get’s the level from the amplitude.
Then we get the element we want to animate, and change the css. In this example I’m changing the fontsize to the amplitude of the audio, but you can change and css style that uses a number as property.

## Final code
The final code looks like this:
```js
let audio, amplitude

function preload () {
  audio = loadSound('assets/audio/cars.mp3')
}

function setup () {
	const someEl = document.getElementById('someId')

  createCanvas(0, 0)
  amplitude = new p5.Amplitude()

  someEl.addEventListener('click', e => {
    audio.play()
  })
}

function draw () {
  let level = amplitude.getLevel()

  document.getElementById('someId').style.fontSize = '${level}em'
}
```