# Copy of 2.2.8 Cycle 8 Wind

## Design

The aim of this cycle is to create a function which can create randomised terrain. The way I will do this is by generating random amplitudes for the terrain and using linear interpolation between the points to fill in the terrain. In the future I could possibly add a different method of interpolation which will appear much more smoother by using segments of a cosine wave.

### Objectives

* [x] Generate random points along the screen to connect terrain
* [x] Use linear interpolation to fill in the spaces between points
* [x] Render the terrain on the screen
* [x] Add collision to the terrain so bullets can't go through
* [x] Make sure the player gets added in on top of the terrain
* [x] Create a new terrain every time the level changes

### Key functions

| Function Name     | Use                                                                                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Terrain object    | For simplicity, all of the functions and variables to do with generating terrain have been placed into this object which is imported into the main code |
| seedTerrain       | Generates random points along the screen for the basic structure of the terrain                                                                         |
| tCollision        | Spawns in objects to add collision to the terrain as drawings don't have collision                                                                      |
| interpolateLinear | Linear interpolation between points of the terrain created by seedTerrain                                                                               |

### Pseudocode

```
OBJECT Terrain:
    points: [],
    rndRes: 192,
    variance: 750,
    minY: 100,
    seedTerrain: FUNCTION:
                    for every rndRes pixel in WIDTH:
                        points[pixel] = random between minY and minY + variance
    interpolateLinear: FUNCTION:
                            for every rndRes point in points:
                                current = points[point]
                                next = points[point + rndRes]
                                dy = next - current
                                for every pixel from point to point + rndRes:
                                    points[pixel] = current + (pixel - point) * dy / rndRes
    tCollision: FUNCTION:
                    for every 20th point on the terrain:
                        add a rectangle for collision
```

## Development

### Outcome

The outcome is a terrain which is randomly generated and rendered every time the game starts.

{% code title="main.ts" %}
```javascript
import kaboom from "kaboom"
import "kaboom/global"
import Terrain from "./terrain-generate.js"

export const WIDTH = 1920
export const HEIGHT = 1080
const GRAVITY = 450

kaboom({
  width: WIDTH,
  height: HEIGHT,
  background: [69, 65, 65]
})

/*
const recording = record()

onKeyDown(".", () => {
  recording.download({filename: "recording"})
})
*/

setGravity(GRAVITY)

loadBean()
loadPedit("barrel", "sprites/barrel.pedit")
loadPedit("bullet", "sprites/bullet.pedit")

function addBtn(txt, p, f) {
  // add a parent background object
  const btn = add([
    rect(240, 80, { radius: 8 }),
    pos(p),
    area(),
    scale(1),
    anchor("center"),
    outline(4),
  ])

  // add a child object that displays the text
  btn.add([
    text(txt),
    anchor("center"),
    color(0, 0, 0),
  ])

  // onHoverUpdate() comes from area() component
  // it runs every frame when the object is being hovered
  btn.onHoverUpdate(() => {
    btn.scale = vec2(1.2)
    setCursor("pointer")
  })

  // onHoverEnd() comes from area() component
  // it runs once when the object stopped being hovered
  btn.onHoverEnd(() => {
    btn.scale = vec2(1)
  })

  // onClick() comes from area() component
  // it runs once when the object is clicked
  btn.onClick(f)

  return btn

}

scene("practice", () => {

  onDraw(() => {
    Terrain.drawTerrain()
  })

  Terrain.seedTerrain()
  Terrain.interpolateLinear()
  Terrain.tCollision()
  
  onLoad(() => {
    add([
      pos(250, 250),
      text("Press \"esc\" to return to menu"),
      lifespan(2, { fade: 0.5 })
    ])
  })

  const bean = add([
    sprite("bean"),
    pos(80, HEIGHT - (Terrain.points[80] + 27)),
    rotate(0),
    anchor("center"),
    {
      SPEED_HIGH: 150,
      SPEED_LOW: 45,
      power: 0
    }
  ])

  const target = add([
    sprite("bean"),
    pos(1500, HEIGHT - (Terrain.points[1500] + 27)),
    area(),
    anchor("center")
  ])

  bean.add([
    sprite("barrel"),
    scale(3, 3),
    pos(0, -60)
  ])

  target.onCollide("bullet", () => {
    addKaboom(target.pos)
    destroy(target)
    destroyAll("bullet")
    go("practice")
  })

  onCollide("bullet", "ground", () => {
    destroyAll("bullet")
  })
  
  const newBullet = (position, angle) => {
    add([
      sprite("bullet"),
      pos(position),
      //offscreen({ destroy: true }),
      area(),
      anchor("center"),
      body(),
      rotate(angle),
      move(vec2(Math.cos(angle / 180 * Math.PI), Math.sin(angle / 180 * Math.PI)), (bean.power + 20) * 8.3),              //Calculating movement vector
      "bullet"
    ])
  }

  const angle = add([            //UI for displaying the angle of player
    text(Math.floor(bean.angle).toString()),
    pos(24, 24)
  ])

  const power = add([          //UI to display power of the shot
    text(Math.floor(bean.power).toString()),
    pos(100, 24)
  ])

  onKeyDown("space", () => {
    if (get("bullet").length == 0) {
      let endOfBarrel = vec2(bean.pos.x + 85 * Math.cos(-0.15 + bean.angle / 180 * Math.PI), bean.pos.y + 92 * Math.sin(-0.15 + bean.angle / 180 * Math.PI)) //Find the position of the end of the barrel to add bullets
      let bullet = newBullet(endOfBarrel, bean.angle)
    }
  })

  onUpdate(() => {
    angle.text = Math.abs(Math.floor(bean.angle)).toString()  //Update player angle UI
    power.text = Math.floor(bean.power).toString()

    if (get("bullet").length > 0) {
      if (get("bullet")[0].pos.x > WIDTH) {
        destroyAll("bullet")
      }
    }
  })

  let rotationSpeed = bean.SPEED_HIGH

  onKeyDown("alt", () => {
    rotationSpeed = bean.SPEED_LOW  //Slow rotation when holding alt
  })

  onKeyRelease("alt", () => {
    rotationSpeed = bean.SPEED_HIGH  //Normal when alt released
  })

  onKeyDown("left", () => {
    if (bean.angle > -70) {
      bean.angle += -rotationSpeed * dt() //Rotate left
    }
    else {
      bean.angle = -70  //Stop at 70 degrees anticlockwise
    }
  })

  onKeyDown("right", () => {
    if (bean.angle < 0) {
      bean.angle += rotationSpeed * dt() //Rotate right
    }
    else {
      bean.angle = 0  //Prevent tank rotating the wrong direction
    }
  })

  let smooth = 3;

  onKeyDown("up", () => {
    smooth += 1.2 * dt()
    if (bean.power < 100) {
      bean.power += 0.7 * smooth ** 2 * dt() //Increase power smoothly
    }
    else {
      bean.power = 100  //Maximum power 100
    }
  })

  onKeyRelease("up", () => {
    smooth = 3
  })

  onKeyDown("down", () => {
    smooth += 1.2 * dt()
    if (bean.power > 1) {
      bean.power -= 0.7 * smooth ** 2 * dt()  //Decrease power smoothly
    }
    else {
      bean.power = 0  //Minimum power 0
    }
  })

  onKeyRelease("down", () => {
    smooth = 3
  })
  
  onKeyDown("escape", () => {
    go("main-menu")
  })

})

scene("multiplayer-menu", () => {
  addBtn("Join Game", vec2(250, 150), () => { })
  addBtn("Create Game", vec2(250, 250), () => { })
  addBtn("Return", vec2(250, 350), () => { go("main-menu") })
})

scene("main-menu", () => {
  addBtn("Practice", vec2(250, 250), () => { go("practice") })
  addBtn("Multiplayer", vec2(250, 350), () => { go("multiplayer-menu") })
})

go("main-menu")
```
{% endcode %}

{% code title="terrain-generate.js" %}
```javascript
import {WIDTH, HEIGHT} from "./main.js"

const Terrain = {
  points: [],
  rndRes: 192,
  variance: 750,
  minY: 100,
  seedTerrain: function () {
    //generate random amplitudes every n pixels across width

    for (let w = 0; w <= WIDTH; w += this.rndRes){
      this.points[w] = Math.floor(Math.random() * this.variance + this.minY);
    }
  },
  interpolateLinear: function () {
    //use this.rndRes and difference between each point to interpolate

    for (let point = 0; point < this.points.length; point += this.rndRes) {
      let current = this.points[point];
      let next = this.points[point + this.rndRes];
      let dy = next - current;
      for (let p = point; p < point + this.rndRes; p++) {
        this.points[p] = (p - point) * dy / this.rndRes + current;
      }
    }
  },
  tCollision: function () {
    for (let point = 0; point < this.points.length; point += 20){
      add([
        pos(point, HEIGHT - this.points[point]),
        rect(8,35),
        area(),
        //color(255,255,0),
        anchor("top"),
        opacity(0),
        "ground"
      ])
    }
  }, 
  drawTerrain: function () {
    for (let point = 0; point < WIDTH; point += 3) {
      drawLine({
        p1: vec2(point + 2, HEIGHT),
        p2: vec2(point + 2, HEIGHT - Terrain.points[point]),
        width: 3,
        color: rgb(181, 4, 21)
      })
    }/*
    drawLine({
      p1: vec2(0, HEIGHT - this.minY),
      p2: vec2(WIDTH, HEIGHT - this.minY),
      width: 4,
      color: rgb(20, 10, 180)
    })
    drawLine({
      p1: vec2(0, HEIGHT - (this.minY + this.variance)),
      p2: vec2(WIDTH, HEIGHT - (this.minY + this.variance)),
      width: 4,
      color: rgb(20, 10, 180)
    })debugging to show the maximum and minimum possible terrain heights*/
  }

}

export default Terrain;
```
{% endcode %}

### Challenges

The only challenge for this cycle is figuring where to start, so I did some research on randomised terrain generating and different sound algorithms and eventually decided to do a simple method which consists of generating different amplitudes at set intervals across the screen and using linear interpolation between the points.&#x20;

## Testing

### Tests

| Test | Instructions     | What I expect                                                                    | What actually happens | Pass/Fail |
| ---- | ---------------- | -------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run code         | Kaboom should run                                                                | Kaboom ran normally   | Pass      |
| 2    | Enter "practice" | Terrain randomly generated and should be different each time the game is entered | Terrain works         | Pass      |

### Evidence

<div>

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>Some examples of randomly generated terrain</p></figcaption></figure>

 

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

</div>
