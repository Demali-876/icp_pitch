<script>
    import { onMount } from 'svelte';
  
    let canvas;
    let ctx;
    let center = [400, 250]; // Ensure this is used or adjusted appropriately
    let scene;
  
  
    const colors = [
      ["rgba(41,171,226,0.4)", 30, 20, Math.PI / 4],
      ["rgba(82,39,133,0.5)", 15, 10, Math.PI / 2],
      ["rgba(237,30,121,0.5)", 10, 8, Math.PI],
      ["rgba(241,90,36,0.8)", 10, 4, 1.5 * Math.PI],
      ["rgba(251,176,59,0.7)", 5, 3, 1.5 * Math.PI]
    ];
  
    function square(x) {
      return x * x;
    }
  
    function leminescate_of_bernoulli(t, a) {
      const x = (a * Math.sqrt(2) * Math.cos(t)) / (square(Math.sin(t)) + 1);
      const y = (a * Math.sqrt(2) * Math.cos(t) * Math.sin(t)) / (square(Math.sin(t)) + 1);
      return [x, y];
    }
  
    function rotate_point(pointX, pointY, originX, originY, angle, extrude) {
      let slope = Math.atan2(pointY - originY, pointX - originX);
      pointY = originY + Math.sin(slope) * extrude;
      pointX = originX + Math.cos(slope) * extrude;
      angle = angle * Math.PI / 180;
      return {
        x: Math.cos(angle) * (pointX - originX) - Math.sin(angle) * (pointY - originY) + originX,
        y: Math.sin(angle) * (pointX - originX) + Math.cos(angle) * (pointY - originY) + originY
      };
    }
  
    class InfinityLoop {
    constructor() {
      const pickedColor = colors[Math.round(Math.random() * (colors.length - 1))];
      this.a = Math.random() * 50 + 150;
      this.length = Math.random() * pickedColor[3] + (0.25 * Math.PI);
      this.position = Math.random() * 2 * Math.PI;
      this.speed = (2 * Math.PI / 100) - (Math.random() * (Math.PI / 200));
      this.heightAdjust = Math.random() * 0.2 + 1;
      this.center = {
        x: Math.random() * 20 - 10 + center[0],
        y: Math.random() * 20 - 10 + center[1],
      };
      this.color = pickedColor[0];
      this.thickness = Math.random() * pickedColor[1] + pickedColor[2];
    }
  
    computePath(start, length, a) {
      let main_points = [], extruded_left_points = [], extruded_right_points = [];
      const segments = Math.round(this.length / this.speed);
      let lastPoint = null;
  
      for (let i = 0; i < segments; i++) {
        let t = this.speed * i + this.position;
        if (t > 2 * Math.PI) t -= 2 * Math.PI;
        if (t < 0) t += 2 * Math.PI;
  
        let main_point = leminescate_of_bernoulli(t, a);
        main_point[1] = main_point[1] * this.heightAdjust + this.center.y;
        let newPoint = {
          x: main_point[0] + this.center.x,
          y: main_point[1]
        };
  
        if (!lastPoint) {
          let prevPoint = leminescate_of_bernoulli(t - this.speed, a);
          prevPoint[1] = prevPoint[1] * this.heightAdjust + this.center.y;
          lastPoint = { x: prevPoint[0] + this.center.x, y: prevPoint[1] };
        }
  
        let ribbonReductionIndex = (i / (segments / 200));
        let ribbonThickness = Math.sin(Math.PI/2 * (ribbonReductionIndex / 100)) * this.thickness;
  
        let extrudedLeftPoint = rotate_point(lastPoint.x, lastPoint.y, newPoint.x, newPoint.y, 90, -ribbonThickness / 2);
        extruded_left_points.push(extrudedLeftPoint);
  
        let extrudedRightPoint = rotate_point(lastPoint.x, lastPoint.y, newPoint.x, newPoint.y, 90, ribbonThickness / 2);
        extruded_right_points.push(extrudedRightPoint);
  
        lastPoint = newPoint;
      }
      return extruded_left_points.concat(extruded_right_points.reverse());
i    }
  
    drawPath() {
      let leafPath = this.computePath(this.position, this.length, this.a);
  
      this.position += this.speed;
      ctx.fillStyle = this.color;
      ctx.beginPath();
      ctx.moveTo(leafPath[0].x, leafPath[0].y);
      for (let i = 1; i < leafPath.length; i++) {
        ctx.lineTo(leafPath[i].x, leafPath[i].y);
      }
      ctx.closePath();
      ctx.fill();
  
      if (this.position > 2 * Math.PI) {
        this.position -= 2 * Math.PI;
      }
    }
  
    reposition() {
      this.center = {
        x: Math.random() * 20 - 10 + center[0],
        y: Math.random() * 20 - 10 + center[1]
      };
    }
  
    tick(timestamp) {
      this.drawPath();
    }
  }
  class Scene {
    constructor() {
      this.loops = [];
      this.lastTick = 0;
  
      for (let i = 0; i < 50; i++) {
        this.loops.push(new InfinityLoop());
      }
    }
  
    clearCanvas() {
      ctx.fillStyle = "rgba(255, 255, 220, 0.1)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
    }
  
    animate(timestamp) {
      if (timestamp - this.lastTick < 1000 / 60) {
        requestAnimationFrame(this.animate.bind(this));
        return;
      }
  
      this.clearCanvas();
      this.loops.forEach(loop => loop.tick(timestamp));
  
      this.lastTick = timestamp;
      requestAnimationFrame(this.animate.bind(this));
    }
  
    run() {
      requestAnimationFrame(this.animate.bind(this));
    }
  
    reset() {
      this.loops = [];
      this.lastTick = 0;
  
      for (let i = 0; i < 50; i++) {
        this.loops.push(new InfinityLoop());
      }
    }
  
    reposition() {
      this.loops.forEach(loop => loop.reposition());
    }
  }
  
    onMount(() => {
      ctx = canvas.getContext('2d');
      scene = new Scene();
      scene.run();
      resize();
  
      window.addEventListener('resize', resize);
      return () => {
        scene.destroy?.();
        window.removeEventListener('resize', resize);
      };
      
    });
  
    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      center[0] = canvas.width / 2;
      center[1] = canvas.height / 2;
      scene.reposition();
    }
    
  
    function reset() {
      scene.reset();
    }
  </script>
  <canvas bind:this={canvas} width="800" height="500"></canvas>
  <div class="metadata">
    <p>10 Reasons the Internet Computer Protocol will win</p>
    <p>
    Made by <a href="https://oajhk-xaaaa-aaaap-qca7a-cai.icp0.io/" target="_blank">Demali.icp</a>
    </p>
    <button class="opacity-0 bg-black/50 text-white hover:bg-white hover:text-black transition-colors text-6xl w-12 h-12 md:w-16 md:h-16 flex items-center justify-center rounded-lg md:rounded-2xl absolute left-4 md:left-8 top-1/2 -translate-y-1/2" id="previousButton" style="opacity: 1; pointer-events: auto;">
        <svg width="28" height="32" viewBox="0 0 28 32" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M20 8V24L8 16L20 8Z" fill="currentColor"></path>
        </svg>
      </button>
    <button class="pointer-events-auto bg-black/50 text-white hover:bg-white hover:text-black transition-colors text-6xl w-12 h-12 md:w-16 md:h-16 flex items-center justify-center rounded-lg md:rounded-2xl absolute right-4 md:right-8 top-1/2 -translate-y-1/2" id="nextButton" style="opacity: 1; pointer-events: auto;">
        <svg width="28" height="32" viewBox="0 0 28 32" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8 8L8 24L20 16L8 8Z" fill="currentColor"></path>
        </svg>
      </button>
  </div>
  <div class="title">Dfinity</div>
  

