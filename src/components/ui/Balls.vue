<script lang='ts' setup>
import { onMounted } from 'vue'
import * as Matter  from 'matter-js';
import * as MatterWrap  from 'matter-wrap';

onMounted(() => {

  console.log('Matter', Matter)
  console.log('MatterWrap', MatterWrap)
  Matter.use(MatterWrap);
  // module aliases
  var Engine = Matter.Engine,
      Render = Matter.Render,
      Runner = Matter.Runner,
      Bodies = Matter.Bodies,
      MouseConstraint = Matter.MouseConstraint,
      Mouse = Matter.Mouse,
      Composite = Matter.Composite,
      Composites = Matter.Composites;
  
  // create an engine
  var engine = Engine.create();
  
  // create a renderer
  var render = Render.create({
      element: document.querySelector('#matter-canvas'),
      engine: engine,
      options: {
        width: window.innerWidth,
        height: window.innerHeight,
        wireframes: false, // Draw the shapes as solid colors
        background: "transparent",
        hasBounds: true,
      }
  });
  
  var stack = Composites.stack(100, 0, 10, 8, 10, 10, function(x, y) {
      return Bodies.circle(x, y, 30, { restitution: 0.6, friction: 0.1 });
  });
  var ground = Bodies.rectangle(window.innerWidth / 2, window.innerHeight, window.innerWidth * 2, 10, { isStatic: true });
  
  // add all of the bodies to the world
  Composite.add(engine.world, [stack, ground]);

  // wrapping using matter-wrap plugin
  var allBodies = Composite.allBodies(engine.world);

  for (var i = 0; i < allBodies.length; i += 1) {
    allBodies[i].plugin.wrap = {
      min: { x: render.bounds.min.x - 100, y: render.bounds.min.y },
      max: { x: render.bounds.max.x + 100, y: render.bounds.max.y }
    };
  }
  
  // run the renderer
  Render.run(render);
  
  // create runner
  var runner = Runner.create();
  
  // run the engine
  Runner.run(runner, engine);

  // add mouse control
  var mouse = Mouse.create(render.canvas),
      mouseConstraint = MouseConstraint.create(engine, {
          mouse: mouse,
          constraint: {
              stiffness: 0.2,
              render: {
                  visible: false
              }
          }
      });

  Composite.add(engine.world, mouseConstraint);

  // keep the mouse in sync with rendering
  render.mouse = mouse;
})
</script>

<template>
  <div class="balls-container">

    <div id="matter-canvas"></div>
  </div>
</template>


<style scoped>
#matter-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>