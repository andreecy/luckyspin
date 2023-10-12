<script lang="ts">
  import * as PIXI from "pixi.js";
  import * as TWEEN from "@tweenjs/tween.js";
  import { onMount } from "svelte";
  import { randomInt } from "./utils/random";

  let gameContainer: any;

  let ticker = PIXI.Ticker.shared;
  const wheel = new PIXI.Container();
  wheel.eventMode = "static";
  wheel.cursor = "pointer";
  wheel.on("click", spinWheel);

  // initate spin tween
  let power = { speed: 0 };
  let tween = new TWEEN.Tween(power, false);

  const colors = [0xd3f8e2, 0xe4c1f9, 0xf694c1, 0xede7b1, 0xa9def9];

  let segments: PIXI.Graphics[] = [];

  // generate wheel
  function genWheel(opts: string[]) {
    const count = opts.length;
    const radius = 300;
    const fullCircleAngle = Math.PI * 2;
    const segmentAngle = fullCircleAngle / count;

    segments = [];

    for (let i = 0; i < count; i++) {
      const segment = new PIXI.Graphics();
      // segment.lineStyle(2, 0x000000, 1);
      const color = colors[i % colors.length];
      segment.beginFill(color, 1);
      segment.lineTo(0, 0);
      segment.arc(0, 0, radius, 0, segmentAngle);
      segment.lineTo(0, 0);
      segment.endFill();
      segment.rotation = segmentAngle * i;
      wheel.addChild(segment);

      const style = new PIXI.TextStyle({
        fontFamily: "Arial",
        fontSize: 24,
        fontWeight: "bold",
        fill: ["#ffffff", "#00ff99"], // gradient
        stroke: "#4a1850",
        strokeThickness: 5,
        lineJoin: "round",
      });

      const text = new PIXI.Text(opts[i], style);
      text.anchor.set(0.5);
      text.x = radius / 2;

      const labelContainer = new PIXI.Container();
      labelContainer.rotation = segmentAngle * i + segmentAngle / 2;
      labelContainer.addChild(text);

      wheel.addChild(labelContainer);

      segments.push(segment);
    }
  }

  function spinWheel() {
    power = { speed: 0 };
    const randSpeed = randomInt(20, 100);
    tween = new TWEEN.Tween(power, false)
      .to({ speed: randSpeed }, randSpeed * 100)
      .yoyo(true)
      .repeat(1)
      .easing(TWEEN.Easing.Quadratic.Out)
      .onComplete(() => {
        console.log(wheel.rotation);

        let pointing = segments.find((s) => isCollide(s, pointer));
        console.log("pointing: ", pointing);
      })
      .stop()
      .start();
  }

  // Test For collision hit
  function isCollide(object1: PIXI.DisplayObject, object2: PIXI.DisplayObject) {
    const bounds1 = object1.getBounds();
    const bounds2 = object2.getBounds();

    return (
      bounds1.x < bounds2.x + bounds2.width &&
      bounds1.x + bounds1.width > bounds2.x &&
      bounds1.y < bounds2.y + bounds2.height &&
      bounds1.y + bounds1.height > bounds2.y
    );
  }

  const pointer = new PIXI.Graphics();

  // game loop update
  function animate(time: number) {
    ticker.update(time);
    tween.update(time);
    wheel.rotation += 0.01 * power.speed;

    requestAnimationFrame(animate);
  }

  function createApp() {
    gameContainer.innerHTML = ""; // clear children

    const app = new PIXI.Application({
      background: "#222",
      backgroundAlpha: 0,
      resizeTo: gameContainer,
    });
    gameContainer.appendChild(app.view);

    ticker.autoStart = false;
    ticker.stop();

    app.stage.addChild(wheel);
    wheel.x = app.screen.width / 2;
    wheel.y = app.screen.height / 2;

    pointer.beginFill(0xde3249);
    pointer.drawRect(-10, -10, 20, 20);
    pointer.endFill();
    pointer.x = app.screen.width / 2 + 300;
    pointer.y = app.screen.height / 2;
    app.stage.addChild(pointer);

    animate(performance.now());
  }

  onMount(() => {
    contentText = "Eat\nSleep\nBath\nRetry";
    createApp();
  });

  let contentText = "";
  $: options = contentText.split("\n");
  $: genWheel(options);
</script>

<div class="relative bg-zinc-900 w-screen h-screen">
  <div bind:this={gameContainer} class="w-screen h-screen" />

  <div class="absolute left-0 top-0 h-full bg-zinc-800 w-[400px] p-4">
    <label for="content">Content</label>
    <textarea
      name=""
      id="content"
      class="p-2 rounded bg-zinc-700 w-full h-60"
      bind:value={contentText}
    />
  </div>
</div>
