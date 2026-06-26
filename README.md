<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Steal a Brainrot - Fixed</title>

<style>
body { margin:0; overflow:hidden; font-family:Arial; }

#hud{
  position:absolute;
  top:10px;
  left:10px;
  background:rgba(0,0,0,0.5);
  color:white;
  padding:10px;
  z-index:10;
}

#admin{
  position:absolute;
  top:10px;
  right:10px;
  background:rgba(255,0,0,0.5);
  color:white;
  padding:10px;
  z-index:10;
  cursor:move;
}

#shiftBtn{
  position:absolute;
  bottom:200px;
  right:10px;
  padding:10px;
  background:#00d4ff;
  border:none;
  z-index:10;
}

#games{
  position:absolute;
  top:120px;
  left:10px;
  background:rgba(0,0,0,0.4);
  color:white;
  padding:10px;
  z-index:10;
}

button{
  display:block;
  margin:5px 0;
}

/* joystick */
#joystick{
  position:absolute;
  bottom:40px;
  left:40px;
  width:120px;
  height:120px;
  background:rgba(255,255,255,0.1);
  border-radius:50%;
  z-index:10;
}

#stick{
  width:50px;
  height:50px;
  background:#00d4ff;
  border-radius:50%;
  position:absolute;
  top:35px;
  left:35px;
}
</style>
</head>

<body>

<div id="hud">
👤 Jose<br>
💰 Money: $<span id="money">20</span>
</div>

<div id="admin">
👑 ADMIN PANEL<br>
<button onclick="addMoney()">+100 Money</button>
<button onclick="spawnBrainrot()">Spawn Brainrot</button>
</div>

<button id="shiftBtn" onclick="toggleShift()">🔄 Shift Lock OFF</button>

<div id="games">
🎮 Games<br>
<button onclick="setGame('Brainrot')">Steal a Brainrot</button>
<button onclick="setGame('Obby')">Obby World</button>
<button onclick="setGame('FPS')">FPS Arena</button>
</div>

<div id="joystick">
  <div id="stick"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

/* WORLD */
let scene=new THREE.Scene();
scene.background=new THREE.Color(0x87ceeb);

let camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,1000);

let renderer=new THREE.WebGLRenderer();
renderer.setSize(innerWidth,innerHeight);
document.body.appendChild(renderer.domElement);

/* LIGHT */
let light=new THREE.DirectionalLight(0xffffff,1);
light.position.set(5,10,5);
scene.add(light);

/* GROUND */
let ground=new THREE.Mesh(
  new THREE.BoxGeometry(60,1,60),
  new THREE.MeshStandardMaterial({color:0x228B22})
);
ground.position.y=-1;
scene.add(ground);

/* PLAYER */
let player=new THREE.Mesh(
  new THREE.BoxGeometry(1,2,1),
  new THREE.MeshStandardMaterial({color:0x00d4ff})
);
player.position.y=1;
scene.add(player);

/* GAME STATE */
let money=20;
let game="Brainrot";

/* BRAINROTS */
let brainrots=[];

function spawnBrainrot(){
  let b=new THREE.Mesh(
    new THREE.SphereGeometry(0.3,12,12),
    new THREE.MeshStandardMaterial({color:0x00ff00})
  );
  b.position.set(Math.random()*20-10,0,Math.random()*20-10);
  scene.add(b);
  brainrots.push(b);
}

/* SHIFT LOCK */
let shift=false;

function toggleShift(){
  shift=!shift;
  document.getElementById("shiftBtn").innerText =
  shift ? "🔄 Shift Lock ON" : "🔄 Shift Lock OFF";
}

/* GAMES */
function setGame(g){
  game=g;
}

/* ADMIN */
function addMoney(){
  money+=100;
  document.getElementById("money").innerText=money;
}

/* DRAG ADMIN */
let admin=document.getElementById("admin");
let drag=false;

admin.onmousedown=()=>drag=true;
document.onmouseup=()=>drag=false;

document.onmousemove=(e)=>{
  if(drag){
    admin.style.left=e.clientX+"px";
    admin.style.top=e.clientY+"px";
  }
};

/* JOYSTICK */
let joystick=document.getElementById("joystick");
let stick=document.getElementById("stick");

let xMove=0,zMove=0;

joystick.addEventListener("touchmove",e=>{
  let t=e.touches[0];
  let r=joystick.getBoundingClientRect();

  xMove=(t.clientX-r.left-60)/40;
  zMove=(t.clientY-r.top-60)/40;

  stick.style.left=(t.clientX-r.left-25)+"px";
  stick.style.top=(t.clientY-r.top-25)+"px";
});

joystick.addEventListener("touchend",()=>{
  xMove=0;
  zMove=0;
  stick.style.left="35px";
  stick.style.top="35px";
});

/* CAMERA FOLLOW (SHIFT LOCK FIX) */
function updateCamera(){
  if(shift){
    // over-the-shoulder view
    camera.position.x = player.position.x - Math.sin(player.rotation.y)*2;
    camera.position.z = player.position.z - Math.cos(player.rotation.y)*2;
    camera.position.y = player.position.y + 2;
    camera.lookAt(player.position);
  }else{
    // normal third person
    camera.position.x = player.position.x;
    camera.position.z = player.position.z + 5;
    camera.position.y = player.position.y + 2;
    camera.lookAt(player.position);
  }
}

/* COLLECT */
function collect(){
  for(let i=brainrots.length-1;i>=0;i--){
    let b=brainrots[i];

    let d=player.position.distanceTo(b.position);

    if(d<1.5){
      scene.remove(b);
      brainrots.splice(i,1);
      money+=10;
      document.getElementById("money").innerText=money;
    }
  }
}

/* LOOP */
function animate(){
  requestAnimationFrame(animate);

  player.position.x += xMove*0.15;
  player.position.z += zMove*0.15;

  collect();
  updateCamera();

  renderer.render(scene,camera);
}

animate();

</script>

</body>
</html>
