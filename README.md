# Bianlian Animator
_Bianlian_ (变脸) is a set of animation player/maker for creating **dynamic, smooth, expressive robot faces**. 

This repository, _Bianlian Animator_, is a curve-based animation editor. You can find the animation player [here](https://github.com/shaunabanana/bianlian-player).

## Usage
First, Create a base face shape and export as SVG, preferably using Sketch. 

**IMPORTANT: The SVG must be flat, meaning no groups can be used. Every element must be directly under the root.**

An example:

<img src="https://github.com/shaunabanana/bianlian-animator/raw/master/assets/svg-example.png" width="400px" />

For every animation, choose Animation > New. Select the base face SVG.

Create keyframe animations as you would in AE.

Don't forget to save!

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
