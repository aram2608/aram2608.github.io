---
title: Space Invaders in Go: Part 2
seo_title: How to Build Space Invaders in Go
seo_description: Previously, we got our window drawn, now we can get our ship ready to go!
date: 2025-11-06
permalink: /posts/2025/11/space-invaders-go2/
tags:
  - Programming
  - Game Dev
  - Go
---

A great man once said, if you want to learn a programming language, build a Space
Invaders knockoff, part 2.

Back Into It
===

In the last post, we got our screen drawn, which on its own is a terrible game.
In this post, we start to breath some life into our game by getting our ship drawn
and moving.

Loading our Assets
===

Now normally, a sprite map is used, where each sprite is spread out in a single PNG image.
You can then do some pixel math to subset out each individual sprite for the given
game object. In my case, all of my assets are divided into their own PNGs so the way
I go about this will be a bit different.

If you will remember, my file tree looks like this.

```bash
.
├── assets
│   ├── alien_1.png
│   ├── alien_2.png
│   ├── alien_3.png
│   ├── assets.go
│   └── spaceship.png
├── cmd
│   └── space-invaders
│       └── main.go
├── font
│   ├── font.go
│   └── monogram.ttf
├── go.mod
├── go.sum
└── md.md
```

In the `assets.go` file, we use the following code.

```go
package assets

import (
	"embed"
)

//go:embed *.png
var Embedded embed.FS
```

This packages up all our PNGss into a single embedded attribute that is publically
available.

In our main go file, we can now do the following.

```go
package main

import (
	"bytes"

	"image"
	"image/color"
	_ "image/png"

	"log"

	"space-invaders/assets"

	"github.com/hajimehoshi/ebiten/v2"
)

var shipImage *ebiten.Image
```

We forward declare the image so we can assign it later on. We also need to add
a couple more dependencies. The bytes package is used for reading PNGs and the 
image packages are used for rendering our asset into a suitable form for 
ebiten.Images. Because our go.mod file has created a module called
`space-invaders`, we need to load in our assets file using the
`space-invaders/assets` import.

We can now create an init method that assigns our variable at run time.

```go
func init() {
	shipFile, err := assets.Embedded.ReadFile("spaceship.png")
	if err != nil {
		log.Fatal(err)
	}

	img, _, err := image.Decode(bytes.NewReader(shipFile))

	if err != nil {
		log.Fatal(err)
	}

	shipImage = ebiten.NewImageFromImage(img)
}
```

We first read the embedded file and error check using the famed "if err != nil" that
litters most Go code bases.

We can then decode the image and error check one more time before creating the
*ebiten.Image needed for downstream rendering.

Drawing our Ship
===

To actually draw our ship, we need to create a method that we can plumb into
the Draw interface method employed by the Game struct.

Which is quite simple!

```go
func (g *Game) drawShip(screen *ebiten.Image) {
	op := &ebiten.DrawImageOptions{}
	op.GeoM.Translate(g.ship.posX, g.ship.posY)
	screen.DrawImage(shipImage, op)
}
```

The ebiten.DrawImageOptions struct is used to transform the image so that it can
be rendered properly. There are a whole slew of methods available and I
recommend checking out the docs if you have any fringe use cases in the future.

In this case, we simply translate the ship's position using the GeoM struct and
get our image drawn to the screen.

In our main Draw method, we can now do the following.

```go
func (g *Game) Draw(screen *ebiten.Image) {
	screen.Fill(color.RGBA{28, 3, 51, 255})
	g.drawShip(screen)
}
```

Creating our Ship Struct
===

The keen eyed should have noticed that there is now a new attribute in our Game
object called ship. And even if you didn't notice, your IDE surely did.

That object looks something like this.

```go
type Ship struct {
	posX  float64
	posY  float64
	speed float64
}

type Game struct {
	state State
	ship  *Ship
}
```

It is simply a data container that stores the positon on screen and the speed
we want our ship to travel at. We then store a pointer to it in our Game struct
and our ship is ready for drawing!

```go
func (g *Game) init() {
	g.state = GameOn
	g.ship = &Ship{
		posX:  float64((screenWidth - shipImage.Bounds().Dx()) / 2),
		posY:  float64(screenHeight - shipImage.Bounds().Dy() - 50),
		speed: 5,
	}
}
```

We calculate the middle of the screen using the predefined screen width and make
sure to subtract the ship's image size in pixels. This ensures proper placement given
how pixel rendering works in most game engines. We do the same for the height so
our ship is drawn slightly above the bottom-most edge of the window.

Movement!
===

Now that our ship is drawn, actually moving it is dead simple.

```go
func (g *Game) moveLeft() {
	g.ship.posX -= g.ship.speed
}

func (g *Game) moveRight() {
	g.ship.posX += g.ship.speed
}
```

We make two simple helper methods to modify the ships internal position. We can
then plop that straight into the Update method and our ship will be cruising along!

```go
func (g *Game) Update() error {
	if ebiten.IsKeyPressed(ebiten.KeyEscape) {
		return ebiten.Termination
	}

	if ebiten.IsKeyPressed(ebiten.KeyArrowLeft) {
		g.moveLeft()
	}

	if ebiten.IsKeyPressed(ebiten.KeyArrowRight) {
		g.moveRight()
	}
	return nil
}
```

We simply need to check key presses and update accordingly. With these simple
implementations, we now have a ship that can move side to side! Not the best game
ever, but a large improvement over a simple purple window to say the least.

```go
package main

import (
	"bytes"

	"image"
	"image/color"
	_ "image/png"

	"log"

	"space-invaders/assets"

	"github.com/hajimehoshi/ebiten/v2"
)

var shipImage *ebiten.Image

const (
	screenWidth  = 750
	screenHeight = 750
)

type State int

const (
	GameOn State = iota
	GameOver
)

func init() {
	shipFile, err := assets.Embedded.ReadFile("spaceship.png")
	if err != nil {
		log.Fatal(err)
	}

	img, _, err := image.Decode(bytes.NewReader(shipFile))

	if err != nil {
		log.Fatal(err)
	}

	shipImage = ebiten.NewImageFromImage(img)
}

type Ship struct {
	posX  float64
	posY  float64
	speed float64
}

type Game struct {
	state State
	ship  *Ship
}

func (g *Game) moveLeft() {
	g.ship.posX -= g.ship.speed
}

func (g *Game) moveRight() {
	g.ship.posX += g.ship.speed
}

func (g *Game) Update() error {
	if ebiten.IsKeyPressed(ebiten.KeyEscape) {
		return ebiten.Termination
	}

	if ebiten.IsKeyPressed(ebiten.KeyArrowLeft) {
		g.moveLeft()
	}

	if ebiten.IsKeyPressed(ebiten.KeyArrowRight) {
		g.moveRight()
	}
	return nil
}

func (g *Game) Draw(screen *ebiten.Image) {
	screen.Fill(color.RGBA{28, 3, 51, 255})
	g.drawShip(screen)
}

func (g *Game) drawShip(screen *ebiten.Image) {
	op := &ebiten.DrawImageOptions{}
	op.GeoM.Translate(g.ship.posX, g.ship.posY)
	screen.DrawImage(shipImage, op)
}

func (g *Game) Layout(outsideWidth, outsideHeight int) (screenWidth, screenHeight int) {
	return outsideWidth, outsideHeight
}

func (g *Game) init() {
	g.state = GameOn
	g.ship = &Ship{
		posX:  float64((screenWidth - shipImage.Bounds().Dx()) / 2),
		posY:  float64(screenHeight - shipImage.Bounds().Dy() - 50),
		speed: 5,
	}
}

func NewGame() ebiten.Game {
	g := &Game{}
	g.init()
	return g
}

func main() {
	ebiten.SetWindowSize(screenWidth, screenHeight)
	ebiten.SetWindowTitle("Go - Space Invaders")
	if err := ebiten.RunGame(NewGame()); err != nil {
		log.Fatal(err)
	}
}
```