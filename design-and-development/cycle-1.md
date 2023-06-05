# 2.2.1 Cycle 1 Setting Up

## Design

For this cycle I want to get a repl with kaboom.js running and be able to render a simple sprite to get used to the environment I will be working in for the rest of this project. After I have set up the foundation I can work on my project online through replit.

### Objectives

* [x] Get a basic kaboom.js repl running
* [x] Render a sprite

### Usability Features

### Key Variables

| Variable Name | Use                                         |
| ------------- | ------------------------------------------- |
| height        | height in pixels of the game to be rendered |
| width         | width in pixels of the game to be rendered  |

### Pseudocode

```
import "kaboom"

const WIDTH = 1920
const HEIGHT = 1080

kaboom({
    width: WIDTH
    height: HEIGHT    
})

loadBean()

add(bean)
```

## Development

### Outcome

The outcome of this cycle is a stage set for development with kaboom.js within replit.

```
import kaboom from "kaboom"
import "kaboom/global"

const WIDTH = 1920
const HEIGHT = 1080

kaboom({
  width: WIDTH,
  height: HEIGHT
})

loadBean()

add([
  sprite("bean")
])
```

### Challenges

The only challenge I encountered is the times for the kaboom project to load being very long. This fixed itself, however as leaving it for a minute results in the project working

## Testing

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>Screenshot of kaboom running in replit</p></figcaption></figure>

### Tests

| Test | Instructions  | What I expect     | What actually happens | Pass/Fail |
| ---- | ------------- | ----------------- | --------------------- | --------- |
| 1    | Run code      | Kaboom should run | Kaboom ran normally   | Pass      |
| 2    | Press buttons | Something happens | As expected           | Pass      |

### Evidence
