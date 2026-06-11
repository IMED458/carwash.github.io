<!DOCTYPE html>
<html lang="ka">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>ავტოსამრეცხაო 3D — ჩაბინაანი, ახმეტა</title>
<style>
  html,body{margin:0;padding:0;width:100%;height:100%;overflow:hidden;background:#0a0f1c;
    font-family:'Noto Sans Georgian','Segoe UI',sans-serif;-webkit-user-select:none;user-select:none}
  #c{position:absolute;inset:0;touch-action:none}
  .hud{position:absolute;left:8px;right:8px;display:flex;flex-wrap:wrap;gap:6px;z-index:5}
  .top{top:8px}
  .bot{bottom:max(8px, env(safe-area-inset-bottom))}
  .card{background:rgba(12,16,24,.85);color:#fff;padding:8px 12px;border-radius:10px;font-size:13px;font-weight:700}
  .card small{display:block;font-weight:400;font-size:10px;opacity:.75}
  button{padding:8px 11px;border-radius:8px;border:1px solid rgba(255,255,255,.18);
    background:rgba(255,255,255,.07);color:#fff;font-size:12px;cursor:pointer;
    font-family:inherit;-webkit-tap-highlight-color:transparent}
  button.on{border-color:#3aa0ff;background:rgba(58,160,255,.25)}
</style>
</head>
<body>
<div id="c"></div>
<div class="hud top">
  <div class="card">ავტოსამრეცხაო • 3 ბოქსი — ჩაბინაანი, ახმეტა
    <small>ბრუნვა — გადაათრიე თითით • ზუმი — ორი თითით (პინჩი)</small>
  </div>
</div>
<div class="hud bot" id="bar">
  <button data-v="iso" class="on">იზომეტრია</button>
  <button data-v="top">ზედხედი</button>
  <button data-v="front">ფასადი</button>
  <button data-v="side">გვერდი</button>
  <button data-v="interior">ბოქსის შიგნით</button>
  <button data-v="truck">სატვირთო ბოქსი</button>
  <button id="nightBtn">🌙 ღამის ხედი</button>
  <button id="labelsBtn" class="on">🏷 ზომები ✓</button>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
(function(){
"use strict";
/* ავტოსამრეცხაოს 3D კონცეფცია — ს/კ 50.12.32.483, სოფ. ჩაბინაანი, ახმეტა.
   ერთეული: მეტრი. ეს კონცეპტუალური მოდელია — მშენებლობამდე საჭიროა
   ადგილზე ზუსტი აზომვა და კონსტრუქტორის გაანგარიშება. */

var mount = document.getElementById("c");
var renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setPixelRatio(Math.min(window.devicePixelRatio,2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
mount.appendChild(renderer.domElement);

var scene = new THREE.Scene();
var DAY_BG = new THREE.Color(0xcfe0ea), NIGHT_BG = new THREE.Color(0x0a0f1c);
scene.background = DAY_BG.clone();
scene.fog = new THREE.Fog(DAY_BG.clone(), 90, 220);
var camera = new THREE.PerspectiveCamera(50, 1, 0.1, 500);

var hemi = new THREE.HemisphereLight(0xeaf2ff, 0x6b6f63, 0.85); scene.add(hemi);
var sun = new THREE.DirectionalLight(0xfff3df, 1.05);
sun.position.set(35,45,-25); sun.castShadow = true;
sun.shadow.mapSize.set(2048,2048);
sun.shadow.camera.left=-45; sun.shadow.camera.right=45;
sun.shadow.camera.top=45; sun.shadow.camera.bottom=-45; sun.shadow.camera.far=150;
scene.add(sun);

var nightLights=[], ledMaterials=[];
function addNightPoint(x,y,z,intensity,dist,color){
  var p=new THREE.PointLight(color||0xbfd9ff,0,dist||14,2);
  p.position.set(x,y,z); p.userData.ni=intensity||0.9;
  scene.add(p); nightLights.push(p);
}

var M={
  steel:new THREE.MeshStandardMaterial({color:0x2c3340,roughness:.45,metalness:.7}),
  panel:new THREE.MeshStandardMaterial({color:0x9aa3ad,roughness:.6,metalness:.35}),
  panelBlue:new THREE.MeshStandardMaterial({color:0x1f4f8f,roughness:.55,metalness:.4}),
  roofPoly:new THREE.MeshStandardMaterial({color:0xa8c8e6,roughness:.25,metalness:.1,transparent:true,opacity:.55,side:THREE.DoubleSide}),
  concrete:new THREE.MeshStandardMaterial({color:0xb8bbbe,roughness:.95}),
  concreteBay:new THREE.MeshStandardMaterial({color:0xa7abaf,roughness:.9}),
  asphalt:new THREE.MeshStandardMaterial({color:0x3a3e44,roughness:1}),
  drive:new THREE.MeshStandardMaterial({color:0x8e9296,roughness:.95}),
  grass:new THREE.MeshStandardMaterial({color:0x7d8f63,roughness:1}),
  drain:new THREE.MeshStandardMaterial({color:0x23262a,roughness:.8}),
  grate:new THREE.MeshStandardMaterial({color:0x55606b,roughness:.5,metalness:.6}),
  yellow:new THREE.MeshStandardMaterial({color:0xe0b000,roughness:.6}),
  white:new THREE.MeshStandardMaterial({color:0xeef0f2,roughness:.7}),
  dark:new THREE.MeshStandardMaterial({color:0x1b1e23,roughness:.7}),
  red:new THREE.MeshStandardMaterial({color:0xc23b3b,roughness:.7}),
  houseWall:new THREE.MeshStandardMaterial({color:0xcfc4ae,roughness:.95}),
  houseRoof:new THREE.MeshStandardMaterial({color:0x7a4a3a,roughness:.9}),
  glassDark:new THREE.MeshStandardMaterial({color:0x33414f,roughness:.2,metalness:.6}),
  arrow:new THREE.MeshBasicMaterial({color:0x2faf6e,transparent:true,opacity:.85}),
  lineYellow:new THREE.MeshBasicMaterial({color:0xf2c200}),
  lineWhite:new THREE.MeshBasicMaterial({color:0xf5f5f5})
};
function ledMat(){
  var m=new THREE.MeshStandardMaterial({color:0xdfe9f5,emissive:0xbfd9ff,emissiveIntensity:.15,roughness:.4});
  ledMaterials.push(m); return m;
}
function box(w,h,d,mat,x,y,z,parent,shadow){
  var m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat);
  m.position.set(x,y,z);
  m.castShadow = shadow!==false; m.receiveShadow=true;
  (parent||scene).add(m); return m;
}
function cyl(rt,rb,h,mat,x,y,z,parent,seg){
  var m=new THREE.Mesh(new THREE.CylinderGeometry(rt,rb,h,seg||20),mat);
  m.position.set(x,y,z); m.castShadow=true; m.receiveShadow=true;
  (parent||scene).add(m); return m;
}

var labelsGroup=new THREE.Group(); labelsGroup.name="Labels"; scene.add(labelsGroup);
function makeLabel(text,x,y,z,scale,accent){
  scale=scale||1;
  var pad=28, cnv=document.createElement("canvas");
  var font="bold 46px 'Noto Sans Georgian','Segoe UI',sans-serif";
  var ctx=cnv.getContext("2d"); ctx.font=font;
  var tw=ctx.measureText(text).width;
  cnv.width=Math.ceil(tw+pad*2); cnv.height=92;
  var c=cnv.getContext("2d");
  c.fillStyle=accent?"rgba(20,70,140,.92)":"rgba(18,22,28,.88)";
  var r=18;
  c.beginPath();
  c.moveTo(r,0);c.lineTo(cnv.width-r,0);c.quadraticCurveTo(cnv.width,0,cnv.width,r);
  c.lineTo(cnv.width,cnv.height-r);c.quadraticCurveTo(cnv.width,cnv.height,cnv.width-r,cnv.height);
  c.lineTo(r,cnv.height);c.quadraticCurveTo(0,cnv.height,0,cnv.height-r);
  c.lineTo(0,r);c.quadraticCurveTo(0,0,r,0);c.closePath();c.fill();
  c.font=font;c.fillStyle="#fff";c.textBaseline="middle";
  c.fillText(text,pad,cnv.height/2+2);
  var tex=new THREE.CanvasTexture(cnv); tex.minFilter=THREE.LinearFilter;
  var sp=new THREE.Sprite(new THREE.SpriteMaterial({map:tex,depthTest:false}));
  var k=0.011*scale;
  sp.scale.set(cnv.width*k,cnv.height*k,1);
  sp.position.set(x,y,z); sp.renderOrder=999;
  labelsGroup.add(sp); return sp;
}
function dimLineX(x1,x2,z,y,text,scale){
  var len=Math.abs(x2-x1);
  box(len,.04,.06,M.red,(x1+x2)/2,y,z,labelsGroup,false);
  box(.04,.5,.06,M.red,x1,y+.22,z,labelsGroup,false);
  box(.04,.5,.06,M.red,x2,y+.22,z,labelsGroup,false);
  makeLabel(text,(x1+x2)/2,y+1.1,z,scale||0.8,true);
}
function groundArrow(x,z,len,rotY){
  var s=new THREE.Shape(), w=.45,hw=1.0,hl=1.1;
  s.moveTo(-w/2,0);s.lineTo(-w/2,len-hl);s.lineTo(-hw/2,len-hl);
  s.lineTo(0,len);s.lineTo(hw/2,len-hl);s.lineTo(w/2,len-hl);
  s.lineTo(w/2,0);s.closePath();
  var m=new THREE.Mesh(new THREE.ShapeGeometry(s),M.arrow);
  m.rotation.x=-Math.PI/2; m.rotation.z=rotY||0;
  m.position.set(x,.07,z); scene.add(m); return m;
}

/* ნიადაგი, გზა, ეზო */
var ground=new THREE.Mesh(new THREE.PlaneGeometry(140,160),M.grass);
ground.rotation.x=-Math.PI/2; ground.position.set(9,-.02,14);
ground.receiveShadow=true; scene.add(ground);

var accessRoad=new THREE.Group(); accessRoad.name="AccessRoad"; scene.add(accessRoad);
var road=new THREE.Mesh(new THREE.PlaneGeometry(140,7),M.asphalt);
road.rotation.x=-Math.PI/2; road.position.set(9,0,-6); road.receiveShadow=true;
accessRoad.add(road);
for(var i=-60;i<70;i+=6){
  var dash=new THREE.Mesh(new THREE.PlaneGeometry(2.4,.18),M.lineWhite);
  dash.rotation.x=-Math.PI/2; dash.position.set(i,.012,-6); accessRoad.add(dash);
}
var apron=new THREE.Mesh(new THREE.PlaneGeometry(26,3.6),M.concrete);
apron.rotation.x=-Math.PI/2; apron.position.set(10.5,.005,-1);
apron.receiveShadow=true; accessRoad.add(apron);

var yard=new THREE.Mesh(new THREE.PlaneGeometry(24,26),M.concrete);
yard.rotation.x=-Math.PI/2; yard.position.set(10,.004,14);
yard.receiveShadow=true; scene.add(yard);

/* ნაკვეთის საზღვარი */
(function(){
  var pts=[new THREE.Vector3(-2,.06,-2.2),new THREE.Vector3(30,.06,-2.2),
           new THREE.Vector3(30,.06,32),new THREE.Vector3(-2,.06,32)];
  pts.push(pts[0]);
  scene.add(new THREE.Line(new THREE.BufferGeometry().setFromPoints(pts),
    new THREE.LineBasicMaterial({color:0xff3b30})));
})();
makeLabel("ნაკვეთი ს/კ 50.12.32.483 • ჩაბინაანი, ახმეტა",14,.7,30.5,1,true);

/* მეზობლის სახლი */
(function(){
  var g=new THREE.Group(); g.name="NeighborHouse";
  box(6,3.2,8,M.houseWall,-6,1.6,5,g);
  var roof=new THREE.Mesh(new THREE.ConeGeometry(5.4,2.2,4),M.houseRoof);
  roof.rotation.y=Math.PI/4; roof.scale.set(1,1,1.25);
  roof.position.set(-6,4.3,5); roof.castShadow=true; g.add(roof);
  scene.add(g);
  makeLabel("მეზობლის სახლი",-6,6.3,5,.9);
})();
/* არსებული ნაგებობა */
(function(){
  var g=new THREE.Group(); g.name="ExistingBuilding";
  box(6,3,7,M.houseWall,26,1.5,5,g);
  var roof=new THREE.Mesh(new THREE.ConeGeometry(5.2,2,4),M.houseRoof);
  roof.rotation.y=Math.PI/4; roof.scale.set(1,1,1.15);
  roof.position.set(26,4,5); roof.castShadow=true; g.add(roof);
  scene.add(g);
  makeLabel("არსებული ნაგებობა",26,6,5,.9);
})();

/* ბოქსის მშენებელი */
var WALL_T=.18;
function buildBay(o){
  var g=new THREE.Group(); g.name=o.name; scene.add(g);
  var cx=o.x0+o.width/2, height=o.height, depth=o.depth, width=o.width, x0=o.x0;
  box(width,.12,depth,M.concreteBay,cx,.06,depth/2,g);
  box(WALL_T,height,depth,M.panel,x0+WALL_T/2,height/2,depth/2,g);
  box(WALL_T,height,depth,M.panel,x0+width-WALL_T/2,height/2,depth/2,g);
  box(WALL_T+.02,.5,depth,M.panelBlue,x0+WALL_T/2,height-.6,depth/2,g,false);
  box(WALL_T+.02,.5,depth,M.panelBlue,x0+width-WALL_T/2,height-.6,depth/2,g,false);
  [0,depth/2,depth].forEach(function(zz){
    box(.18,height,.18,M.steel,x0+.12,height/2,zz,g);
    box(.18,height,.18,M.steel,x0+width-.12,height/2,zz,g);
  });
  box(width,.2,.2,M.steel,cx,height-.1,.1,g);
  box(width,.2,.2,M.steel,cx,height-.1,depth-.1,g);
  box(.2,.2,depth,M.steel,x0+.12,height-.1,depth/2,g);
  box(.2,.2,depth,M.steel,x0+width-.12,height-.1,depth/2,g);
  if(o.roofed){
    var roof=new THREE.Mesh(new THREE.BoxGeometry(width+.5,.08,depth+.7),M.roofPoly);
    roof.position.set(cx,height+.18,depth/2);
    roof.rotation.x=.045; roof.castShadow=true; g.add(roof);
    for(var i=0;i<=3;i++) box(width,.12,.12,M.steel,cx,height-.25,(depth/3)*i,g,false);
    box(.12,.05,depth-.6,ledMat(),cx-width/4,height-.4,depth/2,g,false);
    box(.12,.05,depth-.6,ledMat(),cx+width/4,height-.4,depth/2,g,false);
    addNightPoint(cx,height-.8,depth*.3,1,12);
    addNightPoint(cx,height-.8,depth*.75,1,12);
  } else {
    box(.06,.08,depth-.6,ledMat(),x0+WALL_T+.05,height-1,depth/2,g,false);
    box(.06,.08,depth-.6,ledMat(),x0+width-WALL_T-.05,height-1,depth/2,g,false);
    addNightPoint(cx,height-1.2,depth*.3,1.1,14);
    addNightPoint(cx,height-1.2,depth*.75,1.1,14);
  }
  /* DrainageChannels — ცენტრალური ტრაპი */
  var drainG=new THREE.Group(); drainG.name="DrainageChannels";
  box(.4,.05,depth-.6,M.drain,cx,.135,depth/2,drainG,false);
  for(var zz=.6; zz<depth-.4; zz+=.55) box(.42,.02,.06,M.grate,cx,.155,zz,drainG,false);
  g.add(drainG);
  /* სარეცხი ზონის მონიშვნა */
  var mz=.7;
  box(width-1.6,.015,.1,M.lineYellow,cx,.13,mz,g,false);
  box(width-1.6,.015,.1,M.lineYellow,cx,.13,depth-mz,g,false);
  box(.1,.015,depth-2*mz,M.lineYellow,cx-(width-1.6)/2,.13,depth/2,g,false);
  box(.1,.015,depth-2*mz,M.lineYellow,cx+(width-1.6)/2,.13,depth/2,g,false);
  /* ბორდიურები */
  box(.25,.18,depth-.6,M.yellow,x0+.45,.2,depth/2,g);
  box(.25,.18,depth-.6,M.yellow,x0+width-.45,.2,depth/2,g);
  /* აღჭურვილობა */
  var ex=x0+WALL_T+.32;
  box(.5,.85,.45,M.yellow,ex,.62,1.2,g);            /* კერხერი */
  box(.45,.1,.4,M.dark,ex,1.1,1.2,g);
  cyl(.05,.05,.5,M.dark,ex,1.35,1.2,g);
  cyl(.16,.16,.7,M.white,ex,1.9,2.2,g);             /* ქაფის აპარატი */
  var reel=new THREE.Mesh(new THREE.TorusGeometry(.32,.07,10,24),M.panelBlue);
  reel.position.set(ex,2.3,3.2); reel.rotation.y=Math.PI/2;
  reel.castShadow=true; g.add(reel);                 /* შლანგის გორგოლაჭი */
  cyl(.03,.03,1.1,M.dark,ex,1.6,3.2,g,8);
  box(.12,.6,.45,M.dark,ex-.18,1.6,4.2,g);           /* სამართავი პანელი */
  box(.02,.18,.3,ledMat(),ex-.1,1.72,4.2,g,false);
  makeLabel(o.labelText,cx,height+1,depth/2,1);
  return g;
}

buildBay({name:"LargeTruckBay",x0:0,width:6,depth:10,height:6,roofed:false,
  labelText:"ბოქსი 1 • დიდი მანქანები • 10m×6m×6m"});
buildBay({name:"SmallCarBay1",x0:6,width:6,depth:6,height:5,roofed:true,
  labelText:"ბოქსი 2 • 6m×6m×5m"});
buildBay({name:"SmallCarBay2",x0:12,width:6,depth:6,height:5,roofed:true,
  labelText:"ბოქსი 3 • 6m×6m×5m"});

/* შემკრები არხი + ზეთის დამჭერი */
(function(){
  var g=new THREE.Group(); g.name="DrainageChannels_Main";
  box(18,.06,.5,M.drain,9,.04,10.8,g,false);
  for(var xx=.5;xx<18;xx+=.6) box(.08,.025,.52,M.grate,xx,.07,10.8,g,false);
  box(2.2,.3,1.6,M.dark,2.2,.15,18.5,g);
  box(2,.06,1.4,M.grate,2.2,.33,18.5,g,false);
  scene.add(g);
  makeLabel("ზეთის/ტალახის დამჭერი • ფილტრაცია",2.2,1.4,18.5,.8);
  makeLabel("წყლის შემკრები არხი",9,.9,10.8,.75);
})();

/* აბრა */
(function(){
  var cnv=document.createElement("canvas"); cnv.width=1024; cnv.height=160;
  var c=cnv.getContext("2d");
  c.fillStyle="#101722"; c.fillRect(0,0,1024,160);
  c.fillStyle="#3aa0ff"; c.font="bold 84px 'Segoe UI',sans-serif"; c.textAlign="center";
  c.fillText("CAR WASH",512,78);
  c.fillStyle="#fff"; c.font="bold 50px 'Noto Sans Georgian','Segoe UI',sans-serif";
  c.fillText("ავტოსამრეცხაო",512,140);
  var tex=new THREE.CanvasTexture(cnv);
  var mat=new THREE.MeshStandardMaterial({map:tex,emissive:0xffffff,emissiveMap:tex,emissiveIntensity:.15});
  ledMaterials.push(mat);
  var sg=new THREE.Mesh(new THREE.BoxGeometry(10,1.4,.22),mat);
  sg.position.set(12,5.9,-.35); sg.castShadow=true; scene.add(sg);
  box(.15,1,.15,M.steel,7.4,5.2,-.35);
  box(.15,1,.15,M.steel,16.6,5.2,-.35);
  addNightPoint(12,5.6,-1.6,.8,10,0x9fccff);
})();

/* DryingArea */
(function(){
  var g=new THREE.Group(); g.name="DryingArea";
  var zone=new THREE.Mesh(new THREE.PlaneGeometry(11,6),
    new THREE.MeshStandardMaterial({color:0x9fb3c4,roughness:.95}));
  zone.rotation.x=-Math.PI/2; zone.position.set(12,.015,16.5);
  zone.receiveShadow=true; g.add(zone);
  [8.5,12,15.5].forEach(function(xx){
    box(2.6,.012,.1,M.lineWhite,xx,.03,13.8,g,false);
    box(2.6,.012,.1,M.lineWhite,xx,.03,19.2,g,false);
    box(.1,.012,5.4,M.lineWhite,xx-1.3,.03,16.5,g,false);
    box(.1,.012,5.4,M.lineWhite,xx+1.3,.03,16.5,g,false);
  });
  scene.add(g);
  makeLabel("გასამშრალებელი ზონა",12,1.6,16.5,.9);
})();

/* VacuumStation */
(function(){
  var g=new THREE.Group(); g.name="VacuumStation";
  cyl(.12,.12,2.4,M.steel,3.5,1.2,15,g);
  box(1.4,.08,1.4,M.panelBlue,3.5,2.5,15,g);
  box(.6,1.1,.6,M.panelBlue,3.5,.55,15,g);
  cyl(.05,.05,1.6,M.dark,4.1,1.2,15.3,g,8);
  box(.06,.05,1.2,ledMat(),3.5,2.44,15,g,false);
  addNightPoint(3.5,2.2,15,.5,7);
  scene.add(g);
  makeLabel("მტვერსასრუტის პოსტი",3.5,3.4,15,.85);
})();

/* CompressorStation */
(function(){
  var g=new THREE.Group(); g.name="CompressorStation";
  var tank=cyl(.35,.35,1.3,M.panelBlue,19.6,.75,12.5,g);
  tank.rotation.z=Math.PI/2;
  box(.5,.5,.5,M.dark,19.6,1.35,12.5,g);
  box(.35,1.2,.3,M.yellow,19.6,.6,14,g);
  scene.add(g);
  makeLabel("კომპრესორი • საბურავის წნევა",19.6,2.4,13.2,.8);
})();

/* TechnicalRoom */
(function(){
  var g=new THREE.Group(); g.name="TechnicalRoom";
  box(4,2.7,3,M.panel,2,1.35,22.5,g);
  box(4.3,.12,3.3,M.steel,2,2.78,22.5,g);
  box(.9,2,.08,M.dark,2,1,21,g);
  box(.5,.7,.1,M.dark,3.4,1.6,20.97,g);
  box(.4,.08,.06,ledMat(),3.4,1.95,20.94,g,false);
  cyl(.9,.9,2.4,new THREE.MeshStandardMaterial({color:0x4f7ea8,roughness:.5}),5.8,1.2,23,g);
  box(1,1.6,.7,M.panelBlue,.2,.8,24.6,g);
  scene.add(g);
  makeLabel("ტექნიკური ოთახი • ტუმბოები, ფილტრები, ქიმია",2,4,22.5,.8);
  makeLabel("წყლის ავზი",5.8,3,23,.75);
})();

cyl(.35,.3,.9,M.dark,16.5,.45,20.5);
cyl(.35,.3,.9,M.dark,17.4,.45,20.5);
makeLabel("ნაგვის ურნები",17,1.8,20.5,.7);

/* სამანქანე გასასვლელი */
(function(){
  var d=new THREE.Mesh(new THREE.PlaneGeometry(3.8,18),M.drive);
  d.rotation.x=-Math.PI/2; d.position.set(20.1,.01,6.5);
  d.receiveShadow=true; d.name="SideDriveway"; scene.add(d);
  groundArrow(20.1,.5,5,0);
  groundArrow(20.1,8,5,0);
})();

/* მოძრაობის ისრები */
groundArrow(3,-2.3,2.6,0); groundArrow(9,-2.3,2.6,0); groundArrow(15,-2.3,2.6,0);
groundArrow(3,10.6,2.4,0); groundArrow(9,10.6,2.4,0); groundArrow(15,10.6,2.4,0);
groundArrow(12,20,2.4,Math.PI/2);

/* ზომების ხაზები */
dimLineX(-3,0,5.5,.1,"3m დაშორება",.8);
dimLineX(18.2,22,1.5,.1,"3.8m გასასვლელი",.8);
makeLabel("შესასვლელი გზიდან ⟶ თითო ბოქსში",10.5,.9,-3.6,.85,true);
makeLabel("მანევრის ზონა",10,.8,12.5,.8);

/* მანქანები მასშტაბისთვის */
function makeCar(color,x,z,rotY){
  var g=new THREE.Group();
  var body=new THREE.MeshStandardMaterial({color:color,roughness:.35,metalness:.5});
  box(1.8,.55,4.2,body,0,.62,0,g);
  box(1.6,.5,2.2,M.glassDark,0,1.12,-.1,g);
  [[-0.85,1.35],[0.85,1.35],[-0.85,-1.35],[0.85,-1.35]].forEach(function(p){
    var w=cyl(.33,.33,.25,M.dark,p[0],.33,p[1],g,16); w.rotation.z=Math.PI/2;
  });
  g.position.set(x,.12,z); g.rotation.y=rotY||0; scene.add(g); return g;
}
function makeTruck(x,z){
  var g=new THREE.Group(); g.name="ScaleTruck";
  box(2.3,2,1.9,new THREE.MeshStandardMaterial({color:0x2f6fb0,roughness:.4,metalness:.4}),0,1.45,-3,g);
  box(2.1,.7,.1,M.glassDark,0,1.85,-3.93,g);
  box(2.45,2.6,5.6,M.white,0,1.95,.6,g);
  box(2.3,.3,7.6,M.dark,0,.6,-.2,g);
  [[-1.05,-3],[1.05,-3],[-1.05,1.4],[1.05,1.4],[-1.05,2.6],[1.05,2.6]].forEach(function(p){
    var w=cyl(.52,.52,.35,M.dark,p[0],.52,p[1],g,16); w.rotation.z=Math.PI/2;
  });
  g.position.set(x,.12,z); scene.add(g); return g;
}
makeTruck(3,5);
makeCar(0xb8412f,9,3);
makeCar(0x3b4757,12,16.5,0);

/* კამერა და მართვა */
var ctrl={target:new THREE.Vector3(10,2,7), sph:new THREE.Spherical(38,1.05,-2.55)};
function applyCam(){
  var off=new THREE.Vector3().setFromSpherical(ctrl.sph);
  camera.position.copy(ctrl.target).add(off);
  camera.lookAt(ctrl.target);
}
var VIEWS={
  iso:{pos:[34,24,-18],tgt:[10,1,9]},
  top:{pos:[10,58,8.2],tgt:[10,0,8]},
  front:{pos:[11,4.5,-22],tgt:[10,3,4]},
  side:{pos:[44,7,6],tgt:[6,3,6]},
  interior:{pos:[9,1.7,5.4],tgt:[9,1.9,-3]},
  truck:{pos:[5.5,3.2,-9],tgt:[3,2.8,6]}
};
function setView(key){
  var v=VIEWS[key]; if(!v) return;
  ctrl.target.set(v.tgt[0],v.tgt[1],v.tgt[2]);
  var off=new THREE.Vector3(v.pos[0],v.pos[1],v.pos[2]).sub(ctrl.target);
  ctrl.sph.setFromVector3(off);
  applyCam();
}
setView("iso");

var dom=renderer.domElement, pointers=new Map(), pinchDist=0;
dom.addEventListener("pointerdown",function(e){
  pointers.set(e.pointerId,{x:e.clientX,y:e.clientY});
  dom.setPointerCapture(e.pointerId);
  if(pointers.size===2){
    var a=[],it=pointers.values(),v;while(!(v=it.next()).done)a.push(v.value);
    pinchDist=Math.hypot(a[0].x-a[1].x,a[0].y-a[1].y);
  }
});
dom.addEventListener("pointermove",function(e){
  if(!pointers.has(e.pointerId)) return;
  var prev=pointers.get(e.pointerId);
  var dx=e.clientX-prev.x, dy=e.clientY-prev.y;
  pointers.set(e.pointerId,{x:e.clientX,y:e.clientY});
  if(pointers.size===1){
    ctrl.sph.theta-=dx*.005;
    ctrl.sph.phi=Math.max(.08,Math.min(1.52,ctrl.sph.phi-dy*.005));
    applyCam();
  } else if(pointers.size===2){
    var a=[],it=pointers.values(),v;while(!(v=it.next()).done)a.push(v.value);
    var d=Math.hypot(a[0].x-a[1].x,a[0].y-a[1].y);
    if(pinchDist>0){
      ctrl.sph.radius=Math.max(3,Math.min(130,ctrl.sph.radius*(pinchDist/d)));
      applyCam();
    }
    pinchDist=d;
  }
});
function onUp(e){ pointers.delete(e.pointerId); if(pointers.size<2) pinchDist=0; }
dom.addEventListener("pointerup",onUp);
dom.addEventListener("pointercancel",onUp);
dom.addEventListener("wheel",function(e){
  e.preventDefault();
  ctrl.sph.radius=Math.max(3,Math.min(130,ctrl.sph.radius*(1+e.deltaY*.0012)));
  applyCam();
},{passive:false});

/* დღე/ღამე */
var night=false;
function setNightMode(on){
  night=on;
  scene.background=(on?NIGHT_BG:DAY_BG).clone();
  scene.fog.color=scene.background.clone();
  hemi.intensity=on?.18:.85;
  sun.intensity=on?.06:1.05;
  nightLights.forEach(function(p){p.intensity=on?p.userData.ni:0;});
  ledMaterials.forEach(function(m){m.emissiveIntensity=on?1.6:.15;});
}

/* UI */
var bar=document.getElementById("bar");
bar.querySelectorAll("button[data-v]").forEach(function(b){
  b.addEventListener("click",function(){
    bar.querySelectorAll("button[data-v]").forEach(function(x){x.classList.remove("on");});
    b.classList.add("on");
    setView(b.getAttribute("data-v"));
  });
});
var nightBtn=document.getElementById("nightBtn");
nightBtn.addEventListener("click",function(){
  setNightMode(!night);
  nightBtn.classList.toggle("on",night);
  nightBtn.textContent=night?"🌙 ღამე ✓":"🌙 ღამის ხედი";
});
var labelsBtn=document.getElementById("labelsBtn");
labelsBtn.addEventListener("click",function(){
  labelsGroup.visible=!labelsGroup.visible;
  labelsBtn.classList.toggle("on",labelsGroup.visible);
  labelsBtn.textContent=labelsGroup.visible?"🏷 ზომები ✓":"🏷 ზომები";
});

/* ზომა და რენდერი */
function resize(){
  var w=window.innerWidth,h=window.innerHeight;
  renderer.setSize(w,h);
  camera.aspect=w/h;
  camera.updateProjectionMatrix();
}
resize();
window.addEventListener("resize",resize);
(function loop(){
  requestAnimationFrame(loop);
  renderer.render(scene,camera);
})();
})();
</script>
</body>
</html>
