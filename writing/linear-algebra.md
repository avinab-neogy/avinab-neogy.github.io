---
layout: default
title: linear-algebra-primer
description: beginner-friendly deep dive into linear algebra with interactive illustrations
---

### why linear algebra?

linear algebra is the study of **linear maps**: functions that preserve addition and scaling.  
- if you double the input → the output doubles  
- if you add two inputs → the outputs add  

in math terms, for a linear map \( f \):

$$
\color{#2062b8}{f(\mathbf{v}_1 + \mathbf{v}_2) = f(\mathbf{v}_1) + f(\mathbf{v}_2)}
$$


$$
\color{#2062b8}{f(c \mathbf{v}) = c f(\mathbf{v}) \quad \text{for any number } c}
$$

these rules might sound abstract, but they actually describe real operations that show up everywhere: stretching, rotating, shearing or flipping space. you'll find them in physics (forces), graphics (rotations, scaling) and at the heart of every neural network layer.

---

### seeing linear maps in action: transformation matrices

in two dimensions, a 2×2 matrix is the most direct way to describe a linear map. 

the interactive demo below lets you see this in action:
- use the sliders to change how each point on the grid moves
- each slider controls one entry in a 2×2 matrix
- as you adjust the matrix, you create a new linear map

<div class="matrix-label" style="font-family:monospace; font-size:1.1rem; margin-bottom:0.8rem;">
  matrix:<br>
  <span id="label-matrix">⎡ 1  0 ⎤<br>⎣ 0  1 ⎦</span>
</div>
<div class="grid-container" style="display:flex; flex-direction:column; align-items:center;">
  <canvas id="grid-canvas" class="grid-canvas" width="500" height="300"></canvas>
  <div class="grid-controls" style="margin-top:20px;">
    <div style="display:flex; gap:32px; justify-content:center; margin-bottom:8px;">
      <label>a₁₁: <input type="range" id="m11" min="-3" max="3" step="0.1" value="1"></label>
      <label>a₁₂: <input type="range" id="m12" min="-3" max="3" step="0.1" value="0"></label>
    </div>
    <div style="display:flex; gap:32px; justify-content:center;">
      <label>a₂₁: <input type="range" id="m21" min="-3" max="3" step="0.1" value="0"></label>
      <label>a₂₂: <input type="range" id="m22" min="-3" max="3" step="0.1" value="1"></label>
    </div>
  </div>
</div>


<p style="max-width:500px; margin-top:1.2rem;">
<b>what do the sliders do?</b><br>
- <b>a₁₁:</b> controls stretching/compression or flipping along the <b>x</b> direction.<br>
- <b>a₁₂:</b> mixes the <b>y</b> value into the new <b>x</b> (shearing/rotation).<br>
- <b>a₂₁:</b> mixes the <b>x</b> value into the new <b>y</b> (shearing/rotation).<br>
- <b>a₂₂:</b> controls stretching/compression or flipping along the <b>y</b> direction.<br>
<br>
these four sliders make a 2×2 matrix:<br>
<span style="font-family:monospace;">
[ a₁₁  a₁₂ ]<br>
[ a₂₁  a₂₂ ]<br>
</span>
<br>
this matrix transforms every point (x, y) to (a₁₁x + a₁₂y, a₂₁x + a₂₂y).<br>
experiment with the sliders to see stretches, shears, rotations, and reflections!
</p>

<script>
  let sliders;
  document.addEventListener("DOMContentLoaded", () => {
    const canvas = document.getElementById('grid-canvas');
    const ctx = canvas.getContext('2d');
    sliders = ['m11','m12','m21','m22'].map(id => document.getElementById(id));
    function draw() {
      const a11 = +sliders[0].value,
            a12 = +sliders[1].value,
            a21 = +sliders[2].value,
            a22 = +sliders[3].value;
      document.getElementById('label-matrix').innerHTML =
        `⎡ ${a11.toFixed(2)}  ${a12.toFixed(2)} ⎤<br>⎣ ${a21.toFixed(2)}  ${a22.toFixed(2)} ⎦`;
      const w = canvas.width, h = canvas.height;
      ctx.clearRect(0,0,w,h);
      // grid lines
      const step = 20;
      ctx.strokeStyle = '#eee';
      for (let x=-w; x<=w; x+=step) {
        ctx.beginPath();
        for (let y=-h; y<=h; y+=step) {
          const X =  w/2 + a11*x + a12*y;
          const Y =  h/2 + a21*x + a22*y;
          if (y===-h) ctx.moveTo(X,Y); else ctx.lineTo(X,Y);
        } ctx.stroke();
      }
      for (let y=-h; y<=h; y+=step) {
        ctx.beginPath();
        for (let x=-w; x<=w; x+=step) {
          const X =  w/2 + a11*x + a12*y;
          const Y =  h/2 + a21*x + a22*y;
          if (x===-w) ctx.moveTo(X,Y); else ctx.lineTo(X,Y);
        } ctx.stroke();
      }
      // axes
      // x-axis (red)
      ctx.strokeStyle = '#d00';
      ctx.beginPath();
      ctx.moveTo(w/2 + a11*(-w), h/2 + a21*(-w));
      ctx.lineTo(w/2 + a11*(w),  h/2 + a21*(w));
      ctx.stroke();
      // y-axis (blue)
      ctx.strokeStyle = '#00a';
      ctx.beginPath();
      ctx.moveTo(w/2 + a12*(-h), h/2 + a22*(-h));
      ctx.lineTo(w/2 + a12*(h),  h/2 + a22*(h));
      ctx.stroke();
      // axis labels
      ctx.fillStyle = "#d00";
      ctx.font = "16px monospace";
      ctx.fillText("x'", w/2 + a11*110, h/2 + a21*110);
      ctx.fillStyle = "#00a";
      ctx.fillText("y'", w/2 + a12*110, h/2 + a22*110);
    }
    sliders.forEach(s => s.addEventListener('input', draw));
    draw();
  });
</script>

*the effect you see on the grid is exactly what happens when your transformation matrix ie. a **linear map** is applied to every vector on the plane.*
<span style="color:blue;">
the power of linear maps: no matter how the grid morphs, straight lines remain straight and the origin never moves.
</span>

---

### kernel, rank and degrees of freedom

Every linear map 
$$
\color{#2062b8}{ f : \mathbb{R}^n \to \mathbb{R}^m }
$$ can be represented by a matrix 

$$
\color{#2062b8}{ A \in \mathbb{R}^{m \times n} .}
$$
The **kernel (null space)** of <span style="color:#2062b8;">\( f \)</span> is the set of all input vectors sent to zero when the linear map is applied:
<div style="color:#2062b8;">
\[
\ker(f) = \{\, \mathbf{x} \in \mathbb{R}^n \mid f(\mathbf{x}) = \mathbf{0} \,\}
\]
</div>
This kernel consists of all solutions to the homogeneous system:
<div style="color:#2062b8;">
\[
A \mathbf{x} = \mathbf{0}
\]
</div>

The **rank** of <span style="color:#2062b8;">\( f \)</span> is the dimension of its image (the set of outputs you can reach by applying <span style="color:#2062b8;">\( f \)</span>):
<div style="color:#2062b8;">
\[

\operatorname{rank}(f) = \dim(\operatorname{Im}(f))

\]
</div>

The dimension of the kernel tells us **how many degrees of freedom remain** after applying the constraints, so they are like free variables which can vary independently and are  defined by <span style="color:#2062b8;">\( f \)</span>:
<div style="color:#2062b8;">
\[
\dim(\ker(f)) = n - \operatorname{rank}(f)
\]
</div>
If all constraints are independent (<span style="color:#2062b8;">
$$
\operatorname{rank}(f) = m 
$$</span>), then
<div style="color:#2062b8;">
\[
\text{degrees of freedom} = n - m
\]
</div>

**geometric intuition:**
- <span style="color:#2062b8;">\( n \)</span>: number of variables = initial degrees of freedom  
- <span style="color:#2062b8;">\( m \)</span>: number of independent equations = constraints reducing freedom

the interactive demo shows how a 2×2 matrix acts as a <span style="color:#2062b8;">linear map</span>. 
when you use a matrix for a linear map, each constraint cuts down your degrees of freedom. the kernel is the set of directions(yes think vectors) that get collapsed to zero by those constraints. the rank tells you how many constraints are actually independent.

as you add more constraints, you lose freedom. fewer constraints means more freedom. your degrees of freedom are the directions left after the linear map acts.


<style>
  .dof-flex { display: flex; gap: 2rem; align-items: flex-start; }
  .dof-canvas { border: 1px solid #ddd; }
  .dof-controls { display: grid; row-gap: 0.5rem; }
</style>

<div class="dof-flex">
  <canvas id="dof-canvas" class="dof-canvas" width="380" height="300"></canvas>
  <div class="dof-controls">
    <label>variables (n): 
      <input type="range" min="1" max="3" value="2" id="var_n" step="1">
      <span id="label_n">2</span>
    </label>
    <label>constraints (m): 
      <input type="range" min="0" max="3" value="0" id="con_m" step="1">
      <span id="label_m">0</span>
    </label>
    <div style="margin-top:1rem; font-family:monospace;">
      dof = n − m = <span id="label_dof">2</span>
    </div>
  </div>
</div>
<div style="max-width:800px;margin-top:0.8em;">
    - if dof = 2, you have a full plane; dof = 1, a line; dof = 0, a point<br/>
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
  const canvas = document.getElementById('dof-canvas');
  const ctx = canvas.getContext('2d');
  const n_slider = document.getElementById('var_n');
  const m_slider = document.getElementById('con_m');
  const label_n = document.getElementById('label_n');
  const label_m = document.getElementById('label_m');
  const label_dof = document.getElementById('label_dof');
  function drawDoF() {
    let n = +n_slider.value;
    let m = +m_slider.value;
    let dof = Math.max(0, n - m);
    label_n.textContent = n;
    label_m.textContent = m;
    label_dof.textContent = dof;
    // clear
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.save();
    ctx.translate(canvas.width/2, canvas.height/2);
    ctx.lineWidth = 2;
    // 2D
    if (n === 2) {
      // full plane (dof 2)
      if (dof === 2) {
        ctx.strokeStyle="#ddd";
        for(let x=-80;x<=80;x+=20){
          ctx.beginPath(); ctx.moveTo(x,-80); ctx.lineTo(x,80); ctx.stroke();
          ctx.beginPath(); ctx.moveTo(-80,x); ctx.lineTo(80,x); ctx.stroke();
        }
        ctx.strokeStyle="#000";
        ctx.beginPath(); ctx.moveTo(-85,0); ctx.lineTo(85,0); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(0,-85); ctx.lineTo(0,85); ctx.stroke();
      }
      // 1 constraint: line
      if (dof === 1) {
        ctx.strokeStyle="#77c";
        ctx.beginPath();
        ctx.moveTo(-90,-60);
        ctx.lineTo(90,60);
        ctx.stroke();
      }
      // 2 constraints: point
      if (dof === 0) {
        ctx.fillStyle = "#d30";
        ctx.beginPath();
        ctx.arc(0,0,8,0,2*Math.PI);
        ctx.fill();
      }
    }
    // 1D line: just a line
    if (n === 1) {
      if (dof === 1) {
        ctx.strokeStyle = "#000";
        ctx.beginPath();
        ctx.moveTo(-85,0); ctx.lineTo(85,0); ctx.stroke();
      } else {
        ctx.fillStyle = "#d30";
        ctx.beginPath();
        ctx.arc(0,0,8,0,2*Math.PI);
        ctx.fill();
      }
    }
    // 3D simulated with wireframe square, line, point (schematic)
    if (n === 3) {
      if (dof === 3) {
        ctx.strokeStyle="#ccc";
        ctx.strokeRect(-60,-60,120,120);
        ctx.strokeStyle="#999";
        ctx.strokeRect(-30,-30,60,60);
        ctx.strokeStyle="#000";
        ctx.strokeRect(-80,-80,160,160);
      } else if (dof === 2) {
        ctx.strokeStyle="#77c";
        ctx.beginPath();
        ctx.ellipse(0,0,70,25,0,0,2*Math.PI);
        ctx.stroke();
      } else if (dof === 1) {
        ctx.strokeStyle="#c58";
        ctx.beginPath();
        ctx.moveTo(-80,0); ctx.lineTo(80,0); ctx.stroke();
      } else {
        ctx.fillStyle = "#d30";
        ctx.beginPath();
        ctx.arc(0,0,8,0,2*Math.PI);
        ctx.fill();
      }
    }
    ctx.restore();
  }
  n_slider.addEventListener('input', function(){
    m_slider.max = n_slider.value;
    drawDoF();
  });
  m_slider.addEventListener('input', drawDoF);
  drawDoF();
});
</script>
