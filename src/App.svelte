<script lang="ts">
  import * as PIXI from "pixi.js";
  import * as TWEEN from "@tweenjs/tween.js";
  import { onMount } from "svelte";
  import { randomInt } from "./utils/random";

  let gameContainer: any;

  let isSpinning = false;
  let isShowNotif = false;
  let notifMsg = "";

  let ticker = PIXI.Ticker.shared;
  const wheel = new PIXI.Container();
  wheel.eventMode = "static";
  wheel.cursor = "pointer";
  wheel.on("click", spinWheel);

  // initate spin tween
  let power = { speed: 0 };
  let tween = new TWEEN.Tween(power, false);

  const colors = [0xd3f8e2, 0xe4c1f9, 0xf694c1, 0xede7b1, 0xa9def9];

  let segments: { segment: PIXI.Graphics; label: string }[] = [];

  let pointer = new PIXI.Graphics();

  const debugRect = new PIXI.Graphics()
    .beginFill(0xffffff)
    .drawRect(0, 0, 20, 20)
    .endFill();

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

      segments.push({ segment, label: opts[i] });
    }
  }

  function showNotif(msg: string) {
    isShowNotif = true;
    notifMsg = msg;
    setTimeout(() => {
      isShowNotif = false;
      notifMsg = "";
    }, 2000);
  }

  function spinWheel() {
    isSpinning = true;
    power = { speed: 0 };
    const randSpeed = randomInt(10, 20);
    tween = new TWEEN.Tween(power, false)
      .to({ speed: randSpeed }, randSpeed * 100)
      .yoyo(true)
      .repeat(1)
      .easing(TWEEN.Easing.Quadratic.Out)
      .onComplete(() => {
        isSpinning = false;

        const collides = segments.filter((i) => isCollide(i.segment, pointer));
        let pointing = collides[0];
        if (collides.length > 1) {
          pointing = collides[1];
        }
        console.log("pointing", pointing?.label);
        showNotif("You got " + pointing?.label);

        // let bound = pointing?.segment.getBounds();
        // if (bound) {
        //   debugRect.x = bound.x;
        //   debugRect.y = bound.y;
        //   debugRect.width = bound.width;
        //   debugRect.height = bound.height;
        // }

        segments.forEach((i) => {
          let collide = isCollide(i.segment, pointer);
          console.log(collide, i.label);
        });
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
    pointer.x = app.screen.width / 2;
    pointer.y = app.screen.height / 2 - 300;
    app.stage.addChild(pointer);

    // app.stage.addChild(debugRect);

    animate(performance.now());
  }

  onMount(() => {
    let contentText = "0,10,20,30,40,0,200,120,60,20,0,90,60,30,10";
    // let contentText = "0,10,20,30";
    let options = contentText.split(",");
    genWheel(options);
    createApp();
  });
</script>

<div class="relative bg-zinc-900 w-screen h-screen">
  <div bind:this={gameContainer} class="w-screen h-screen" />

  {#if isShowNotif}
    <div
      class="absolute top-10 h-[100px] left-1/2 transform -translate-x-1/2 w-[400px] p-4"
    >
      <p class="text-center text-3xl font-bold text-white">{notifMsg}</p>
    </div>
  {/if}

  <div
    class="absolute bottom-10 h-[100px] left-1/2 transform -translate-x-1/2 w-[400px] p-4"
  >
    <button
      disabled={isSpinning}
      class="w-full h-full bg-blue-500 hover:bg-blue-600 disabled:bg-zinc-700 transition py-2 px-4 rounded-full text-3xl font-bold text-white"
      on:click={spinWheel}>SPIN</button
    >
  </div>

  <!-- <div class="absolute left-0 top-0 h-full bg-zinc-800 w-[400px] p-4"> -->
  <!--   <label for="content">Content</label> -->
  <!--   <textarea -->
  <!--     name="" -->
  <!--     id="content" -->
  <!--     class="p-2 rounded bg-zinc-700 w-full h-60" -->
  <!--     bind:value={contentText} -->
  <!--   /> -->
  <!-- </div> -->
</div>
