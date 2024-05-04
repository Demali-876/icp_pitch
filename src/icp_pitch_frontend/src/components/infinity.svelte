<script>
  import { onMount } from 'svelte';
  import { fly } from 'svelte/transition';
  let canvas;
    let ctx;
    let center = [400, 250]; 
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
    }
  
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
      ctx.fillStyle = "rgba(55, 22, 66, 0.1)";
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
  let timeoutId;
  let isVisible = true; 
  let currentSlide = -1;
  const totalSlides = 10;
  const slides = Array.from({ length: totalSlides }, (_, i) => `/slide${i + 1}.png`);
  
  function nextSlide() {
      if (currentSlide < totalSlides - 1) {
          currentSlide++;
      }
      resetTimer();
  }

  function prevSlide() {
    if (currentSlide > 0) {
      currentSlide--;
    } else if (currentSlide === 0) {
      currentSlide = -1; 
    }
    resetTimer();
  }


  function resetTimer() {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
          isVisible = false;
      }, 3000); 
  }
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
  onMount(() => {
      ctx = canvas.getContext('2d');
      scene = new Scene();
      scene.run();
      resetTimer();
      resize();
      window.addEventListener('resize', resize);
      window.addEventListener('mousemove', () => {
          isVisible = true; 
          resetTimer();
      });
      window.addEventListener('keydown', () => {
          isVisible = true; 
          resetTimer();
      });
      return () => {
          window.removeEventListener('resize', resize);
          window.removeEventListener('mousemove', resetTimer);
          window.removeEventListener('keydown', resetTimer);
          clearTimeout(timeoutId);
      };
  });
</script>


<div class="landing-page">
  <canvas bind:this={canvas} width="900px" height="1440px"></canvas>
  <div class="metadata">
    <div class="slide-container" style="--current-slide: {currentSlide}">
      {#each slides as slide, index}
      {#if index === currentSlide}
      <div class="slide" in:fly={{ x: 200, duration: 500, delay: 0 }} out:fly={{ x: -400, duration: 500 }}>
          <img src={slide} alt={`Slide ${index + 1}`}>
        </div>
      {/if}
    {/each}    
      </div>
      <button class="button previousButton" on:click={prevSlide} style="opacity: {currentSlide > -1 && isVisible ? '1' : '0'};">
        <svg width="28" height="32" viewBox="0 0 28 32" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M20 8V24L8 16L20 8Z" fill="currentColor"></path>
      </svg>
      </button>
      <button class="button nextButton" on:click={nextSlide} style="opacity: {currentSlide < totalSlides - 1 && isVisible ? '1' : '0'};">
        <svg width="28" height="32" viewBox="0 0 28 32" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8 8L8 24L20 16L8 8Z" fill="currentColor"></path>
      </svg>
      </button>
      <div class="text-white font-semibold text-sm md:text-xl bg-black/50 px-4 py-2 rounded-md" id="slideNumber">Slide 1/10</div>
      <div class="text text_center">
          <h4 class="title">10 reasons why ICP will win | Developer Edition</h4>
      </div>
      <div class="text-white/60 italic px-4 py-3 hover:text-black hover:bg-white hover:shadow-lg landscape:hidden text-center">
        Rotate your device for better experience
      </div>
      <p>Made by <a href="https://oajhk-xaaaa-aaaap-qca7a-cai.icp0.io/" target="_blank">Demali.icp</a></p>
  </div>
</div>

