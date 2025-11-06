---
title: 'Space Invaders in Go: Part 1'
seo_title: 'How to Build Space Invaders in Go'
seo_description: 'Every programmer starts somewhere, with Space Invaders being a great place to learn the basics.'
date: 2025-11-04
permalink: /posts/2025/11/space-invaders-go1/
tags:
  - Programming
  - Game Dev
  - Go
---

A great man once said, if you want to learn a programming language, build a Space
Invaders knockoff.

Go
===

Go is a high level, compiled programming language. It was developed at 
Google by Robert Griesemer, Rob Pyke, and Ken Thompson. I would argue 
Pyke and Thompson deserve entire books, let alone a simple blog post. 
Prior to working for Google, they were both employed at Bell Labs, basically 
forging the programming landscape we have all come to know and sometimes love. 
Griesemer has quite a few achievments under his belt as well, having worked on 
the V8 JavaScript engine and a couple other programming language design 
projects. 

Go itself is known for being dead simple. Compared to C++ and its 81 keywords 
in the C++20 standard, Go only has 25.

```go
break     default      func    interface  select
case      defer        go      map        struct
chan      else         goto    package    switch
const     fallthrough  if      range      type
continue  for          import  return     var
```

It has a C-like syntax that leans toward simplicity, garbage collected 
memory management, a property-based type system, and safe concurrency features. 
Similar to Rust, the Go build tool hosts an entire tool chain, that is capable of
managing dependencies, formatting code, and initializing Go modules. It could be
described as multi-paradigm, with concurrent, functional, imperative, and even
some object-oriented features.

With all that said, it's a pretty neat language. It is primarily used for networking,
CLIs, and backend stuff in web development, so making a game in it isn't entirely
orthodox. However, there is always some brave soul willing to make bindings for
the graphics frameworks needed, so here we go!

Getting Started
---

I am using the [ebiten](https://github.com/hajimehoshi/ebiten) framework for this
game, as it has a pretty simple API, and I'm tired of using Raylib since it's a bit
bulky.

The way the API works is that a Game object needs to implement all the methods
in the ebiten.Game interface. 
These methods are,
  * Update()
  * Draw()
  * Layout()

Finally, in the `main` func, we initialize window parameters and start our 
game loop.

```go
type Game struct

func (g *Game) Update() {
  // Update logic
}

func (g *Game) Draw() {
  // Drawing logic
}

func (g *Game) Layout() {
  // Game layout logic
}

func main() {
	ebiten.SetWindowSize(screenWidth, screenHeight)
	ebiten.SetWindowTitle("Go - Space Invaders")
	if err := ebiten.RunGame(&Game); err != nil {
		log.Fatal(err)
	}
}
```

Before we get started, it's important to get your layout all squared away
to save yourself from import headaches later on.

```bash
.
├── assets
│   ├── alien_1.png
│   ├── alien_2.png
│   ├── alien_3.png
│   ├── assets.go
│   ├── mystery.png
│   └── spaceship.png
├── cmd
│   └── space-invaders
│       └── main.go
├── font
│   ├── font.go
│   └── monogram.ttf
├── go.mod
├── go.sum
├── LICENSE
└── README.md
```

My file tree looks something like this. With the root dir `.` named whatever you want.

The go.mod file looks something like this.

```bash
module space-invaders

go 1.24.2

require github.com/hajimehoshi/ebiten/v2 v2.9.3
```

It'll change as we add deps and include more imports, but for now this should
hopefully get you started. And if you're unsure, a `go mod tidy` usually fixes
most problems.

The Game
---

A word of caution, as I have mentioned I am learning Go, so this post likely 
does not follow best practices, so treat this code with the degree of 
scrutiny it deserves.

With that said we can ge started with our Game object.

```go
type Game struct {
  state State
}
```

Fairly bare bones for now, but we first need to get a window drawn before we can
really get to making more changes. The goal is iterative development, not to copy, 
paste, and pray for the best. My many experiences vibe-coding have taught me that
this saves a world of heart break and annoyance.

The only data we need for now is whether our game is on or not so we store an
enum, which looks like the following.

```go
type State int
const (
  GameOn State = iota
  GameOver
)
```

Quite simple, if you are unsure what that code is doing, it's basically a C-style
enum. You store a constant int and give it a fancy name. It's const so you don't
change it on accident, and its fancily named so you can easily keep track of what
it's doing.

With that out of the way, we can start implementing our interface methods so our
game is treated like an ebiten.Game object.

```go
func (g *Game) Update() error {
  if ebiten.IsKeyPressed(ebiten.KeyEscape) {
    return ebiten.Termination
  }
  return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
  // Dark purple background, change however you please
  screen.Fill(color.RGBA{28, 3, 51, 255})
}

func (g *Game) Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int) {
	return outsideWidth, outsideHeight
}
```

Super simply for now, that screenWidth and screenHeight are global parameters we
set to make our lives easier.

```go
const(
  screenWidth = 750
  screenHeight = 750
)
```

The final piece of the puzzle is to make an init method for our Game object as
well as a method that can create a new Game instance on the startup of our game.

```go
func (g *Game) init() {
  g.state = GameOn
}

func NewGame() ebiten.Game {
  g := &Game{}
  g.init()
  return g
}
```

Now, we can plumb that into our main func and hopefully get ourselves a screen drawn.

```go
func main() {
	ebiten.SetWindowSize(screenWidth, screenHeight)
	ebiten.SetWindowTitle("Go - Space Invaders")
	if err := ebiten.RunGame(NewGame()); err != nil { // <- goes in here!
		log.Fatal(err)
	}
}
```

So, if you have your imports sqaured away this should create a 750 x 750 pixel window
that can close after an `Escape` button press.

If you are unsure what to include, it should look something like this so far.

```go
import (
	"image/color"
	"log"

	"github.com/hajimehoshi/ebiten/v2"
)
```

That is all for now, in the next post, we will create our ship and get it to
move around a bit.

In sum...

```go
package main

import (

	"image/color"
	"log"

  "github.com/hajimehoshi/ebiten/v2"
)

const(
  screenWidth = 750
  screenHeight = 750
)

type State int
const (
  GameOn State = iota
  GameOver
)

type Game struct {
  state State
}

func (g *Game) Update() error {
  if ebiten.IsKeyPressed(ebiten.KeyEscape) {
    return ebiten.Termination
  }
  return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
  screen.Fill(color.RGBA{28, 3, 51, 255})
}

func (g *Game) Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int) {
	return outsideWidth, outsideHeight
}

func (g *Game) init() {
  g.state = GameOn
}

func NewGame() ebiten.Game {
  g := &Game{}
  g.init()
  return g
}

func main() {
	ebiten.SetWindowSize(screenWidth, screenHeight)
	ebiten.SetWindowTitle("Go - Space Invaders")
	if err := ebiten.RunGame(NewGame()); err != nil { // <- goes in here!
		log.Fatal(err)
	}
}
```