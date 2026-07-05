As it's not yet even implemented, this goes to Alpha stage:

# Laegna Visualization Mapping  
### (GitHub‑flavored Markdown with inline KaTeX)

This document defines the **first practical Laegna coordinate system** suitable for visualization.  
It is based on the double‑unit ball, linear/exponent dual bands, and the interlace rule.

---

## 1. Double‑Unit Ball

Laegna uses a *double‑unit ball*:

- The **unit** is not the radius.  
- The unit is **one quarter of the diameter**.

Thus the full diameter is:

- from south pole to north pole  
- measured as  
  $4$ units.

We define a vertical coordinate:

- $v \in [0,4]$  
  - $v = 0$ → south pole  
  - $v = 2$ → equator  
  - $v = 4$ → north pole

This axis is where **linear unfolding** and **exponential folding** meet.

---

## 2. Linear vs Exponential Coordinates

Every triangle (sphere) and every cell (chessfield) receives **two coordinates**:

### Linear band (local)
This grows at constant speed:

- $T_{\text{lin}} = v$

### Exponential band (global)
This grows rapidly from south to equator,  
then collapses symmetrically from equator to north:

- For $v \in [0,2]$:  
  $R_{\text{exp}} = 2v$

- For $v \in [2,4]$:  
  $R_{\text{exp}} = 8 - 2v$

This expresses the Laegna rule:

> What is linear in one direction is exponential in the other,  
> and the flip happens at the equator.

---

## 3. The $1/4$ Dimension

The $1/4$ dimension is the Laegna base‑2 cut:

- First cut: $0 \rightarrow 2 \rightarrow 4$  
- Second cut: $0 \rightarrow 1 \rightarrow 2$ and $2 \rightarrow 3 \rightarrow 4$

This mirrors:

- “$1$ between $0$ and $2$” in the first octave  
- “$0$ between $0$ and \infty$” in the second octave

Precision bands switch roles:

- linear becomes exponential  
- exponential becomes linear

This is why the equator ($v=2$) is the **flip point**.

---

## 4. Interlace Anchors

Interlacing happens where:

- $T_{\text{lin}}$ and $R_{\text{exp}}$  
- fall into the same Laegna band  
- or mirror each other across the equator.

We check:

- $T_{\text{lin}} = R_{\text{exp}}$  
- or both in $\{0,2,4\}$  
- or symmetric under the equator flip.

These are the **true Laegna points**.

Everything else is scaffolding.

---

## 5. How This Drives Visualization

For each triangle on the sphere:

1. Compute its average vertical position $v$.
2. Assign:
   - $T_{\text{lin}} = v$
   - $R_{\text{exp}}$ using the piecewise rule.
3. Highlight triangles where linear and exponent coordinates **meet or mirror**.

For each cell on the chessfield:

1. Assign $(X_{\text{lin}}, Y_{\text{exp}})$ using the same $0\!-\!2\!-\!4$ structure.
2. Highlight cells where $X_{\text{lin}}$ and $Y_{\text{exp}}$ **agree**.

This gives us a **first Laegna‑consistent interlacing rule** we can implement.

---

## 6. Summary of the Mapping

- Vertical coordinate:  
  $v \in [0,4]$

- Linear band:  
  $T_{\text{lin}} = v$

- Exponential band:  
  $R_{\text{exp}} = 2v$ for $v \le 2$  
  $R_{\text{exp}} = 8 - 2v$ for $v \ge 2$

- Interlace anchors:  
  $T_{\text{lin}} = R_{\text{exp}}$  
  or both in $\{0,2,4\}$  
  or symmetric under equator flip.

This is the first usable Laegna coordinate system for visualization.

# Laegna Lane Visualization — First Implementation  
### Sphere + Chessfield + Linear/Exponent Bands + Interlace Anchors

This file implements the first working visualization of Laegna geometry:

- A sphere subdivided linearly  
- A chessfield subdivided exponentially  
- Each triangle/cell receives **Laegna coordinates**  
  - $T_{\text{lin}} = v$  
  - $R_{\text{exp}} = 2v$ for $v \le 2$  
  - $R_{\text{exp}} = 8 - 2v$ for $v \ge 2$  
- Interlace anchors where linear and exponent bands **meet or mirror**  
- Hovering highlights the same Laegna coordinate on both sphere and chessfield

---

Something like this, let's see:

## `laegna-visualization.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Laegna Lane Visualization</title>
<style>
  html, body { margin:0; height:100%; background:#111; color:#eee; font-family:sans-serif; }
  #root { display:flex; height:100%; }
  #sphereCanvas, #squareCanvas { flex:1; display:block; }
  #controls {
    position:absolute; top:10px; left:10px;
    background:#0008; padding:8px 12px; border-radius:6px;
    font-size:14px; z-index:1000;
  }
</style>
</head>
<body>

<div id="controls">
  Subdivision level:
  <select id="levelSelect">
    <option value="0">0 (8 triangles)</option>
    <option value="1">1</option>
    <option value="2" selected>2</option>
    <option value="3">3</option>
  </select>
</div>

<div id="root">
  <canvas id="sphereCanvas"></canvas>
  <canvas id="squareCanvas"></canvas>
</div>

<script type="module">
import * as THREE from "https://unpkg.com/three@0.159.0/build/three.module.js";
window.THREE = THREE;

/* vec helpers */
function vec3(x,y,z){ return {x,y,z}; }
function norm(v){
  const l = Math.hypot(v.x,v.y,v.z);
  return {x:v.x/l,y:v.y/l,z:v.z/l};
}
function mid(a,b){
  return {x:(a.x+b.x)/2,y:(a.y+b.y)/2,z:(a.z+b.z)/2};
}

/* octahedron */
function buildOctahedron(){
  const V = [
    vec3(1,0,0), vec3(-1,0,0),
    vec3(0,1,0), vec3(0,-1,0),
    vec3(0,0,1), vec3(0,0,-1)
  ];
  const T = [
    [0,2,4],[2,1,4],[1,3,4],[3,0,4],
    [2,0,5],[1,2,5],[3,1,5],[0,3,5]
  ];
  return {V,T};
}

/* subdivision */
function subdivide(V,T,depth){
  for(let d=0; d<depth; d++){
    const midCache = new Map();
    function key(a,b){ return a<b ? a+"_"+b : b+"_"+a; }
    function getMidIndex(i,j){
      const k = key(i,j);
      if(midCache.has(k)) return midCache.get(k);
      const m = mid(V[i],V[j]);
      const idx = V.length;
      V.push(norm(m));
      midCache.set(k,idx);
      return idx;
    }
    const newT = [];
    for(const tri of T){
      const [a,b,c] = tri;
      const ab = getMidIndex(a,b);
      const bc = getMidIndex(b,c);
      const ca = getMidIndex(c,a);
      newT.push([a,ab,ca]);
      newT.push([b,bc,ab]);
      newT.push([c,ca,bc]);
      newT.push([ab,bc,ca]);
    }
    T = newT;
  }
  for(let i=0;i<V.length;i++) V[i] = norm(V[i]);
  return {V,T};
}

/* adjacency */
function buildAdjacency(V,T){
  const vertTris = Array(V.length).fill(0).map(()=>[]);
  const edgeTris = new Map();
  function edgeKey(a,b){ return a<b ? a+"_"+b : b+"_"+a; }
  T.forEach((tri,tid)=>{
    const [a,b,c] = tri;
    vertTris[a].push(tid);
    vertTris[b].push(tid);
    vertTris[c].push(tid);
    [[a,b],[b,c],[c,a]].forEach(([i,j])=>{
      const k = edgeKey(i,j);
      if(!edgeTris.has(k)) edgeTris.set(k,[]);
      edgeTris.get(k).push(tid);
    });
  });
  return {vertTris,edgeTris};
}

/* tangent basis */
function tangentBasis(v){
  const n = norm(v);
  let tx = Math.abs(n.x) < 0.9 ? vec3(1,0,0) : vec3(0,1,0);
  const dot = tx.x*n.x + tx.y*n.y + tx.z*n.z;
  tx = norm(vec3(tx.x - n.x*dot, tx.y - n.y*dot, tx.z - n.z*dot));
  const ty = norm(vec3(
    n.y*tx.z - n.z*tx.y,
    n.z*tx.x - n.x*tx.z,
    n.x*tx.y - n.y*tx.x
  ));
  return {tx,ty};
}
function projectToTangent(v,center,tx,ty){
  const d = vec3(v.x-center.x,v.y-center.y,v.z-center.z);
  return {
    x: d.x*tx.x + d.y*tx.y + d.z*tx.z,
    y: d.x*ty.x + d.y*ty.y + d.z*ty.z
  };
}

/* extract squares */
function extractSquares(V,vertTris,T){
  const squares = [];
  vertTris.forEach((tris,vid)=>{
    if(tris.length!==4) return;
    const center = V[vid];
    const {tx,ty} = tangentBasis(center);
    const triInfos = tris.map(tid=>{
      const tri = T[tid];
      const others = tri.filter(i=>i!==vid);
      const p1 = projectToTangent(V[others[0]],center,tx,ty);
      const p2 = projectToTangent(V[others[1]],center,tx,ty);
      const cx = (p1.x+p2.x)*0.5;
      const cy = (p1.y+p2.y)*0.5;
      const angle = Math.atan2(cy,cx);
      return {tid,angle};
    });
    triInfos.sort((a,b)=>a.angle-b.angle);
    squares.push({
      vertex: vid,
      triangles: triInfos.map(t=>t.tid),
      neighbors: []
    });
  });
  return squares;
}

/* link squares */
function linkSquares(squares,T,edgeTris){
  const triToSquare = new Map();
  squares.forEach((sq,i)=>{
    sq.triangles.forEach(tid=>triToSquare.set(tid,i));
  });
  function edgeKey(a,b){ return a<b ? a+"_"+b : b+"_"+a; }

  squares.forEach((sq,i)=>{
    const neigh = new Set();
    for(const tid of sq.triangles){
      const tri = T[tid];
      const edges = [[tri[0],tri[1]],[tri[1],tri[2]],[tri[2],tri[0]]];
      for(const [a,b] of edges){
        const k = edgeKey(a,b);
        const trisOnEdge = edgeTris.get(k) || [];
        for(const otherTid of trisOnEdge){
          if(otherTid===tid) continue;
          const otherSq = triToSquare.get(otherTid);
          if(otherSq!==undefined && otherSq!==i){
            neigh.add(otherSq);
          }
        }
      }
    }
    sq.neighbors = Array.from(neigh);
  });

  const visited = new Set();
  function rotateArray(arr,rot){
    const n = arr.length;
    const out = [];
    for(let i=0;i<n;i++){
      out.push(arr[(i-rot+n)%n]);
    }
    return out;
  }
  function fixOrientation(root){
    visited.add(root);
    const sq = squares[root];
    for(const nb of sq.neighbors){
      if(visited.has(nb)) continue;
      const sq2 = squares[nb];
      let sharedTri = null;
      for(const tid of sq.triangles){
        if(sq2.triangles.includes(tid)){ sharedTri = tid; break; }
      }
      if(sharedTri!==null){
        const idx1 = sq.triangles.indexOf(sharedTri);
        const idx2 = sq2.triangles.indexOf(sharedTri);
        const rot = (idx1-idx2+4)%4;
        if(rot!==0){
          sq2.triangles = rotateArray(sq2.triangles,rot);
        }
      }
      fixOrientation(nb);
    }
  }
  if(squares.length>0) fixOrientation(0);
  return squares;
}

/* Laegna coordinates */
function laegnaCoordsForTriangle(V,tri){
  const v0 = V[tri[0]];
  const v1 = V[tri[1]];
  const v2 = V[tri[2]];
  const avgZ = (v0.z + v1.z + v2.z)/3;

  const v = (avgZ + 1)*2;

  const T_lin = v;

  let R_exp;
  if(v <= 2){
    R_exp = 2*v;
  } else {
    R_exp = 8 - 2*v;
  }

  return {T_lin,R_exp};
}

/* build sphere */
let sphereMesh=null;
let sphereData=null;
let triToSquare=null;
let squarePos=null;
let hoveredSquareId=-1;
let hoveredTriangleIndex=-1;

function buildLaegnaSphere(level){
  let {V,T} = buildOctahedron();
  ({V,T} = subdivide(V,T,level));
  const {vertTris,edgeTris} = buildAdjacency(V,T);
  let squares = extractSquares(V,vertTris,T);
  squares = linkSquares(squares,T,edgeTris);

  const gridSize = Math.round(Math.sqrt(squares.length));
  const grid = Array(gridSize).fill(0).map(()=>Array(gridSize).fill(-1));
  const visited = new Set();
  const q = [];
  if(squares.length>0){
    q.push(0);
    visited.add(0);
  }
  let idx=0;
  while(q.length>0 && idx<gridSize*gridSize){
    const s = q.shift();
    const r = Math.floor(idx/gridSize);
    const c = idx%gridSize;
    grid[r][c]=s;
    idx++;
    for(const nb of squares[s].neighbors){
      if(!visited.has(nb)){
        visited.add(nb);
        q.push(nb);
      }
    }
  }

  return {V,T,squares,gridSize,grid};
}

/* rendering */
const sphereCanvas = document.getElementById("sphereCanvas");
const squareCanvas = document.getElementById("squareCanvas");
const squareCtx = squareCanvas.getContext("2d");

function resize(){
  const w = window.innerWidth;
  const h = window.innerHeight;
  sphereCanvas.width = w/2;
  sphereCanvas.height = h;
  squareCanvas.width = w/2;
  squareCanvas.height = h;
}
resize();
window.addEventListener("resize",resize);

const renderer = new THREE.WebGLRenderer({canvas:sphereCanvas,antialias:true});
renderer.setPixelRatio(window.devicePixelRatio);

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x222222);

const camera = new THREE.PerspectiveCamera(
  45,
  sphereCanvas.width/sphereCanvas.height,
  0.1,10
);
camera.position.set(0,0,3);

const dirLight = new THREE.DirectionalLight(0xffffff,1.2);
dirLight.position.set(2,2,3);
scene.add(dirLight);
scene.add(new THREE.AmbientLight(0x808080));

function buildSphere(level){
  sphereData = buildLaegnaSphere(level);
  const V = sphereData.V;
  const T = sphereData.T;
  const squares = sphereData.squares;
  const grid = sphereData.grid;
  const n = sphereData.gridSize;

  triToSquare = new Array(T.length).fill(null);
  squares.forEach((sq,sid)=>{
    sq.triangles.forEach((tid,idx)=>{
      triToSquare[tid] = {squareId:sid,triIndex:idx};
    });
  });

  squarePos = {};
  for(let r=0;r<n;r++){
    for(let c=0;c<n;c++){
      const sqId = grid[r][c];
      if(sqId>=0) squarePos[sqId]={row:r,col:c};
    }
  }

  const triCount = T.length;
  const positions = new Float32Array(triCount*3*3);
  const triIds = new Float32Array(triCount*3);
  const triColors = new Float32Array(triCount*3*3);

  for(let tid=0; tid<triCount; tid++){
    const tri = T[tid];
    const map = triToSquare[tid];

    const coords = laegnaCoordsForTriangle(V,tri);
    const isInterlace = Math.abs(coords.T_lin - coords.R_exp) < 0.01;

    let baseColor = isInterlace ? [1.0,0.0,0.0] : [0.8,0.8,0.8];

    for(let k=0;k<3;k++){
      const vi = tri[k];
      const v = V[vi];
      const idx = tid*9 + k*3;
      positions[idx+0]=v.x;
      positions[idx+1]=v.y;
      positions[idx+2]=v.z;
      triColors[idx+0]=baseColor[0];
      triColors[idx+1]=baseColor[1];
      triColors[idx+2]=baseColor[2];
      triIds[tid*3+k]=tid;
    }
  }

  const geom = new THREE.BufferGeometry();
  geom.setAttribute('position',new THREE.BufferAttribute(positions,3));
  geom.setAttribute('triId',new THREE.BufferAttribute(triIds,1));
  geom.setAttribute('triColor',new THREE.BufferAttribute(triColors,3));
  geom.computeVertexNormals();

  const vertexShader = `
    attribute float triId;
    attribute vec3 triColor;
    flat varying float vTriId;
    varying vec3 vTriColor;
    varying vec3 vNormal;
    void main(){
      vTriId = triId;
      vTriColor = triColor;
      vNormal = normalize(normalMatrix * normal);
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
    }
  `;
  const fragmentShader = `
    precision highp float;
    flat varying float vTriId;
    varying vec3 vTriColor;
    varying vec3 vNormal;
    uniform float uSelectedTri;
    uniform vec3 uLightDir;
    void main(){
      vec3 base = vTriColor;
      if(abs(vTriId - uSelectedTri) < 0.5){
        base = mix(base, vec3(1.0,1.0,0.0), 0.7);
      }
      float diff = max(dot(normalize(vNormal), normalize(uLightDir)), 0.0);
      vec3 col = base * (0.2 + 0.8*diff);
      gl_FragColor = vec4(col,1.0);
    }
  `;
  const mat = new THREE.ShaderMaterial({
    vertexShader,
    fragmentShader,
    uniforms:{
      uSelectedTri:{value:-1.0},
      uLightDir:{value:new THREE.Vector3(2,2,3).normalize()}
    },
    side:THREE.DoubleSide
  });

  if(sphereMesh){
    scene.remove(sphereMesh);
    sphereMesh.geometry.dispose();
    sphereMesh.material.dispose();
  }
  sphereMesh = new THREE.Mesh(geom,mat);
  scene.add(sphereMesh);

  drawSquarefield();
}

/* chessfield */
function drawSquarefield(){
  if(!sphereData) return;
  const grid = sphereData.grid;
  const n = sphereData.gridSize;
  const w = squareCanvas.width;
  const h = squareCanvas.height;
  const cellW = n? w/n : w;
  const cellH = n? h/n : h;
  squareCtx.clearRect(0,0,w,h);

  for(let r=0;r<n;r++){
    for(let c=0;c<n;c++){
      const sqId = grid[r][c];
      if(sqId<0) continue;

      const parity = (r+c)%2;
      squareCtx.fillStyle = parity?"#000":"#fff";
      squareCtx.fillRect(c*cellW,r*cellH,cellW,cellH);

      const sx = c*cellW;
      const sy = r*cellH;
      const cx = sx+cellW*0.5;
      const cy = sy+cellH*0.5;
      const corners = [
        [sx,sy],
        [sx+cellW,sy],
        [sx+cellW,sy+cellH],
        [sx,sy+cellH]
      ];
      const tris = [
        [cx,cy,corners[0][0],corners[0][1],corners[1][0],corners[1][1]],
        [cx,cy,corners[1][0],corners[1][1],corners[2][0],corners[2][1]],
        [cx,cy,corners[2][0],corners[2][1],corners[3][0],corners[3][1]],
        [cx,cy,corners[3][0],corners[3][1],corners[0][0],corners[0][1]]
      ];
      
      squareCtx.strokeStyle="#444";
      squareCtx.lineWidth=1;
      
      for(let t=0;t<4;t++){
        const [ax,ay,bx,by,cx2,cy2]=tris[t];
        squareCtx.beginPath();
        squareCtx.moveTo(ax,ay);
        squareCtx.lineTo(bx,by);
        squareCtx.lineTo(cx2,cy2);
        squareCtx.closePath();
        squareCtx.stroke();
      }
      
      if(sqId===hoveredSquareId && hoveredTriangleIndex>=0){
        const [ax,ay,bx,by,cx2,cy2]=tris[hoveredTriangleIndex];
        squareCtx.beginPath();
        squareCtx.moveTo(ax,ay);
        squareCtx.lineTo(bx,by);
        squareCtx.lineTo(cx2,cy2);
        squareCtx.closePath();
        squareCtx.fillStyle="rgba(255,0,0,0.6)";
        squareCtx.fill();
        squareCtx.strokeStyle="#f00";
        squareCtx.lineWidth=2;
        squareCtx.stroke();
      }
    }
  }
}

/* rotation */
let isDragging=false,lastX=0,lastY=0;
sphereCanvas.addEventListener("mousedown",e=>{
  isDragging=true; lastX=e.clientX; lastY=e.clientY;
});
sphereCanvas.addEventListener("mouseup",()=>isDragging=false);
sphereCanvas.addEventListener("mouseleave",()=>isDragging=false);
sphereCanvas.addEventListener("mousemove",e=>{
  if(!sphereMesh) return;
  if(!isDragging) return;
  const dx=e.clientX-lastX, dy=e.clientY-lastY;
  lastX=e.clientX; lastY=e.clientY;
  sphereMesh.rotation.y += dx*0.005;
  sphereMesh.rotation.x += dy*0.005;
});

/* hover mapping */
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

sphereCanvas.addEventListener("mousemove",e=>{
  if(!sphereMesh || !sphereData) return;
  const rect = sphereCanvas.getBoundingClientRect();
  mouse.x = ((e.clientX-rect.left)/rect.width)*2-1;
  mouse.y = -((e.clientY-rect.top)/rect.height)*2+1;
  raycaster.setFromCamera(mouse,camera);
  const hits = raycaster.intersectObject(sphereMesh);
  if(hits.length>0){
    const tid = hits[0].faceIndex;
    sphereMesh.material.uniforms.uSelectedTri.value = tid;
    hoveredSquareId = triToSquare[tid]?.squareId ?? -1;
    hoveredTriangleIndex = triToSquare[tid]?.triIndex ?? -1;
  } else {
    sphereMesh.material.uniforms.uSelectedTri.value = -1;
    hoveredSquareId = -1;
    hoveredTriangleIndex = -1;
  }
  drawSquarefield();
});

/* level selector */
document.getElementById("levelSelect").addEventListener("change",e=>{
  const level = parseInt(e.target.value,10);
  buildSphere(level);
});

/* start */
function init(){
  const level = 2;
  buildSphere(level);
}
init();

function animate(){
  requestAnimationFrame(animate);
  renderer.setSize(sphereCanvas.width,sphereCanvas.height,false);
  camera.aspect = sphereCanvas.width/sphereCanvas.height;
  camera.updateProjectionMatrix();
  renderer.render(scene,camera);
}
animate();
</script>

</body>
</html>
