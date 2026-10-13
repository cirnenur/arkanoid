# arkanoid
<!DOCTYPE html>
<html lang="es"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Arkanoid</title>
<style>
:root{--bg:#0b1020;--fg:#e8ecff;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
html,body{margin:0;height:100%;background:var(--bg);color:var(--fg);font-family:system-ui,-apple-system,"Segoe UI",sans-serif;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:8px;user-select:none;-webkit-user-select:none;touch-action:none;overflow:hidden}
canvas{max-height:90vh;max-width:96vw;aspect-ratio:3/4;border-radius:12px;box-shadow:0 0 40px #3b82f655}
small{opacity:.65}
</style></head><body>
<canvas id="c" width="480" height="640"></canvas>
<small>Ratón, dedo o flechas ◀ ▶ · Espacio o clic para lanzar · T = Tienda</small>
<script>
const cv=document.getElementById('c'),x=cv.getContext('2d'),W=480,H=640,PY=596,BW=44,BH=20,DUR=1200;
const COLS=['#ef4444','#f97316','#eab308','#22c55e','#06b6d4','#6366f1','#a855f7','#ec4899'];
const PU={W:['#22c55e','Ancha'],S:['#3b82f6','Lenta'],L:['#ef4444','Vida'],M:['#eab308','Multi'],F:['#f97316','Fuego'],I:['#a855f7','Imán'],Z:['#ec4899','Láser'],E:['#06b6d4','Escudo'],B:['#fb7185','Bomba'],D:['#facc15','x2 Puntos'],X:['#c026d3','Súper láser']};
const SH=[{k:'F',d:'La bola atraviesa los ladrillos',c:800,r:0},{k:'I',d:'La bola se pega a la pala; la lanzas tú',c:1000,r:0},{k:'Z',d:'La pala dispara láseres sola',c:1400,r:0},{k:'E',d:'Una red salva tu bola una vez',c:1200,r:0},{k:'B',d:'La bola explota y rompe los ladrillos de al lado',c:2200,r:2},{k:'D',d:'Todos los puntos valen el doble',c:2800,r:4},{k:'X',d:'Triple láser potente, rompe más rápido',c:3500,r:6}];
let px=W/2,pw=84,balls,bricks,ups,parts,buls,lives,score,level,st,prev='ready',best=0,bank=0,U={},keys={},msg='',msg2='',maxCl=0,fr=0,A={W:0,S:0,F:0,I:0,Z:0,E:0,B:0,D:0,X:0};
try{best=+localStorage.getItem('ark')||0;bank=+localStorage.getItem('arkb')||0;U=JSON.parse(localStorage.getItem('aku')||'{}');maxCl=+localStorage.getItem('arkl')||0}catch(e){}
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v)),sp=()=>Math.min(5.6,3.5+level*.25)*(A.S>0?.72:1);
function sv(){try{localStorage.setItem('arkb',bank);localStorage.setItem('aku',JSON.stringify(U));localStorage.setItem('arkl',maxCl)}catch(e){}}
function add(n){if(A.D>0)n*=2;score+=n;bank+=n;sv()}
function layout(l){const g=[];for(let r=0;r<8;r++){g[r]=[];for(let c=0;c<10;c++){let h=0;
 if(l==0)h=r<5?(r<2?2:1):0;
 else if(l==1)h=r<6&&(r+c)%2==0?(c%3==0?2:1):0;
 else if(l==2)h=r<6&&c>=r&&c<=9-r?(r<2?2:1):0;
 else h=r<7&&(r==0||r==6||c==0||c==9)?2:(r<7&&(r+c)%2?1:0);
 g[r][c]=h}}return g}
function build(){bricks=[];layout(level%4).forEach((row,r)=>row.forEach((h,c)=>{if(h)bricks.push({x:20+c*(BW+2),y:70+r*(BH+2),r,hp:h,mx:h,f:-99})}));
 balls=[{x:px,y:PY-9,vx:0,vy:0}];ups=[];parts=[];buls=[];for(const k in A)A[k]=0;st='ready'}
function newGame(){lives=3;score=0;level=0;build()}
function rel(b){const a=clamp(b.stuck/(pw/2),-1,1)*1.1,s=sp();b.vx=s*Math.sin(a);b.vy=-s*Math.cos(a);b.stuck=null}
function launch(){if(st=='over'){newGame();return}if(st=='ready'){st='play';const b=balls[0],s=sp();b.vx=s*.35;b.vy=-s*.94}else if(st=='play')balls.forEach(b=>{if(b.stuck!=null)rel(b)})}
function shop(){if(st=='shop')st=prev;else if(st=='ready'||st=='over'){prev=st;st='shop'}}
function pt(e){const r=cv.getBoundingClientRect();return[(e.clientX-r.left)/r.width*W,(e.clientY-r.top)/r.height*H]}
cv.addEventListener('pointerdown',e=>{const[cx,cy]=pt(e);
 if(st=='shop'){if(cy>584){shop();return}SH.forEach((s,i)=>{const y=106+i*66;if(cy>y&&cy<y+58&&!U[s.k]&&maxCl>=s.r&&bank>=s.c){bank-=s.c;U[s.k]=1;sv()}});return}
 if((st=='ready'||st=='over')&&cx>W/2-80&&cx<W/2+80&&cy>(st=='over'?400:450)&&cy<(st=='over'?440:490)){shop();return}
 launch();mv(e)});
function mv(e){if(st=='shop')return;px=clamp(pt(e)[0],pw/2,W-pw/2)}
addEventListener('pointermove',mv);
addEventListener('keydown',e=>{if(['ArrowLeft','ArrowRight',' '].includes(e.key))e.preventDefault();keys[e.key]=1;if(e.key==' '&&st!='shop')launch();if(e.key=='t'||e.key=='T'||(e.key=='Escape'&&st=='shop'))shop()});
addEventListener('keyup',e=>keys[e.key]=0);
function boom(b){for(let i=0;i<10;i++)parts.push({x:b.x+BW/2,y:b.y+BH/2,vx:(Math.random()-.5)*5,vy:(Math.random()-.5)*5,l:30,c:COLS[b.r]})}
function hitBrick(b){b.hp--;add(10);if(b.hp<=0){boom(b);bricks.splice(bricks.indexOf(b),1);add(20);
 if(Math.random()<.22){const pool='WSLM'+Object.keys(U).join('');ups.push({x:b.x+BW/2,y:b.y,k:pool[Math.random()*pool.length|0]})}
 if(!bricks.length){level++;lives=Math.max(3,lives);msg='NIVEL '+(level+1);msg2='♥ Vidas restauradas';if(level>maxCl){maxCl=level;sv();if(level%2==0&&level<=6)msg2+=' · ¡Nueva habilidad en la tienda!'}build();setTimeout(()=>{msg='';msg2=''},3200)}}}
function stepBall(b){if(b.stuck!=null){b.x=clamp(px+b.stuck,7,W-7);b.y=PY-9;if(A.I<=0)rel(b);return}
 const s=sp(),m=Math.hypot(b.vx,b.vy)||1;b.vx*=s/m;b.vy*=s/m;
 for(let i=0;i<3;i++){b.x+=b.vx/3;b.y+=b.vy/3;
  if(b.x<7){b.x=7;b.vx=Math.abs(b.vx)}if(b.x>W-7){b.x=W-7;b.vx=-Math.abs(b.vx)}if(b.y<7){b.y=7;b.vy=Math.abs(b.vy)}
  if(A.E>0&&b.vy>0&&b.y>PY+22){b.vy=-Math.abs(b.vy);A.E=0}
  if(b.vy>0&&b.y+7>=PY&&b.y-7<=PY+12&&b.x>px-pw/2-6&&b.x<px+pw/2+6){
   if(A.I>0){b.stuck=b.x-px;b.vx=b.vy=0;b.y=PY-9;return}
   const a=clamp((b.x-px)/(pw/2),-1,1)*1.1;b.vx=s*Math.sin(a);b.vy=-s*Math.cos(a);b.y=PY-7}
  for(const k of bricks){const cx=clamp(b.x,k.x,k.x+BW),cy=clamp(b.y,k.y,k.y+BH),dx=b.x-cx,dy=b.y-cy;
   if(dx*dx+dy*dy<49){if(A.F>0){if(fr-k.f>10){k.f=fr;hitBrick(k)}continue}
    if(Math.abs(dx)>Math.abs(dy)){b.vx=dx>0?Math.abs(b.vx):-Math.abs(b.vx);b.x+=dx>0?1:-1}else{b.vy=dy>=0?Math.abs(b.vy):-Math.abs(b.vy);b.y+=dy>=0?1:-1}hitBrick(k);if(A.B>0)bricks.filter(o=>o!==k&&Math.abs(o.x-k.x)<=BW+4&&Math.abs(o.y-k.y)<=BH+4).forEach(o=>{if(bricks.includes(o))hitBrick(o)});break}}}}
function step(){fr++;const d=(keys.ArrowRight?1:0)-(keys.ArrowLeft?1:0);pw=A.W>0?124:84;if(d)px=clamp(px+d*9,pw/2,W-pw/2);px=clamp(px,pw/2,W-pw/2);
 if(st=='ready')balls[0].x=px,balls[0].y=PY-9;
 if(st=='play'){for(const k in A)if(A[k]>0)A[k]--;
  if(A.Z>0&&fr%16==0)buls.push({x:px-pw/2+8,y:PY},{x:px+pw/2-8,y:PY});if(A.X>0&&fr%9==0)buls.push({x:px-pw/2+8,y:PY,p:2},{x:px,y:PY,p:2},{x:px+pw/2-8,y:PY,p:2});
  buls.forEach(u=>{u.y-=9;const k=bricks.find(k=>u.x>k.x&&u.x<k.x+BW&&u.y<k.y+BH&&u.y>k.y-6);if(k){u.dead=1;hitBrick(k);if(u.p&&bricks.includes(k))hitBrick(k)}});buls=buls.filter(u=>!u.dead&&u.y>0);
  balls.forEach(stepBall);balls=balls.filter(b=>b.y<H+10);
  if(!balls.length){lives--;for(const k in A)A[k]=0;if(lives<=0){st='over';if(score>best){best=score;try{localStorage.setItem('ark',best)}catch(e){}}}else{balls=[{x:px,y:PY-9,vx:0,vy:0}];st='ready'}}}
 ups.forEach(u=>{u.y+=2.6;if(u.y>PY-8&&u.y<PY+14&&Math.abs(u.x-px)<pw/2+14){u.dead=1;add(50);
  if('WSFIZEBDX'.includes(u.k))A[u.k]=DUR;if(u.k=='L')lives=Math.min(5,lives+1);
  if(u.k=='M'&&st=='play'){const b=balls.find(b=>b.stuck==null)||balls[0];[-.5,.5].forEach(a=>balls.push({x:b.x,y:b.y,vx:b.vx*Math.cos(a)-b.vy*Math.sin(a),vy:b.vx*Math.sin(a)+b.vy*Math.cos(a)}))}}});
 ups=ups.filter(u=>!u.dead&&u.y<H+20);parts.forEach(p=>{p.x+=p.vx;p.y+=p.vy;p.vy+=.15;p.l--});parts=parts.filter(p=>p.l>0)}
function rr(a,b,w,h,r){x.beginPath();x.roundRect(a,b,w,h,r)}
function T(t,px_,py_,f,c,al='center'){x.fillStyle=c||'#fff';x.font=f;x.textAlign=al;x.fillText(t,px_,py_)}
function drawShop(){x.fillStyle='#0b1020f2';x.fillRect(0,0,W,H);T('TIENDA',W/2,70,'bold 36px system-ui');T('Banco: '+bank+' puntos',W/2,105,'18px system-ui','#facc15');
 SH.forEach((s,i)=>{const y=150+i*100,own=U[s.k],ok=bank>=s.c;x.fillStyle=own?'#14532d':'#1e293b';rr(30,y,420,86,12);x.fill();
  x.fillStyle=PU[s.k][0];rr(46,y+26,40,34,17);x.fill();T(s.k,66,y+49,'bold 18px system-ui');
  T(PU[s.k][1],104,y+34,'bold 22px system-ui','#fff','left');T(s.d,104,y+58,'13px system-ui','#cbd5e1','left');
  T(own?'✔ Tuya':s.c+' pts',436,y+48,'bold 17px system-ui',own?'#4ade80':ok?'#facc15':'#64748b','right')});
 T('Toca un objeto para comprarlo. Al comprarlo, empezará a caer al romper ladrillos y durará 20 s.',W/2,562,'12px system-ui','#94a3b8');
 x.fillStyle='#3b82f6';rr(W/2-70,580,140,40,10);x.fill();T('Volver',W/2,606,'bold 18px system-ui')}
function draw(){const g=x.createLinearGradient(0,0,0,H);g.addColorStop(0,'#0b1020');g.addColorStop(1,'#1e1b4b');x.fillStyle=g;x.fillRect(0,0,W,H);
 x.fillStyle='#ffffff10';for(let i=0;i<40;i++)x.fillRect((i*97)%W,(i*53)%H,2,2);
 if(A.E>0){x.fillStyle='#22d3ee';x.shadowColor='#22d3ee';x.shadowBlur=12;x.fillRect(0,PY+26,W,4);x.shadowBlur=0}
 bricks.forEach(k=>{x.fillStyle=k.mx==2&&k.hp==2?'#cbd5e1':COLS[k.r];rr(k.x,k.y,BW,BH,4);x.fill();x.fillStyle='#fff4';rr(k.x+2,k.y+2,BW-4,6,3);x.fill();
  if(k.mx==2){x.strokeStyle=k.hp==2?'#64748b':'#fff8';x.lineWidth=2;rr(k.x+1,k.y+1,BW-2,BH-2,4);x.stroke()}});
 parts.forEach(p=>{x.globalAlpha=p.l/30;x.fillStyle=p.c;x.fillRect(p.x,p.y,4,4)});x.globalAlpha=1;
 x.fillStyle='#f472b6';buls.forEach(u=>x.fillRect(u.x-1.5,u.y-10,3,12));
 ups.forEach(u=>{x.fillStyle=PU[u.k][0];rr(u.x-18,u.y-9,36,18,9);x.fill();T(u.k,u.x,u.y+4,'bold 11px system-ui')});
 const pg=x.createLinearGradient(0,PY,0,PY+12);pg.addColorStop(0,'#67e8f9');pg.addColorStop(1,'#0e7490');x.fillStyle=pg;x.shadowColor=A.Z>0?'#ec4899':'#22d3ee';x.shadowBlur=14;rr(px-pw/2,PY,pw,12,6);x.fill();x.shadowBlur=0;
 balls.forEach(b=>{x.shadowColor=A.F>0?'#f97316':A.B>0?'#fb7185':'#fff';x.shadowBlur=14;x.fillStyle=A.F>0?'#fdba74':A.B>0?'#fecdd3':'#fff';x.beginPath();x.arc(b.x,b.y,7,0,7);x.fill();x.shadowBlur=0});
 T('Puntos '+score,14,28,'bold 18px system-ui','#e8ecff','left');T('Nivel '+(level+1),W/2,28,'bold 18px system-ui');T('♥'.repeat(lives),W-14,28,'bold 18px system-ui','#e8ecff','right');
 T('Banco '+bank,14,46,'12px system-ui','#facc15','left');
 let n=0;for(const k in A)if(A[k]>0)T(PU[k][1]+' '+Math.ceil(A[k]/60)+'s',14,62+14*n++,'bold 12px system-ui',PU[k][0],'left');
 const btn=y=>{x.fillStyle='#334155';rr(W/2-80,y,160,40,10);x.fill();T('🛒 Tienda (T)',W/2,y+26,'bold 16px system-ui')};
 if(st=='over'){x.fillStyle='#000a';x.fillRect(0,0,W,H);T('GAME OVER',W/2,300,'bold 40px system-ui');T('Puntos: '+score+'  ·  Récord: '+best,W/2,340,'20px system-ui');T('Espacio o toca para jugar otra vez',W/2,380,'18px system-ui');btn(400)}
 else if(st=='ready'||msg){T(msg||'¿Listo?',W/2,400,'bold 28px system-ui');T(msg2,W/2,525,'bold 15px system-ui','#facc15');T('Espacio o clic para lanzar',W/2,430,'15px system-ui','#cbd5e1');if(st=='ready')btn(450)}
 if(st=='shop')drawShop2()}
function drawShop2(){x.fillStyle='#0b1020f5';x.fillRect(0,0,W,H);T('TIENDA',W/2,52,'bold 32px system-ui');T('Banco: '+bank+' pts  ·  Niveles superados: '+maxCl,W/2,86,'16px system-ui','#facc15');
 SH.forEach((s,i)=>{const y=106+i*66,own=U[s.k],lock=maxCl<s.r,ok=bank>=s.c;x.fillStyle=own?'#14532d':'#1e293b';rr(24,y,432,58,10);x.fill();
  x.globalAlpha=lock?.4:1;x.fillStyle=PU[s.k][0];rr(36,y+12,34,34,17);x.fill();T(s.k,53,y+35,'bold 16px system-ui');
  T(PU[s.k][1],82,y+25,'bold 18px system-ui','#fff','left');T(s.d,82,y+45,'12px system-ui','#cbd5e1','left');x.globalAlpha=1;
  T(own?'✔ Tuya':lock?'🔒 Nivel '+s.r:s.c+' pts',444,y+34,'bold 15px system-ui',own?'#4ade80':lock?'#94a3b8':ok?'#facc15':'#64748b','right')});
 T('Toca para comprar. Al comprarla cae al romper ladrillos y dura 20 s.',W/2,574,'12px system-ui','#94a3b8');
 x.fillStyle='#3b82f6';rr(W/2-70,586,140,40,10);x.fill();T('Volver',W/2,612,'bold 18px system-ui')}
function loop(){if(st!='shop')step();draw();requestAnimationFrame(loop)}
window.focus();newGame();loop();
</script></body></html>
