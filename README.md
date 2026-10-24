<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Dead Signal</title>
<style>
:root{box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html,body{margin:0;background:radial-gradient(ellipse at 50% 0,#1b2a72 0,#0a1030 55%,#050816 100%) fixed;min-height:100%;color:#e8ecff;font-family:system-ui,sans-serif;touch-action:none;user-select:none}
#app{width:min(calc(100vw - 12px),calc((100vh - 110px) * 1.6667));width:min(calc(100vw - 12px),calc((100dvh - 110px) * 1.6667));margin:0 auto;padding:6px}
canvas{width:100%;height:auto;display:block;border-radius:10px;background:#0b1020;border:1px solid rgba(111,181,255,.45);box-shadow:0 0 24px rgba(61,200,255,.15)}
#pad{display:flex;gap:6px;margin-top:8px}
#pad div{display:flex;gap:6px;flex:1}
button{flex:1;padding:16px 4px;font-size:15px;font-weight:700;border:0;border-radius:10px;background:#2a3570;color:#fff}
button:active{background:#ffd23f;color:#000}
button.off{opacity:.25}
</style>
</head>
<body>
<div id="app">
<canvas id="c" width="800" height="480"></canvas>
<div id="pad">
<div><button data-h="btnL">◀</button><button data-h="btnR">▶</button></div>
<div><button data-a="jump">Jump</button><button data-a="g" id="bg">G flip</button><button data-a="x" id="bx">X world</button><button data-a="f" id="bf">F freeze</button></div>
</div></div>

<script>
const T=40,C=20,R=12,cv=document.getElementById('c'),ctx=cv.getContext('2d');
const B='#'.repeat(20);
const L=(n,p,h,o)=>({n,p,h,m:Array.from({length:12},(_,r)=>r%11?'#'+(o[r]||'.'.repeat(18))+'#':B)});
const S=n=>'^'.repeat(n);
const LB=(n,p,h,...a)=>{const o={};for(let r=1;r<=10;r++)o[r]='.'.repeat(18);a.forEach(([r,c,t])=>{const w=[...o[r]];[...t].forEach((ch,i)=>w[c-1+i]=ch);o[r]=w.join('')});return L(n,p,h,o)};
// Tiles: # wall  = floating platform  ^ spike  A/D AC/DC platform  m/v moving platform  L/l laser (down/up)  u jump pad  o spark  x spark turn  K chip  P start  E exit
const LV=[
L('Boot Sector','','Grab the chip, avoid spikes, reach the exit.',{8:'..........K.......',9:'.........###......',10:'.P.....^^.....^^.E'}),
L('Gravity Glitch','g','Press G to flip gravity and walk on the ceiling.',{1:'........K.....^^^.',10:'.P...^^^^^^...K.E.'}),
L('AC / DC Split','x','Press X to switch worlds. Blue = AC, orange = DC.',{8:'...........K......',9:'.....AA.DD.AA.DD..',10:'.P..^^^^^^^^^^^^.E'}),
LB('Overload Tunnel','f','A low tunnel full of sparks and spikes. Freeze first, then run and hop.',[10,2,'P'],[7,6,'#'.repeat(12)],[10,6,'x'],[8,6,'x'],[10,7,'o'],[8,12,'o'],[10,9,'^'],[10,11,'K'],[10,12,'^'],[10,15,'^'],[10,16,'K'],[10,17,'x'],[8,17,'x'],[10,18,'E']),
L('Frozen Steps','','Ice is slippery. Hop the floating platforms.',{6:'..............K...',7:'.............===..',9:'.........===......',10:'.P...^^...^^...K.E'}),
L('Laser Lane','f','Lasers warn, then fire. Time it, or freeze them.',{1:'.....L....L....L..',10:'.P......K...^^K..E'}),
L('Ice Elevators','','Ride the moving platforms across the spikes.',{7:'.........K........',9:'......m....m......',10:'.P..^^^^^^^^^^.E..'}),
LB('Ice Cathedral','g','Ice makes you slide. Flip between floor and ceiling to dodge the spikes.',[10,2,'P'],[10,5,S(5)],[10,12,'K'],[10,15,S(2)],[10,18,'E'],[1,7,'K'],[1,10,S(5)],[1,16,'K']),
L('Toxic Bounce','','Low gravity. Green pads launch you high.',{5:'........K.........',6:'.......===........',10:'.P..u^^^^^^^^^..KE'}),
L('Slime Lasers','','Three lasers, three rhythms. Grab every chip.',{1:'....L....L....L...',10:'.P...K^^..K^^.K..E'}),
L('Sludge Patrol','f','Freeze the sparks and dash through the tunnel.',{9:'.....########.....',10:'.P..x.o...oK.x^^.E'}),
LB('Slime Gauntlet','','Low gravity lets you cross the gaps. Move between the laser blasts.',[10,2,'P'],[10,4,S(14)],[10,18,'E'],[9,5,'=='],[9,9,'=='],[9,13,'=='],[8,6,'K'],[8,10,'K'],[8,14,'K'],[1,7,'L'],[1,11,'L'],[1,16,'L']),
L('Stepping Stones','','Headwind pushes you back in mid-air. Mind the lasers.',{1:'.......L....L.....',8:'..........K.......',9:'.....==..==..==...',10:'.P..^^^^^^^^^^^.KE'}),
L('Spark Storm','f','A spark patrols above the lava spikes. Press F to freeze it, then hop across.',{9:'...xo......x......',10:'.P...^.K.^.K.^.K.E'}),
L('Scorched Switch','x','Hop between AC and DC platforms under the lasers.',{1:'......L.....L.....',8:'...........K......',9:'....AA.DD.AA.DD...',10:'.P..^^^^^^^^^^^^.E'}),
LB('Magma Gauntlet','f','Ride the lifts under lasers and sparks. The headwind pushes you back!',[10,2,'P'],[10,4,S(13)],[10,18,'E'],[8,5,'v'],[8,9,'v'],[8,13,'v'],[7,6,'K'],[7,10,'K'],[7,14,'K'],[5,3,'x'],[5,5,'o'],[5,16,'x'],[1,8,'L'],[1,12,'L']),
L('Void Walk','g','Floor spikes, ceiling spikes. Flip to dodge both.',{1:'.....K....^^^^....',10:'.P..^^^^...K..l.E.'}),
L('Phase Patrol','xf','Switch worlds and freeze the sparks while you cross.',{7:'..........K.......',8:'...x.o.......o..x.',9:'....AA.DD.AA.DD...',10:'.P..^^^^^^^^^^^^.E'}),
L('Rift Runner','f','Hop the spikes between lasers. Freeze to steady them.',{1:'.........L.....L..',10:'.P.^^.l.K^^.l.K^^E'}),
LB('Final Glitch','gxf','All three powers. Two ceiling chips, two sparks, a laser and phase platforms.',[10,2,'P'],[10,5,S(12)],[10,17,'K'],[10,18,'E'],[9,5,'AA'],[9,8,'DD'],[9,11,'AA'],[9,14,'DD'],[1,8,'K'],[1,12,'L'],[1,14,'K'],[1,16,S(2)],[8,4,'x'],[8,6,'o'],[8,17,'x'],[7,4,'x'],[7,13,'o'],[7,17,'x'])
];
const TH=[
{n:'World 1: Circuit Board',bg:'#0f2a5c',b2:'#071230',fl:[80,160,255],wl:'#2a56b8',ed:'#6fb5ff',ac:'#3dc8ff',gr:.55,jv:11},
{n:'World 2: Cryo Server (ice)',bg:'#0d4466',b2:'#06213a',fl:[220,245,255],wl:'#66b2dd',ed:'#eaf8ff',ac:'#bdeaff',gr:.55,jv:11,ice:1,dy:.6},
{n:'World 3: Toxic Swamp (low gravity)',bg:'#17502a',b2:'#08200f',fl:[120,255,100],wl:'#3a8a45',ed:'#9dff7a',ac:'#a6ff6e',gr:.38,jv:9.6,dy:-.5},
{n:'World 4: Magma Core (headwind)',bg:'#5a1608',b2:'#240804',fl:[255,110,30],wl:'#a03c1a',ed:'#ffb04d',ac:'#ffb347',gr:.55,jv:11,wind:-.7,dy:-.9},
{n:'World 5: The Void',bg:'#34125c',b2:'#0e0620',fl:[255,100,220],wl:'#6a34b8',ed:'#ff8be6',ac:'#ff6bd6',gr:.55,jv:11,gl:1}];
// ---- STORY: pages of {w: speaker, t: lines}. Edit the words freely! ----
const STORY={
0:[
{w:'SYSTEM',t:['ALERT: Signal lost. The GlitchVerse is breaking apart.','A virus called NOISE is scrambling every circuit.']},
{w:'SYSTEM',t:['You are VOLT, the last repair bot still online.','Five worlds. Twenty memory chips. One Core to restore.']},
{w:'VOLT',t:['Noise made the rules unstable, so I will use them.','Polarity flips gravity. Phase swaps AC and DC.','Clock Stop freezes time. Time to fix this world.']}],
4:[
{w:'SYSTEM',t:['World 2: Cryo Server. The cooling system has failed.','Everything is frozen, and the floors are slippery.']},
{w:'NOISE',t:['Slide all you like, little bot.','The cold will stop your clock before you reach the Core.']}],
8:[
{w:'SYSTEM',t:['World 3: Toxic Swamp. Coolant has leaked everywhere.','Gravity is weak here. Green pads will launch you high.']},
{w:'VOLT',t:['Low gravity feels like floating... but lasers still hurt.','Stay sharp. Four more chips to go in this world.']}],
12:[
{w:'SYSTEM',t:['World 4: Magma Core. The processor is overheating.','A headwind pushes you back whenever you are in the air.']},
{w:'NOISE',t:['Feel that heat, Volt? I am closer than you think.','Every chip you grab only makes me angrier.']}],
16:[
{w:'SYSTEM',t:['World 5: The Void. Noise lives here, in the dead space.','Spikes above, spikes below, and glitches everywhere.']},
{w:'NOISE',t:['You fixed their little world, bot. Now meet what broke it.']},
{w:'VOLT',t:['Last four chips. Let us finish this.']}],
end:[
{w:'SYSTEM',t:['Final chip restored. Power returns to the Core.']},
{w:'NOISE',t:['No... my static... everything is going quiet...']},
{w:'VOLT',t:['A clean signal always beats noise.']},
{w:'SYSTEM',t:['The GlitchVerse is restored. Thank you, Volt.']}]
};
// ---- Animations: explosion on death, confetti on level clear, heart helper ----
const fx=[],rings=[],tr=[],pl=[];let sq=0,face=1,wasG=true,shake=0,hp=-1,hpT=0,dyT=0,clT=0;
function burst(x,y,c,n,v){for(let i=0;i<n;i++){const a=Math.random()*6.283,sp2=(.4+Math.random())*v;fx.push({x,y,vx:Math.cos(a)*sp2,vy:Math.sin(a)*sp2-1,g:.25,l:40+Math.random()*20,c,s:3+Math.random()*4})}}
function confetti(x,y){const cs=['#ffd23f','#3ddc97','#ff5d8f','#3dc8ff','#b57cff','#ff9a3d'];for(let i=0;i<70;i++){const a=-1.57+(Math.random()-.5)*2.6,sp2=3+Math.random()*7;fx.push({x,y,vx:Math.cos(a)*sp2,vy:Math.sin(a)*sp2,g:.22,l:90+Math.random()*60,c:cs[i%6],s:4+Math.random()*5})}}
function dust(x,y,d,n){for(let i=0;i<n;i++)fx.push({x:x+(Math.random()-.5)*18,y,vx:(Math.random()-.5)*3.2,vy:d*Math.random()*1.4,g:.02,l:16+Math.random()*12,c:'#dfe6ff',s:2+Math.random()*3})}
function stepFx(){sq*=.82;if(Math.abs(sq)<.01)sq=0;
  for(let i=tr.length-1;i>=0;i--){tr[i].a-=.045;if(tr[i].a<=0)tr.splice(i,1)}
  for(let i=rings.length-1;i>=0;i--){const q=rings[i];q.r+=2;q.a-=.05;if(q.a<=0)rings.splice(i,1)}
  for(let i=pl.length-1;i>=0;i--){const q=pl[i];q.r+=q.dr;q.a-=q.da;if(q.a<=0)pl.splice(i,1)}
  for(let i=fx.length-1;i>=0;i--){const q=fx[i],dd=q.d||.99;q.x+=q.vx;q.y+=q.vy;q.vy+=q.g;q.vx*=dd;if(q.d)q.vy*=dd;if(--q.l<=0)fx.splice(i,1)}if(shake>0)shake--;if(hpT>0)hpT--}
function drawFx(){for(const q of rings){ctx.globalAlpha=q.a;ctx.strokeStyle='#ffffff';ctx.lineWidth=1;ctx.beginPath();ctx.ellipse(q.x,q.y,q.r,q.r*.3,0,0,7);ctx.stroke()}
  for(const q of pl){ctx.globalAlpha=Math.max(0,q.a);ctx.strokeStyle=q.c;ctx.lineWidth=q.w;ctx.beginPath();ctx.arc(q.x,q.y,q.r,0,7);ctx.stroke()}
  for(const q of fx)if(!q.t){ctx.globalAlpha=Math.min(1,q.l/30);ctx.fillStyle=q.c;ctx.fillRect(q.x,q.y,q.s,q.s)}
  ctx.globalCompositeOperation='lighter';ctx.lineCap='round';
  for(const q of fx)if(q.t){ctx.globalAlpha=Math.min(1,q.l/30);ctx.strokeStyle=q.c;ctx.lineWidth=q.s;ctx.beginPath();ctx.moveTo(q.x-q.vx*2.4,q.y-q.vy*2.4);ctx.lineTo(q.x,q.y);ctx.stroke()}
  ctx.globalCompositeOperation='source-over';ctx.lineCap='butt';ctx.globalAlpha=1}
function anim(){const was=lastSt;lastSt=st;
  if(st==='dying'&&--dyT<=0){if(lives<=0)st='over';else load(li,1)}
  if(st==='clear'&&--clT<=0){if((li+1)%4===0){st='cpfx';cpT=0;p.vx=0;face=1}else st='between'}
  if(st==='cpfx')cpStep();
  if(st==='win'){if(was!=='win')initWin();winStep()}}
function heart(x,y,sz,fill,line){ctx.beginPath();ctx.moveTo(x,y+sz*.9);ctx.bezierCurveTo(x-sz*1.5,y-sz*.1,x-sz*.7,y-sz*1.1,x,y-sz*.35);ctx.bezierCurveTo(x+sz*.7,y-sz*1.1,x+sz*1.5,y-sz*.1,x,y+sz*.9);ctx.closePath();ctx.fillStyle=fill;ctx.fill();ctx.strokeStyle=line;ctx.lineWidth=1;ctx.stroke()}
let sp=[],si=0,sn='play',st='menu',li=0,g,dim,p,chips,sparks,mv,em,ex,tot,got,score=0,lives=5,time=0,fz=0,fcd=0,gcd=0,flash=0,msg='',bonus=0,lt=0,xl=0,th=TH[0];
const keys={},parts=Array.from({length:36},()=>({x:Math.random()*800,y:Math.random()*480,s:1+Math.random()*2}));
function load(i,q){
  li=i;th=TH[i>>2];g=LV[i].m.map(r=>r.split(''));dim='AC';chips=[];sparks=[];mv=[];em=[];ex=null;tot=0;got=0;time=0;lt=0;fz=0;fcd=0;gcd=0;
  for(let r=0;r<R;r++)for(let c=0;c<C;c++){const h=g[r][c];
    if(h==='P'){p={x:c*T+8,y:r*T+10,w:24,h:30,vx:0,vy:0,g:1,ground:false};g[r][c]='.'}
    else if(h==='K'){chips.push({x:c*T+20,y:r*T+20,on:true});tot++;g[r][c]='.'}
    else if(h==='E'){ex={x:c*T,y:r*T};g[r][c]='.'}
    else if(h==='o'){sparks.push({x:c*T+20,y:r*T+20,d:1});g[r][c]='.'}
    else if(h==='m'||h==='v'){mv.push({bx:c*T,by:r*T,x:c*T,y:r*T,ax:h,ph:mv.length*2.2});g[r][c]='.'}
    else if(h==='L'||h==='l')em.push({c,r,d:h==='L'?1:-1})}
  st='play';tr.length=0;rings.length=0;sq=0;wasG=true;
  if(!q&&i%4===0){msg=th.n;setTimeout(()=>msg='',1800)}
  if(!q&&STORY[i]){sp=STORY[i];si=0;sn='play';st='story'}
}
// Checkpoint: saved after clearing the last level of each world (4, 8, 12, 16)
let cp={li:0,score:0,lives:5};
try{const s=JSON.parse(localStorage.getItem('cb-cp'));if(s&&s.li>=0&&s.li<LV.length)cp=s}catch(e){}
function saveCp(){try{localStorage.setItem('cb-cp',JSON.stringify(cp))}catch(e){}}
function newGame(){score=0;lives=5;load(0)}
function fresh(){cp={li:0,score:0,lives:5};saveCp();fx.length=0;pl.length=0;newGame()}
function cont(){score=cp.score;lives=Math.max(3,cp.lives);load(cp.li)}
function solid(c,r){if(c<0||c>=C||r<0||r>=R)return true;const h=g[r][c];return h==='#'||h==='='||h==='L'||h==='l'||(h==='A'&&dim==='AC')||(h==='D'&&dim==='DC')}
function hit(x,y,w,h){for(let r=Math.floor(y/T);r<=Math.floor((y+h-.01)/T);r++)for(let c=Math.floor(x/T);c<=Math.floor((x+w-.01)/T);c++)if(solid(c,r))return true;
  for(const m of mv)if(x<m.x+80&&x+w>m.x&&y<m.y+20&&y+h>m.y)return true;return false}
function beam(e){let r=e.r+e.d;while(!solid(e.c,r))r+=e.d;return e.d>0?[(e.r+1)*T,r*T]:[(r+1)*T,e.r*T]}
function die(){lives--;flash=15;shake=16;hp=Math.max(0,lives);hpT=40;burst(p.x+12,p.y+15,'#ffd23f',22,5.5);burst(p.x+12,p.y+15,'#ff4d6d',16,4);dyT=50;st='dying'}
function act(a){
  if(st==='dying'||st==='clear')return;
  if(st==='cpfx'){if(a==='jump'&&cpT>90)st='between';return}
  if(st==='story'){if(a==='jump'){si++;if(si>=sp.length){st=sn;sn='play'}}return}
  if(st!=='play'){if(a==='jump'){if(st==='between'){if(li+1>=LV.length){sp=STORY.end;si=0;sn='win';st='story'}else load(li+1)}else if(st==='win'){if(winT>150)fresh()}else cp.li>0?cont():newGame()}return}
  const pw=LV[li].p;
  if(a==='jump'&&p.ground){p.vy=-th.jv*p.g;sq=.35;const fy=p.g>0?p.y+p.h:p.y;dust(p.x+12,fy,-p.g,9);rings.push({x:p.x+12,y:fy,r:4,a:.7})}
  if(a==='g'&&pw.includes('g')&&gcd<=0){p.g*=-1;p.ground=false;gcd=15}
  if(a==='x'&&pw.includes('x')){dim=dim==='AC'?'DC':'AC';if(hit(p.x,p.y,p.w,p.h)){dim=dim==='AC'?'DC':'AC';msg='Blocked: you are inside a platform';setTimeout(()=>msg='',900)}}
  if(a==='f'&&pw.includes('f')&&fcd<=0){fz=210;fcd=480}
}
function update(){
  time+=1/60;if(gcd>0)gcd--;if(fz>0)fz--;else if(fcd>0)fcd--;if(flash>0)flash--;
  const live=fz<=0;if(live)lt++;
  if(live)for(const m of mv){
    const ox=m.x,oy=m.y,rd=(p.g>0?Math.abs(p.y+p.h-oy)<3:Math.abs(p.y-oy-20)<3)&&p.x+p.w>ox&&p.x<ox+80;
    m.ph+=.02;if(m.ax==='m')m.x=m.bx+Math.sin(m.ph)*120;else m.y=m.by+Math.sin(m.ph)*70;
    if(rd||(p.x<m.x+80&&p.x+p.w>m.x&&p.y<m.y+20&&p.y+p.h>m.y)){p.x+=m.x-ox;p.y+=m.y-oy}}
  if(hit(p.x,p.y,p.w,p.h)){die();return}
  const mi=(keys.ArrowLeft||keys.a||keys.btnL?-1:0)+(keys.ArrowRight||keys.d||keys.btnR?1:0),tv=mi*3.4;if(mi)face=mi;
  p.vx=th.ice?p.vx+(tv-p.vx)*.08:tv;spin+=p.vx*.14;
  p.vy=Math.max(-16,Math.min(16,p.vy+th.gr*p.g));
  const d=p.vx+((!p.ground&&th.wind)||0);
  p.x+=d;if(d&&hit(p.x,p.y,p.w,p.h)){p.x-=d;const s=Math.sign(d);while(!hit(p.x+s,p.y,p.w,p.h))p.x+=s;p.vx=0}
  const hv=p.vy;p.y+=p.vy;p.ground=false;
  if(hit(p.x,p.y,p.w,p.h)){p.y-=p.vy;const s=Math.sign(p.vy);while(!hit(p.x,p.y+s,p.w,p.h))p.y+=s;if(s===p.g)p.ground=true;p.vy=0}
  if(p.ground&&!wasG&&Math.abs(hv)>2.5){sq=-Math.min(.35,Math.abs(hv)*.03);const fy=p.g>0?p.y+p.h:p.y;dust(p.x+12,fy,-p.g,8);rings.push({x:p.x+12,y:fy,r:4,a:.5})}
  wasG=p.ground;
  if(!p.ground&&Math.abs(p.vy)>2&&tr.length<8)tr.push({x:p.x+12,y:p.y+15,a:.35});
  for(let r=Math.floor(p.y/T);r<=Math.floor((p.y+p.h)/T);r++)for(let c=Math.floor(p.x/T);c<=Math.floor((p.x+p.w)/T);c++){const h=g[r]&&g[r][c];
    if(h==='^'&&p.x+p.w>c*T+8&&p.x<c*T+T-8&&p.y+p.h>r*T+10&&p.y<r*T+T-4){die();return}
    if(h==='u'&&p.g>0&&p.vy>=0&&p.y+p.h>r*T+24){p.vy=-Math.sqrt(440*th.gr);sq=.4;dust(p.x+12,p.y+p.h,-1,10);rings.push({x:p.x+12,y:p.y+p.h,r:4,a:.7})}}
  for(const e of em)if((lt+e.c*37)%150>=90){const[y1,y2]=beam(e);
    if(p.x+p.w>e.c*T+14&&p.x<e.c*T+26&&p.y+p.h>y1&&p.y<y2){die();return}}
  for(const s of sparks){
    if(fz<=0){const fc=Math.floor((s.x+s.d*16)/T),rr=Math.floor(s.y/T);if(solid(fc,rr)||g[rr][fc]==='x')s.d*=-1;s.x+=s.d*2}
    if(fz<=0&&Math.abs(s.x-(p.x+12))<20&&Math.abs(s.y-(p.y+15))<22){die();return}}
  for(const k of chips)if(k.on&&Math.abs(k.x-(p.x+12))<20&&Math.abs(k.y-(p.y+15))<22){k.on=false;got++;burst(k.x,k.y,'#ffe27a',12,3.2);rings.push({x:k.x,y:k.y,r:6,a:.8})}
  if(got===tot&&Math.abs(ex.x+20-(p.x+12))<24&&Math.abs(ex.y+20-(p.y+15))<26){
    bonus=Math.max(0,Math.floor((90-time)*10));xl=(li+1)%4===0&&lives<5?1:0;lives+=xl;score+=got*100+200+bonus;st='clear';clT=60;confetti(ex.x+20,ex.y+20);if((li+1)%4===0&&li+1<LV.length){cp={li:li+1,score,lives};saveCp()}}
}
function spike(c,r){const up=r>5;ctx.fillStyle='#ff5d5d';ctx.beginPath();
  for(let i=0;i<2;i++){const x=c*T+i*20;if(up){ctx.moveTo(x,r*T+T);ctx.lineTo(x+10,r*T+8);ctx.lineTo(x+20,r*T+T)}else{ctx.moveTo(x,r*T);ctx.lineTo(x+10,r*T+T-8);ctx.lineTo(x+20,r*T)}ctx.closePath()}ctx.fill();ctx.strokeStyle='rgba(255,255,255,.7)';ctx.lineWidth=.8;ctx.stroke()}
const bgCache={};
function makeBg(ti){const o=document.createElement('canvas');o.width=800;o.height=480;const x=o.getContext('2d'),t=TH[ti];
  let sd=ti*977+13;const rnd=()=>(sd=(sd*16807)%2147483647)/2147483647;
  const gr=x.createLinearGradient(0,0,0,480);gr.addColorStop(0,t.bg);gr.addColorStop(1,t.b2);x.fillStyle=gr;x.fillRect(0,0,800,480);
  // distant tower skyline
  for(let i=0;i<20;i++){const w=28+rnd()*26,h=60+rnd()*150,xx=i*41;x.globalAlpha=.3;x.fillStyle='#000';x.fillRect(xx,446-h,w,h);
    for(let wy=446-h+8;wy<436;wy+=14)for(let wx=xx+5;wx<xx+w-6;wx+=10)if(rnd()<.22){x.globalAlpha=.2;x.fillStyle=t.ed;x.fillRect(wx,wy,4,6)}}
  // faint grid
  x.globalAlpha=.07;x.strokeStyle=t.ed;x.lineWidth=1;
  for(let i=0;i<=800;i+=40){x.beginPath();x.moveTo(i+.5,0);x.lineTo(i+.5,480);x.stroke()}
  for(let j=0;j<=480;j+=40){x.beginPath();x.moveTo(0,j+.5);x.lineTo(800,j+.5);x.stroke()}
  // circuit traces with nodes
  x.globalAlpha=.2;x.lineWidth=2;x.strokeStyle=t.ac;x.fillStyle=t.ac;
  for(let k=0;k<26;k++){let cx=Math.floor(rnd()*20)*40+20,cy=Math.floor(rnd()*12)*40+20;x.beginPath();x.moveTo(cx,cy);
    for(let n=0;n<3;n++){const d=(rnd()<.5?-1:1)*40*(1+Math.floor(rnd()*3));if(rnd()<.5)cx+=d;else cy+=d;x.lineTo(cx,cy)}
    x.stroke();x.beginPath();x.arc(cx,cy,5,0,7);x.fill()}
  // vignette
  x.globalAlpha=1;const rg=x.createRadialGradient(400,240,80,400,240,520);rg.addColorStop(0,'rgba(255,255,255,.07)');rg.addColorStop(1,'rgba(0,0,0,.5)');x.fillStyle=rg;x.fillRect(0,0,800,480);
  return o}
let spin=0;
function rr(x,y,w,h,r,f){ctx.beginPath();ctx.moveTo(x+r,y);ctx.lineTo(x+w-r,y);ctx.quadraticCurveTo(x+w,y,x+w,y+r);ctx.lineTo(x+w,y+h-r);ctx.quadraticCurveTo(x+w,y+h,x+w-r,y+h);ctx.lineTo(x+r,y+h);ctx.quadraticCurveTo(x,y+h,x,y+h-r);ctx.lineTo(x,y+r);ctx.quadraticCurveTo(x,y,x+r,y);ctx.closePath();ctx.fillStyle=f;ctx.fill()}
// Memory chip: dark package, silver pins, glowing gold die, gentle hover and sparkle
function drawChip(cx,cy){const t=Date.now(),y=cy+Math.sin(t/260+cx)*2.5;
  ctx.globalAlpha=.28+.14*Math.sin(t/220+cx);ctx.fillStyle='#ffd23f';ctx.beginPath();ctx.arc(cx,y,17,0,7);ctx.fill();ctx.globalAlpha=1;
  ctx.fillStyle='#d4daf0';for(let i=-1;i<=1;i++){ctx.fillRect(cx-13,y+i*5-1,5,2);ctx.fillRect(cx+8,y+i*5-1,5,2);ctx.fillRect(cx+i*5-1,y-13,2,5);ctx.fillRect(cx+i*5-1,y+8,2,5)}
  rr(cx-9,y-9,18,18,3,'#1a2142');ctx.strokeStyle='#6fb5ff';ctx.lineWidth=1;ctx.strokeRect(cx-8.5,y-8.5,17,17);
  rr(cx-5,y-5,10,10,2,'#ffd23f');rr(cx-5,y-5,10,4,2,'#fff0a0');
  ctx.fillStyle='#1a2142';ctx.fillRect(cx-2,y-1,4,4);ctx.fillStyle='#ffffff';ctx.beginPath();ctx.arc(cx-6.5,y-6.5,1.1,0,7);ctx.fill();
  if(Math.sin(t/330+cx*.7)>.82){ctx.strokeStyle='#ffffff';ctx.lineWidth=1.2;ctx.beginPath();ctx.moveTo(cx+11,y-14);ctx.lineTo(cx+11,y-8);ctx.moveTo(cx+8,y-11);ctx.lineTo(cx+14,y-11);ctx.stroke()}}
// Mini repair bot: antenna light, visor eyes, chest lights, wheels and a wrench arm
function drawBot(){const t=Date.now(),air=!p.ground&&st==='play',run=p.ground&&Math.abs(p.vx)>.4;
  ctx.strokeStyle='#cfd6ff';ctx.lineWidth=1.5;ctx.beginPath();ctx.moveTo(face*2,-13);ctx.lineTo(face*3,-18);ctx.stroke();
  ctx.fillStyle=Math.floor(t/350)%2?'#ff4d6d':'#7a1e2e';ctx.beginPath();ctx.arc(face*3,-19.5,2.4,0,7);ctx.fill();
  rr(-12,8,24,7,3.5,'#232842'); 
  for(let i=-1;i<=1;i++){const wx=i*7,a=spin+i;ctx.fillStyle='#8a93c4';ctx.beginPath();ctx.arc(wx,11.5,2.7,0,7);ctx.fill();
    ctx.strokeStyle='#232842';ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(wx,11.5);ctx.lineTo(wx+Math.cos(a)*2.5,11.5+Math.sin(a)*2.5);ctx.stroke()}
  rr(-10,-13,20,22,5,'#ffd23f');rr(-10,-13,20,6,4,'#fff0a0');rr(-10,4,20,5,2,'#e0a800');
  rr(-8,-11,16,9,3,'#0b1020');const bl=Math.floor(t/140)%24===0;
  ctx.fillStyle='#3dc8ff';ctx.fillRect(-6+face*1.5,bl?-7:-9,4,bl?1.5:4.5);ctx.fillRect(2+face*1.5,bl?-7:-9,4,bl?1.5:4.5);
  const k=Math.floor(t/300)%3;['#3ddc97','#ffd23f','#ff4d6d'].forEach((c,i)=>{ctx.fillStyle=i===k?c:'#4a4a5a';ctx.fillRect(-6+i*5,-.5,3,3)});
  const sw=run?Math.sin(t/70)*.5:(air?-1:Math.sin(t/500)*.12);
  ctx.save();ctx.translate(face*10,-1);ctx.rotate(face*(.5+sw));
  ctx.fillStyle='#e0a800';ctx.fillRect(-1.5,0,3,6);ctx.strokeStyle='#cfd6ff';ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(0,5);ctx.lineTo(0,12);ctx.stroke();
  ctx.beginPath();ctx.arc(0,14,3,.5,5.8);ctx.stroke();ctx.restore()}
function tx(s,x,y,sz,col,al){ctx.fillStyle=col||'#e8ecff';ctx.font='700 '+sz+'px system-ui,sans-serif';ctx.textAlign=al||'center';ctx.fillText(s,x,y)}
function draw(){
  ctx.save();if(shake>0)ctx.translate((Math.random()-.5)*shake,(Math.random()-.5)*shake);
  const ti=TH.indexOf(th);ctx.drawImage(bgCache[ti]||(bgCache[ti]=makeBg(ti)),0,0);
  {ctx.fillStyle=th.ac;ctx.globalAlpha=.3;for(let i=0;i<6;i++){const t0=(Date.now()/18+i*150)%900-50;ctx.fillRect(t0,20+((i*5)%11)*40-1,26,2)}ctx.globalAlpha=1}
  {const pu=.75+.25*Math.sin(Date.now()/350),gg=ctx.createLinearGradient(0,414,0,440),fc=th.fl.join(',');gg.addColorStop(0,'rgba('+fc+',0)');gg.addColorStop(1,'rgba('+fc+','+(.55*pu)+')');ctx.fillStyle=gg;ctx.fillRect(0,414,800,26)}
  if(dim==='DC'){ctx.fillStyle='rgba(255,120,40,.1)';ctx.fillRect(0,0,800,480)}
  ctx.globalAlpha=.3;ctx.fillStyle=th.ed;
  for(const q of parts){if(th.dy){q.y+=th.dy*q.s;if(q.y>480)q.y=0;if(q.y<0)q.y=480}ctx.fillRect(q.x,q.y,q.s*1.5,q.s*1.5)}
  ctx.globalAlpha=1;
  if(th.gl&&Math.random()<.05){ctx.fillStyle='rgba(255,107,214,.14)';ctx.fillRect(0,Math.random()*480,800,3+Math.random()*8)}
  if(flash){ctx.fillStyle='rgba(255,60,60,.3)';ctx.fillRect(0,0,800,480)}
  for(let r=0;r<R;r++)for(let c=0;c<C;c++){const h=g[r][c],x=c*T,y=r*T;
    if(h==='#'){if(r===0||r===R-1||c===0||c===C-1){
      if(c===0){ctx.fillStyle=th.wl;ctx.fillRect(x+36,y,4,T);ctx.fillStyle=th.ed;ctx.fillRect(x+38,y,2,T)}
      if(c===C-1){ctx.fillStyle=th.wl;ctx.fillRect(x,y,4,T);ctx.fillStyle=th.ed;ctx.fillRect(x,y,2,T)}
      if(r===0){ctx.fillStyle=th.wl;ctx.fillRect(x,y+36,T,4);ctx.fillStyle=th.ed;ctx.fillRect(x,y+38,T,2)}
      if(r===R-1){ctx.fillStyle=th.wl;ctx.fillRect(x,y,T,4);ctx.fillStyle=th.ed;ctx.fillRect(x,y,T,2)}}
    else{ctx.fillStyle=th.wl;ctx.fillRect(x,y,T,T);ctx.strokeStyle=th.ed;ctx.lineWidth=.6;ctx.strokeRect(x+.5,y+.5,T-1,T-1)}}
    else if(h==='='){ctx.fillStyle=th.wl;ctx.fillRect(x,y,T,T);ctx.fillStyle=th.ac;ctx.fillRect(x,y,T,6);ctx.strokeStyle=th.ed;ctx.lineWidth=.6;ctx.strokeRect(x+.5,y+.5,T-1,T-1)}
    else if(h==='L'||h==='l'){ctx.fillStyle='#2a1426';ctx.fillRect(x,y,T,T);ctx.strokeStyle='#ff4d6d';ctx.lineWidth=.6;ctx.strokeRect(x+.5,y+.5,T-1,T-1);ctx.fillStyle='#ff4d6d';ctx.fillRect(x+14,y+(h==='L'?24:8),12,8)}
    else if(h==='A'||h==='D'){const on=(h==='A')===(dim==='AC'),col=h==='A'?'#3dc8ff':'#ff9a3d';
      if(on){ctx.fillStyle=col;ctx.fillRect(x,y,T,T)}else{ctx.strokeStyle=col;ctx.lineWidth=1;ctx.setLineDash([4,4]);ctx.strokeRect(x+3,y+3,T-6,T-6);ctx.setLineDash([])}}
    else if(h==='^')spike(c,r);
    else if(h==='u'){ctx.fillStyle='#3ddc97';ctx.fillRect(x+4,y+30,32,10);ctx.fillRect(x+10,y+24,20,6)}
    else if(h==='x'){ctx.fillStyle='rgba(255,255,255,.12)';ctx.fillRect(x+17,y+4,6,T-8)}}
  for(const e of em){const ph=(lt+e.c*37)%150;
    if(ph>=70){const[y1,y2]=beam(e),x=e.c*T;
      if(ph>=90){ctx.fillStyle='rgba(255,60,90,.75)';ctx.fillRect(x+15,y1,10,y2-y1);ctx.fillStyle='#ffe0e6';ctx.fillRect(x+19,y1,2,y2-y1)}
      else if(Math.floor(lt/4)%2){ctx.fillStyle='rgba(255,90,110,.55)';ctx.fillRect(x+19,y1,2,y2-y1)}}}
  for(const m of mv){ctx.fillStyle=th.ac;ctx.fillRect(m.x,m.y,80,20);ctx.fillStyle='rgba(0,0,0,.35)';for(let i=0;i<4;i++)ctx.fillRect(m.x+8+i*20,m.y+6,10,8)}
  if(ex){const open=got===tot;ctx.fillStyle=open?'#3ddc97':'#3a4468';ctx.fillRect(ex.x+8,ex.y+4,24,36);tx('EXIT',ex.x+20,ex.y-2,11,open?'#3ddc97':'#6a74a0')}
  for(const k of chips)if(k.on)drawChip(k.x,k.y);
  for(const s of sparks){ctx.fillStyle=fz>0?'#8fd3ff':'#ff4d8d';ctx.beginPath();ctx.arc(s.x,s.y,12,0,7);ctx.fill();ctx.fillStyle='#fff';ctx.beginPath();ctx.arc(s.x,s.y,4,0,7);ctx.fill()}
  if(st==='play'||st==='between'||st==='clear'){
    if(st==='play'){ctx.fillStyle='#ffd23f';for(const t of tr){const k=.55+t.a;ctx.globalAlpha=t.a;ctx.fillRect(t.x-12*k,t.y-15*k,24*k,30*k)}ctx.globalAlpha=1}
    ctx.save();let sc=1,rot=0,sx=1,sy=1,px=p.x+12,py=p.y+15;
    if(st==='clear'){const t=1-clT/60;px+=(ex.x+20-px)*t;py+=(ex.y+22-py)*t;sc=Math.max(.05,1-t);rot=t*10}
    else{const air=p.ground?0:Math.min(.25,Math.abs(p.vy)*.022);sy=Math.max(.6,Math.min(1.45,1+sq+air));sx=1+(1-sy)*.7;rot=p.ground?0:p.vx*.05;py+=15*p.g*(1-sy)}
    ctx.translate(px,py);ctx.rotate(rot);ctx.scale(sc*sx,sc*sy*p.g);
    drawBot();
    ctx.restore()}
  if(fz>0){ctx.fillStyle='rgba(120,200,255,.14)';ctx.fillRect(0,0,800,480)}
  ctx.fillStyle='rgba(4,7,20,.84)';ctx.fillRect(0,0,800,46);ctx.fillRect(0,446,800,34);
  tx('Level '+(li+1)+'/'+LV.length+': '+LV[li].n,12,19,16,'#ffffff','left');
  tx(th.n,12,38,12,th.ed,'left');
  tx('Score '+score+'   Chips '+got+'/'+tot+'   Time '+time.toFixed(1)+'s',788,19,15,'#ffffff','right');
  for(let i=0;i<5;i++){const hx=668+i*24,lost=i>=lives,pop=lost&&i===hp&&hpT>0;
    if(!lost)heart(hx,34,9,'#ff3b5c','#ffffff');
    else if(pop)heart(hx,34,9+(40-hpT)*.2,'rgba(255,59,92,'+(hpT/40)+')','rgba(255,255,255,'+(hpT/40)+')');
    else heart(hx,34,9,'#1b2036','#6b7599')}
  tx(LV[li].h,400,468,15,'#ffffff');
  if(LV[li].p.includes('f')){ctx.fillStyle='#2a3570';ctx.fillRect(10,50,120,8);ctx.fillStyle='#8fd3ff';ctx.fillRect(10,50,120*(fz>0?fz/210:1-fcd/480),8)}
  if(msg)tx(msg,400,240,18,'#ffd23f');
  if(st==='clear'){const t=1-clT/60;ctx.save();ctx.translate(400,200);const z=Math.min(1.2,t*3);ctx.scale(z,z);tx('LEVEL CLEAR!',0,0,48,'#ffffff');ctx.restore()}
  if(st==='dying')tx(lives>0?'Zapped! '+lives+(lives>1?' hearts':' heart')+' left':'Out of hearts!',400,210,28,'#ffffff');
  const bx=(id,k)=>document.getElementById(id).classList.toggle('off',!LV[li].p.includes(k));bx('bg','g');bx('bx','x');bx('bf','f');
  if(st==='menu'||st==='over'||st==='between'||st==='story'){ctx.fillStyle='rgba(8,12,28,.88)';ctx.fillRect(0,0,800,480)}
  if(st==='story'){const pg=sp[si],col={VOLT:'#ffd23f',NOISE:'#ff4d8d',SYSTEM:'#3dc8ff'}[pg.w]||'#e8ecff';
    ctx.fillStyle='rgba(20,28,60,.95)';ctx.fillRect(60,120,680,230);ctx.strokeStyle=col;ctx.lineWidth=1;ctx.strokeRect(60,120,680,230);
    tx(th.n,400,92,20,th.ed);tx(pg.w,90,162,22,col,'left');pg.t.forEach((l,i)=>tx(l,90,208+i*34,20,'#e8ecff','left'));
    tx((si+1)+'/'+sp.length,710,335,14,'#8a93c4','right');tx('Tap or press Space to continue   (Esc = skip)',400,392,18,'#3ddc97')}
  if(st==='menu'){tx('Dead Signal',400,120,52,'#ffd23f');tx('GlitchVerse: the world is broken, so break the rules.',400,162,19);
    tx('Collect all chips, reach the exit. 5 lives, 20 levels, 5 worlds.',400,200,17,'#8a93c4');
    tx('G = flip gravity   X = switch AC/DC   F = freeze time',400,235,17,'#3dc8ff');tx('Move: arrows or buttons   Jump: Space',400,262,17,'#8a93c4');
    tx('Hazards: spikes, sparks, lasers. Ride moving platforms and pads.',400,289,15,'#8a93c4');
    tx(cp.li?'Tap or Space: continue from World '+(cp.li/4+1)+' checkpoint':'Tap or press Space to start   (keys 1-5 start in a world)',400,350,22,'#3ddc97');if(cp.li)tx('Press N or tap the bottom edge for a new game',400,440,15,'#8a93c4')}
  if(st==='between'){tx('Level clear!',400,150,46,'#3ddc97');tx('Chips +'+got*100+'   Clear +200   Speed bonus +'+bonus,400,200,19);
    tx('Score '+score,400,245,28);if(xl)tx('World complete! +1 life',400,285,20,'#ff5d5d');if((li+1)%4===0&&li+1<LV.length)tx('Checkpoint saved',400,312,18,'#3dc8ff');tx(li+1>=LV.length?'Tap for results':'Tap for next level',400,340,22,'#ffd23f')}
  if(st==='over'){tx('Game over',400,170,50,'#ff5d5d');tx('Final score '+score,400,225,26);tx('Reached level '+(li+1)+' of '+LV.length,400,265,18,'#8a93c4');tx(cp.li?'Tap or Space: retry from World '+(cp.li/4+1)+' checkpoint':'Tap or press Space to restart',400,320,22,'#3ddc97');if(cp.li)tx('Press N or tap the bottom edge for a new game',400,440,15,'#8a93c4')}
  if(st==='win')drawWin();
  if(st==='cpfx')drawCp();
  drawFx();ctx.restore()
}

// ---- Cinematics: world-clear checkpoint show (after worlds 1-5) + grand finale ----
let cpT=0,winT=0,lastSt='',wtr=null;
const WR=TH.map(t=>hex2rgb(t.ac)),WCOL=TH.map(t=>t.ac),CPX=400,CPY=262,CPR=104;
function hex2rgb(h){h=h.replace('#','');return[parseInt(h.substr(0,2),16),parseInt(h.substr(2,2),16),parseInt(h.substr(4,2),16)]}
function rgba(a,al){return'rgba('+a[0]+','+a[1]+','+a[2]+','+al+')'}
function clamp01(v){return v<0?0:v>1?1:v}
function eob(t){t=clamp01(t);return 1+2.70158*Math.pow(t-1,3)+1.70158*Math.pow(t-1,2)}
function pulse(x,y,c,dr,w){pl.push({x,y,r:6,a:1,c,dr,da:.022,w})}
function firework(x,y,cols,n){const k=cols[Math.floor(Math.random()*cols.length)],k2=cols[Math.floor(Math.random()*cols.length)];
  for(let i=0;i<n;i++){const a=i/n*6.283+Math.random()*.1,sp2=(i%2?1:.55)*(3.2+Math.random()*1.2)*1.5;
    fx.push({x,y,vx:Math.cos(a)*sp2,vy:Math.sin(a)*sp2,g:.06,d:.955,l:55+Math.random()*30,c:i%2?k:k2,s:2.2,t:1})}
  for(let i=0;i<10;i++){const a=Math.random()*6.283;fx.push({x,y,vx:Math.cos(a)*1.6,vy:Math.sin(a)*1.6,g:0,d:.9,l:22,c:'#ffffff',s:3,t:1})}}
function glitch(n){for(let i=0;i<n;i++){const y=Math.random()*470|0,h=4+(Math.random()*26|0),dx=(Math.random()-.5)*50;ctx.drawImage(cv,0,y,800,h,dx,y,800,h)}}
function hex(cx,cy,r,rot){ctx.beginPath();for(let i=0;i<6;i++){const a=rot+i*Math.PI/3;if(i)ctx.lineTo(cx+Math.cos(a)*r,cy+Math.sin(a)*r);else ctx.moveTo(cx+Math.cos(a)*r,cy+Math.sin(a)*r)}ctx.closePath()}
const cpWorld=()=>Math.round((li+1)/4);
function nodeXY(k,cx,cy,R){const a=-Math.PI/2+k*Math.PI*2/5;return[cx+Math.cos(a)*R,cy+Math.sin(a)*R]}
const scr=(t,i)=>'01#%&@$/<>?'[(Math.floor(t/2)*7+i*13)%11];

function cpStep(){cpT++;const t=cpT,w=cpWorld(),cols=[th.ac,th.ed,'#ffffff',TH[w-1].ac];
  if(t===2){pulse(CPX,CPY,'#ffffff',14,6);pulse(CPX,CPY,th.ac,10,4)}
  if(t===12)pulse(CPX,CPY,th.ed,8,3);
  if(t===80){const[nx,ny]=nodeXY(w-1,CPX,CPY,CPR);firework(nx,ny,cols,46);pulse(nx,ny,'#ffffff',5,3);pulse(nx,ny,th.ac,8,3)}
  if(t>=30&&t%26===0&&t<300&&fx.length<900)firework(120+Math.random()*560,90+Math.random()*200,cols,34);
  if(t===230){pulse(400,417,'#3ddc97',9,3);burst(400,417,'#3ddc97',26,5);burst(528,417,'#ffffff',14,4)}
  if(t>=360)st='between'}

function drawCp(){
  const t=cpT,w=cpWorld(),fin=li+1>=LV.length,cx=CPX,cy=CPY,fl=th.fl,ac=hex2rgb(th.ac),ti=TH.indexOf(th);
  ctx.save();
  ctx.globalAlpha=clamp01(t/16);
  ctx.drawImage(bgCache[ti]||(bgCache[ti]=makeBg(ti)),0,0);
  ctx.fillStyle='rgba(4,7,20,.66)';ctx.fillRect(0,0,800,480);
  ctx.globalAlpha=1;
  // power surge: data streams and spinning light rays
  ctx.globalCompositeOperation='lighter';
  for(let i=0;i<12;i++){const y=60+i*34.5,x0=((t*(5+i%3)+i*131)%1000)-170,gr=ctx.createLinearGradient(x0,0,x0+150,0);
    gr.addColorStop(0,rgba(ac,0));gr.addColorStop(1,rgba(ac,.42));ctx.strokeStyle=gr;ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(x0,y);ctx.lineTo(x0+150,y);ctx.stroke()}
  ctx.fillStyle=rgba(fl,clamp01((t-8)/30)*.15);
  for(let i=0;i<14;i++){const a=t*.006+i*Math.PI/7;ctx.beginPath();ctx.moveTo(cx,cy);ctx.lineTo(cx+Math.cos(a)*700,cy+Math.sin(a)*700);ctx.lineTo(cx+Math.cos(a+.1)*700,cy+Math.sin(a+.1)*700);ctx.closePath();ctx.fill()}
  ctx.globalCompositeOperation='source-over';
  // world-progress emblem
  const ea=clamp01((t-4)/20),gl=ctx.createRadialGradient(cx,cy,6,cx,cy,CPR);
  gl.addColorStop(0,rgba(fl,.5+.12*Math.sin(t*.12)));gl.addColorStop(1,rgba(fl,0));
  ctx.globalAlpha=ea;ctx.fillStyle=gl;ctx.beginPath();ctx.arc(cx,cy,CPR,0,7);ctx.fill();
  ctx.lineWidth=3;ctx.strokeStyle='rgba(255,255,255,.14)';ctx.beginPath();ctx.arc(cx,cy,CPR,0,7);ctx.stroke();
  ctx.lineWidth=2;ctx.strokeStyle=th.ed;ctx.globalAlpha=.45*ea;ctx.setLineDash([5,11]);
  ctx.lineDashOffset=-t*.7;ctx.beginPath();ctx.arc(cx,cy,CPR+16,0,7);ctx.stroke();
  ctx.lineDashOffset=t*.5;ctx.beginPath();ctx.arc(cx,cy,CPR-26,0,7);ctx.stroke();
  ctx.setLineDash([]);ctx.globalAlpha=1;
  const A=k=>-Math.PI/2+k*Math.PI*2/5;
  ctx.lineWidth=5;ctx.lineCap='round';
  for(let k=0;k<w-1;k++){const f=k===w-2?clamp01((t-34)/46):clamp01((t-4-k*8)/24);if(f<=0)continue;
    ctx.save();ctx.shadowColor=TH[k+1].ac;ctx.shadowBlur=12;ctx.strokeStyle=TH[k+1].ac;ctx.beginPath();ctx.arc(cx,cy,CPR,A(k),A(k)+Math.PI*.4*f);ctx.stroke();ctx.restore();
    if(f<1&&k===w-2){const a=A(k)+Math.PI*.4*f;ctx.fillStyle='#fff';ctx.beginPath();ctx.arc(cx+Math.cos(a)*CPR,cy+Math.sin(a)*CPR,5,0,7);ctx.fill()}}
  ctx.lineCap='butt';
  for(let k=0;k<5;k++){const[nx,ny]=nodeXY(k,cx,cy,CPR);let lit=false,sc=1;
    if(k<w-1){lit=true;sc=eob((t-2-k*7)/16)}
    else if(k===w-1){if(t>=80){lit=true;sc=1+.6*Math.max(0,1-(t-80)/18)}else{ctx.globalAlpha=.6;ctx.strokeStyle=th.ac;ctx.lineWidth=2;ctx.beginPath();ctx.arc(nx,ny,21+3*Math.sin(t*.2),0,7);ctx.stroke();ctx.globalAlpha=1}}
    ctx.save();ctx.translate(nx,ny);ctx.scale(sc,sc);
    if(lit){ctx.shadowColor=TH[k].ac;ctx.shadowBlur=18;ctx.fillStyle=TH[k].ac}else ctx.fillStyle='rgba(8,12,30,.92)';
    ctx.beginPath();ctx.arc(0,0,17,0,7);ctx.fill();ctx.shadowBlur=0;
    ctx.lineWidth=2;ctx.strokeStyle=lit?'#ffffff':(k===w-1?th.ac:'rgba(255,255,255,.28)');ctx.stroke();
    tx(String(k+1),0,6,17,lit?'#0b1020':'rgba(255,255,255,.4)');ctx.restore()}
  // Volt celebrates
  const eb=eob((t-6)/22),hop=t>40?Math.abs(Math.sin((t-40)*.075)):0;
  ctx.fillStyle='rgba(0,0,0,.35)';ctx.beginPath();ctx.ellipse(cx,cy+44,30*eb*(1-hop*.3),6,0,0,7);ctx.fill();
  ctx.save();ctx.translate(cx,cy+2-hop*22);ctx.scale(2.3*eb,2.3*eb);drawBot();ctx.restore();
  // glitch-in title
  const tt=fin?'ALL WORLDS CLEARED':'WORLD '+w+' CLEARED',tz=fin?40:46;let s='';
  for(let i=0;i<tt.length;i++)s+=(tt[i]===' '||t>14+i*2.5)?tt[i]:scr(t,i);
  ctx.save();ctx.translate(400,82);const ps=1+Math.max(0,.4*(1-(t-8)/18));ctx.scale(ps,ps);ctx.globalAlpha=clamp01((t-6)/8);
  if(t<30||t%120<6){ctx.globalCompositeOperation='lighter';tx(s,-4,0,tz,'#ff2d6f');tx(s,4,0,tz,'#2de2ff');ctx.globalCompositeOperation='source-over'}
  ctx.shadowColor=th.ac;ctx.shadowBlur=28;tx(s,0,0,tz,'#ffffff');ctx.restore();
  const nm=fin?'The Core is within reach':th.n.split(': ')[1].replace(/\s*\(.*\)/,''),lw=150*eob((t-20)/30);
  ctx.fillStyle=th.ac;ctx.globalAlpha=.8;ctx.fillRect(400-lw,128,lw*2,2);
  ctx.globalAlpha=clamp01((t-24)/14);tx(nm,400,116,22,th.ed);
  // lives + score
  const sa=clamp01((t-44)/16);ctx.globalAlpha=sa;
  heart(120,226,17,'#ff3b5c','#ffffff');tx('x '+lives,120,278,26,'#ffffff');
  if(xl&&t>100){const pp=eob((t-100)/18);ctx.save();ctx.translate(120,306);ctx.scale(pp,pp);ctx.globalAlpha=sa;tx('+1 LIFE',0,0,18,'#ff8fab');ctx.restore()}
  const sc0=score-(got*100+200+bonus),sv=Math.round(sc0+(score-sc0)*(1-Math.pow(1-clamp01((t-50)/90),3)));
  tx('SCORE',680,232,14,'#8a93c4');tx(String(sv),680,270,34,'#ffd23f');
  ctx.globalAlpha=1;
  // checkpoint save bar
  const ba=clamp01((t-120)/16),bt=clamp01((t-140)/90),done=t>=230;
  if(ba>0){ctx.globalAlpha=ba;
    const lab=fin?(done?'CORE ONLINE':'Powering up the Core...'):(done?'CHECKPOINT SAVED':'Saving checkpoint...');
    tx(lab,400,402,20,done?'#3ddc97':'#ffffff');
    rr(300,412,200,10,5,'rgba(255,255,255,.14)');
    if(bt>0){ctx.save();if(done){ctx.shadowColor='#3ddc97';ctx.shadowBlur=14}rr(300,412,Math.max(10,200*bt),10,5,done?'#3ddc97':th.ac);ctx.restore()}
    if(done){const k=eob((t-230)/14);ctx.save();ctx.translate(528,417);ctx.scale(k,k);ctx.fillStyle='#3ddc97';ctx.beginPath();ctx.arc(0,0,13,0,7);ctx.fill();
      ctx.strokeStyle='#0b1020';ctx.lineWidth=3;ctx.lineCap='round';ctx.beginPath();ctx.moveTo(-6,0);ctx.lineTo(-2,5);ctx.lineTo(6,-5);ctx.stroke();ctx.restore()}
    ctx.globalAlpha=1}
  if(t>90){ctx.globalAlpha=.45+.35*Math.sin(t*.1);tx('Tap or press Space to continue',400,462,15,'#8a93c4');ctx.globalAlpha=1}
  if(t<28)glitch(3+((28-t)>>2));else if(t%120<4)glitch(3);
  ctx.restore()}

// ---- Finale ----
function initWin(){winT=0;fx.length=0;pl.length=0;p.vx=0;face=1;wtr=[];let sd=4242;const rnd=()=>(sd=(sd*16807)%2147483647)/2147483647;
  for(let k=0;k<30;k++){let x=400,y=258,dir=Math.floor(rnd()*4);const pts=[[x,y]];let tot=0;
    for(let s=0;s<6;s++){const n=1+Math.floor(rnd()*3),dx=[1,0,-1,0][dir],dy=[0,1,0,-1][dir];x+=dx*n*40;y+=dy*n*40;tot+=n*40;pts.push([x,y]);dir=(dir+(rnd()<.5?1:3))%4}
    wtr.push({pts,tot,col:k%5,off:rnd()*300})}}
function winStep(){winT++;const t=winT,pal=WCOL.concat('#ffffff');
  if(t===62){pulse(400,258,'#ffffff',18,10);firework(400,258,pal,90)}
  if(t===66||t===72||t===78||t===84)pulse(400,258,WCOL[(t-66)/6],12,5);
  if(t>66&&t%16===0&&fx.length<1000)firework(60+Math.random()*680,70+Math.random()*260,pal,34+(Math.random()*16|0));
  if(t>70&&t%2===0&&fx.length<1300)fx.push({x:Math.random()*800,y:-6,vx:(Math.random()-.5)*1.2,vy:1+Math.random()*1.8,g:.004,l:300,c:pal[Math.floor(Math.random()*6)],s:4+Math.random()*4});
  if(t>150&&t%45===0)pulse(400,258,WCOL[(t/45|0)%5],5,3)}
function trPath(q,L){ctx.beginPath();ctx.moveTo(q.pts[0][0],q.pts[0][1]);let rem=L;
  for(let i=1;i<q.pts.length;i++){const a=q.pts[i-1],b=q.pts[i],sl=Math.abs(b[0]-a[0])+Math.abs(b[1]-a[1]);
    if(rem>=sl){ctx.lineTo(b[0],b[1]);rem-=sl}else{const f=rem/sl;ctx.lineTo(a[0]+(b[0]-a[0])*f,a[1]+(b[1]-a[1])*f);return}}}
function trAt(q,L){let rem=L;for(let i=1;i<q.pts.length;i++){const a=q.pts[i-1],b=q.pts[i],sl=Math.abs(b[0]-a[0])+Math.abs(b[1]-a[1]);
  if(rem<=sl){const f=rem/sl;return[a[0]+(b[0]-a[0])*f,a[1]+(b[1]-a[1])*f]}rem-=sl}return q.pts[q.pts.length-1]}
function drawWin(){
  const t=winT,cx=400,cy=258;ctx.save();
  if(t<62){
    ctx.fillStyle='#04060f';ctx.fillRect(0,0,800,480);
    const it=1-t/62,pc=clamp01(t/60);
    for(let i=0;i<20+90*it;i++){const r=Math.random();ctx.fillStyle=r<.15?'rgba(255,60,120,.5)':r<.35?'rgba(60,200,255,.45)':'rgba(255,255,255,'+(.1+Math.random()*.3*it)+')';ctx.fillRect(Math.random()*800,Math.random()*480,20+Math.random()*260,1+Math.random()*3)}
    tx('SIGNAL '+Math.floor(pc*100)+'%',400+(Math.random()-.5)*6*it,244,30,'#3dc8ff');
    rr(300,262,200,8,4,'rgba(255,255,255,.14)');rr(300,262,Math.max(8,200*pc),8,4,'#3dc8ff');
    tx('Noise is shutting down...',400,306,16,'#8a93c4');
    ctx.restore();return}
  const wt=t-62,ga=clamp01(wt/40),ce=eob(wt/30);
  const bg=ctx.createRadialGradient(cx,cy,10,cx,cy,520);bg.addColorStop(0,'#26409e');bg.addColorStop(.5,'#0b1240');bg.addColorStop(1,'#03050f');
  ctx.fillStyle=bg;ctx.fillRect(0,0,800,480);
  ctx.globalCompositeOperation='lighter';
  for(let i=0;i<20;i++){const a=wt*.005+i*Math.PI/10;ctx.fillStyle=rgba(WR[i%5],.08*ga);ctx.beginPath();ctx.moveTo(cx,cy);ctx.lineTo(cx+Math.cos(a)*700,cy+Math.sin(a)*700);ctx.lineTo(cx+Math.cos(a+.09)*700,cy+Math.sin(a+.09)*700);ctx.closePath();ctx.fill()}
  // power returns: circuit traces light up outward from the Core
  ctx.lineWidth=2.5;
  for(let i=0;i<wtr.length;i++){const q=wtr[i],L=(wt-i*2)*9;if(L<=0)continue;const col=TH[q.col].ac;
    ctx.globalAlpha=.4;ctx.strokeStyle=col;trPath(q,Math.min(L,q.tot));ctx.stroke();
    if(L>=q.tot){const e=q.pts[q.pts.length-1];ctx.globalAlpha=.7;ctx.fillStyle=col;ctx.beginPath();ctx.arc(e[0],e[1],4,0,7);ctx.fill();
      const d=trAt(q,(wt*5+q.off)%q.tot);ctx.globalAlpha=1;ctx.fillStyle='#ffffff';ctx.beginPath();ctx.arc(d[0],d[1],3,0,7);ctx.fill()}}
  
  const cg=ctx.createRadialGradient(cx,cy,0,cx,cy,100*ce+1);cg.addColorStop(0,'rgba(255,255,255,.95)');cg.addColorStop(.25,'rgba(120,220,255,.55)');cg.addColorStop(1,'rgba(60,100,255,0)');
  ctx.fillStyle=cg;ctx.beginPath();ctx.arc(cx,cy,100*ce+1,0,7);ctx.fill();
  ctx.globalCompositeOperation='source-over';
  ctx.lineWidth=2;ctx.strokeStyle='#bdeaff';ctx.globalAlpha=.85*ga;hex(cx,cy,46*ce,wt*.02);ctx.stroke();
  ctx.strokeStyle='#ffd23f';ctx.lineWidth=1.5;hex(cx,cy,34*ce,-wt*.03);ctx.stroke();ctx.globalAlpha=1;
  // the five worlds orbit the restored Core
  for(let k=0;k<5;k++){const r=(66+k*14)*ce,col=TH[k].ac,a=wt*(.032-.0045*k)+k*1.26;
    ctx.strokeStyle=col;ctx.lineWidth=1;ctx.globalAlpha=.25*ga;ctx.beginPath();ctx.arc(cx,cy,r,0,7);ctx.stroke();
    ctx.lineWidth=3;ctx.globalAlpha=.75*ga;ctx.beginPath();ctx.arc(cx,cy,r,a-.8,a);ctx.stroke();
    ctx.globalAlpha=ga;ctx.save();ctx.fillStyle=col;ctx.shadowColor=col;ctx.shadowBlur=14;ctx.beginPath();ctx.arc(cx+Math.cos(a)*r,cy+Math.sin(a)*r,6+(k%2)*1.5,0,7);ctx.fill();ctx.restore()}
  ctx.globalAlpha=1;
  // Volt, hero pose with halo
  ctx.save();ctx.translate(cx,cy+Math.sin(wt*.06)*5);const vs=1.8*eob((wt-4)/24);ctx.scale(vs,vs);drawBot();
  ctx.strokeStyle='#ffd23f';ctx.lineWidth=1.6;ctx.globalAlpha=.9;ctx.beginPath();ctx.ellipse(face,-28+Math.sin(wt*.1)*1.2,10,3,0,0,7);ctx.stroke();ctx.restore();
  // title: scrambles into place, then a rainbow of the five world colours flows through it
  const tt='GLITCHVERSE RESTORED';let s='';
  for(let i=0;i<tt.length;i++)s+=(tt[i]===' '||wt>10+i*3)?tt[i]:scr(wt,i);
  const gx=-360-((t*2.2)%720),gg=ctx.createLinearGradient(gx,0,gx+1440,0);for(let k=0;k<=10;k++)gg.addColorStop(k/10,WCOL[k%5]);
  ctx.save();ctx.translate(400,74);ctx.scale(eob(wt/24),eob(wt/24));
  if(wt<40||wt%150<6){ctx.globalCompositeOperation='lighter';tx(s,-4,0,44,'#ff2d6f');tx(s,4,0,44,'#2de2ff');ctx.globalCompositeOperation='source-over'}
  ctx.shadowColor='#8fd3ff';ctx.shadowBlur=24;tx(s,0,0,44,gg);ctx.restore();
  ctx.globalAlpha=clamp01((wt-60)/20);tx('The signal is clean again.',400,108,20,'#bdeaff');
  // results
  ctx.globalAlpha=clamp01((wt-90)/24);tx('FINAL SCORE',400,386,14,'#8a93c4');
  tx(String(Math.round(score*(1-Math.pow(1-clamp01((wt-96)/110),3)))),400,420,34,'#ffd23f');
  for(let i=0;i<lives;i++)heart(400+(i-(lives-1)/2)*26,445,8,'#ff3b5c','#ffffff');
  ctx.globalAlpha=1;
  if(wt>140){ctx.globalAlpha=.55+.45*Math.sin(wt*.08);tx('Tap or press Space to play again',400,470,16,'#ffd23f');ctx.globalAlpha=1}
  if(wt<16){ctx.globalAlpha=1-wt/16;ctx.fillStyle='#ffffff';ctx.fillRect(0,0,800,480);ctx.globalAlpha=1}
  if(wt<14)glitch(5);else if(wt>150&&wt%140<5)glitch(4);
  ctx.restore()}

load(0,1);st='menu';
let last=performance.now(),acc=0;
(function loop(now){
  now=now||performance.now();acc+=Math.min(100,now-last);last=now;
  while(acc>=1000/60){if(st==='play')update();else anim();stepFx();acc-=1000/60}
  draw();requestAnimationFrame(loop)})();
const map={' ':'jump',ArrowUp:'jump',w:'jump',g:'g',x:'x',f:'f',r:'restart'};
addEventListener('keydown',e=>{if(e.repeat)return;keys[e.key]=true;
  if(st==='story'&&(e.key==='Escape'||e.key==='s'||e.key==='S')){st=sn;sn='play';return}
  if((e.key==='n'||e.key==='N')&&st!=='play'&&st!=='between'&&st!=='story'&&st!=='dying'&&st!=='clear'&&st!=='cpfx'){fresh();return}
  if(st==='menu'&&/^[1-5]$/.test(e.key)){score=0;lives=5;load((e.key-1)*4);return}
  if(e.key==='Enter')act('jump');const a=map[e.key];
  if(a==='restart'&&st==='play'){load(li,1)}else if(a)act(a);if(e.key===' '||e.key.startsWith('Arrow'))e.preventDefault()});
addEventListener('keyup',e=>keys[e.key]=false);
cv.addEventListener('pointerdown',e=>{if(st==='play')return;const r=cv.getBoundingClientRect();if((st==='menu'||st==='over')&&cp.li>0&&(e.clientY-r.top)/r.height>.85){fresh();return}act('jump')});
document.querySelectorAll('button').forEach(b=>{
  b.addEventListener('pointerdown',e=>{e.preventDefault();if(b.dataset.h)keys[b.dataset.h]=true;else act(b.dataset.a)});
  ['pointerup','pointerleave','pointercancel'].forEach(t=>b.addEventListener(t,()=>{if(b.dataset.h)keys[b.dataset.h]=false}))});
</script>
</body>
</html># EXTC
volt Verse 
