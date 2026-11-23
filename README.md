# LinkedIn Game Cheats

> Because spending 45 seconds on a daily puzzle is 40 seconds too many.

A collection of zero-dependency JavaScript bookmarklets that solve LinkedIn's daily mini-games straight in your browser. No extensions, no sketchy third-party APIs, no Python backends. Just pure DOM scraping, constraint backtracking, and synthetic pointer events.

Each solver includes a **5-second anti-cheat delay** right before the final move so LinkedIn's server doesn't flag you for superhuman 0-second runs and still puts you at the top of your network leaderboard.

---

## The Arsenal

| Game | Codename | Status | Algorithm |
| :--- | :--- | :--- | :--- |
| **Queens** | `Queen Solver` | Works | N-Queens Backtracker + Region Constraints |
| **Sumbreak** | `Sumbreaker` | Works | BFS State-Space Search |
| **Mini-Sudoku** | `Sudokill` | Works | 6x6 Backtracker + Iframe Traversal |
| **Patches** | `Patchwork` | Beta | Shape/Area Backtracker + Multi-Step Drag |
| **Zip** | `Zipextract` | Beta | Hamiltonian Path + Classless Wall Detection |

---

## Quick Start (Bookmarklets)

1. Create a new bookmark in your browser (Chrome, Brave, Firefox).
2. Name it whatever makes you feel like a 1337 hacker (e.g., `Queen Solver`).
3. Paste the corresponding `javascript:(...)()` one-liner into the **URL** field.
4. Open the LinkedIn game, click the bookmark, wait ~5 seconds, and collect your crown.

---

### 1. Queen Solver (Queens)
Solves any `NxN` Queens board by reading cell colors and firing two separate clicks per cell (since LinkedIn ignores synthetic `dblclick` events).

```javascript
javascript:(async()=>{const e=Array.from(document.querySelectorAll("div[data-cell-idx]")).map(e=>({idx:parseInt(e.getAttribute("data-cell-idx"),10),color:window.getComputedStyle(e).backgroundColor,el:e})).sort((e,t)=>e.idx-t.idx);if(!e.length)return alert("Kein Queens-Board!");const t=Math.round(Math.sqrt(e.length)),l={},n=e.map(e=>(void 0===l[e.color]&&(l[e.color]=Object.keys(l).length),l[e.color])),o=[];!function e(l,r,c,a){if(l===t)return o.push([...a]),!0;for(let u=0;u<t;u++){if(r.has(u))continue;const s=n[l*t+u];if(c.has(s))continue;if(l>0&&Math.abs(a[l-1]-u)<=1)continue;if(r.add(u),c.add(s),a.push(u),e(l+1,r,c,a))return!0;r.delete(u),c.delete(s),a.pop()}return!1}(0,new Set,new Set,[]);if(!o.length)return alert("Keine Lösung!");const c=t=>{t.dispatchEvent(new PointerEvent("pointerdown",{bubbles:!0})),t.dispatchEvent(new MouseEvent("mousedown",{bubbles:!0})),t.dispatchEvent(new PointerEvent("pointerup",{bubbles:!0})),t.dispatchEvent(new MouseEvent("mouseup",{bubbles:!0})),t.dispatchEvent(new MouseEvent("click",{bubbles:!0}))};const p=o[0].map((e,l)=>l*t+e);for(let i=0;i<p.length;i++){if(i===p.length-1)await new Promise(r=>setTimeout(r,5e3));const el=e[p[i]].el;c(el);await new Promise(r=>setTimeout(r,50));c(el);await new Promise(r=>setTimeout(r,70))}})()
```

### 2. Sumbreaker (Sumbreak)
Finds the largest `<svg>` with numbers, clusters coordinates into a grid, and runs a BFS to merge everything down to a single tile.

```javascript
javascript:(async()=>{const svgs=[...document.querySelectorAll('svg')].map(s=>({s,t:[...s.querySelectorAll('text')].filter(e=>/^\d+$/.test((e.textContent||'').trim())&&e.getBoundingClientRect().width>0)})).sort((a,b)=>b.t.length-a.t.length);if(!svgs.length||!svgs[0].t.length)return alert('Kein Sumbreak-Board gefunden!');const svg=svgs[0].s,texts=svgs[0].t;const items=texts.map(el=>{const r=el.getBoundingClientRect();return{val:parseInt(el.textContent.trim(),10),x:r.left+r.width/2,y:r.top+r.height/2};});const cluster=(vals,tol=15)=>{const s=[...vals].sort((a,b)=>a-b);const g=[];for(const v of s)if(!g.length||v-g[g.length-1]>tol)g.push(v);return g;};const xs=cluster(items.map(i=>i.x)),ys=cluster(items.map(i=>i.y));const rows=ys.length,cols=xs.length;const grid=Array.from({length:rows},()=>Array(cols).fill(null));const coords=Array.from({length:rows},()=>Array(cols).fill(null));const nearest=(a,v)=>a.reduce((b,c,i)=>Math.abs(c-v)<Math.abs(a[b]-v)?i:b,0);for(const it of items){const r=nearest(ys,it.y),c=nearest(xs,it.x);grid[r][c]=it.val;coords[r][c]={x:it.x,y:it.y};}for(let r=0;r<rows;r++)for(let c=0;c<cols;c++)if(!coords[r][c])coords[r][c]={x:xs[c],y:ys[r]};const dirs=[[0,1],[1,0],[0,-1],[-1,0]];const bfs=()=>{const q=[{g:grid.map(r=>[...r]),cnt:items.length,m:[]}];const vis=new Set([grid.flat().join(',')]);while(q.length){const{g,cnt,m}=q.shift();if(cnt===1)return m;for(let r=0;r<rows;r++)for(let c=0;c<cols;c++){const v=g[r][c];if(v===null)continue;for(const[dr,dc]of dirs){const nr=r+dr,nc=c+dc;if(nr<0||nr>=rows||nc<0||nc>=cols)continue;const tv=g[nr][nc];if(tv===v||tv===null){const ng=g.map(row=>[...row]);ng[r][c]=null;ng[nr][nc]=tv===v?v*2:v;const k=ng.flat().join(',');if(!vis.has(k)){vis.add(k);q.push({g:ng,cnt:tv===v?cnt-1:cnt,m:[...m,[r,c,nr,nc]]});}}}}}return null;};const moves=bfs();if(!moves)return alert('Keine Lösung gefunden!');const fire=(type,x,y)=>{(document.elementFromPoint(x,y)||svg).dispatchEvent(new PointerEvent(type,{bubbles:true,cancelable:true,view:window,clientX:x,clientY:y,pointerId:1,pointerType:'mouse',isPrimary:true,buttons:type==='pointerup'?0:1}));};for(let i=0;i<moves.length;i++){if(i===moves.length-1)await new Promise(r=>setTimeout(r,5e3));const[r,c,nr,nc]=moves[i];const p1=coords[r][c],p2=coords[nr][nc];fire('pointerdown',p1.x,p1.y);await new Promise(r=>setTimeout(r,30));fire('pointermove',p2.x,p2.y);await new Promise(r=>setTimeout(r,30));fire('pointerup',p2.x,p2.y);await new Promise(r=>setTimeout(r,60));}})()
```

### 3. Sudokill (Mini-Sudoku 6x6)
Searches both the main document and any same-origin `iframe` for the 6x6 board, solves it, and clicks each cell plus its number button with real coordinates.

```javascript
javascript:(async()=>{let c=[];for(const d of[document,...[...document.querySelectorAll('iframe')].map(f=>{try{return f.contentDocument}catch{}}).filter(Boolean)]){const x=[...d.querySelectorAll('.sudoku-cell-content')];if(x.length==36){c=x;break}}if(!c.length)return alert('No 6x6!');const D=c[0].ownerDocument,g=Array.from({length:6},(_,r)=>Array.from({length:6},(_,x)=>+c[r*6+x].textContent.trim()||0)),o=g.map(r=>[...r]),ok=(r,x,v)=>{for(let i=0;i<6;i++)if(g[r][i]==v||g[i][x]==v)return 0;for(let a=0;a<2;a++)for(let b=0;b<3;b++)if(g[(r/2|0)*2+a][(x/3|0)*3+b]==v)return 0;return 1},s=()=>{for(let r=0;r<6;r++)for(let x=0;x<6;x++)if(!g[r][x]){for(let v=1;v<7;v++)if(ok(r,x,v)){g[r][x]=v;if(s())return 1;g[r][x]=0}return 0}return 1};if(!s())return alert('No sol!');const m=[];for(let r=0;r<6;r++)for(let x=0;x<6;x++)if(!o[r][x])m.push([r,x,g[r][x]]);const b={};for(let v=1;v<7;v++)b[v]=[...D.querySelectorAll('button')].find(e=>e.textContent.trim()==v&&e.offsetWidth);const tap=e=>{const r=e.getBoundingClientRect(),p={bubbles:1,cancelable:1,view:D.defaultView,clientX:r.left+r.width/2,clientY:r.top+r.height/2};['pointerdown','mousedown','pointerup','mouseup','click'].forEach(t=>e.dispatchEvent(new(t[0]=='p'?PointerEvent:MouseEvent)(t,p)))};for(let i=0;i<m.length;i++){if(i==m.length-1)await new Promise(r=>setTimeout(r,5e3));const[r,x,v]=m[i];tap(c[r*6+x].parentElement);await new Promise(r=>setTimeout(r,30));b[v]?tap(b[v]):D.dispatchEvent(new KeyboardEvent('keydown',{key:''+v,code:'Digit'+v,keyCode:48+v,bubbles:1}));await new Promise(r=>setTimeout(r,50))}})()
```

### 4. Patchwork (Patches)
Parses shape hints from `aria-label` (wide, tall, square, freeform) and drags each rectangle in 4 intermediate steps so long bars don't skip cells.

```javascript
javascript:(async()=>{const c=[...document.querySelectorAll('[data-cell-idx]')].sort((a,b)=>a.dataset.cellIdx-b.dataset.cellIdx);if(!c.length)return alert('Kein Patches-Board!');const N=Math.round(Math.sqrt(c.length)),cl=[],vis=new Uint8Array(c.length);c.forEach((e,i)=>{const v=+e.textContent.trim()||0,l=e.getAttribute('aria-label')||'';if(v>0||/Hinweis:/i.test(l))cl.push({i,r:i/N|0,x:i%N,v,t:/breit/i.test(l)?'W':/hoch|hohe/i.test(l)?'H':/quadrat/i.test(l)?'S':'F'})});cl.sort((a,b)=>(b.v>0)-(a.v>0));const rects=cl.map(k=>{const a=[];for(let r1=0;r1<=k.r;r1++)for(let r2=k.r;r2<N;r2++)for(let x1=0;x1<=k.x;x1++)for(let x2=k.x;x2<N;x2++){const h=r2-r1+1,w=x2-x1+1;if(k.v>0&&h*w!=k.v)continue;if(k.t=='W'&&w<=h||k.t=='H'&&h<=w||k.t=='S'&&w!=h)continue;if(cl.some(o=>o.i!=k.i&&o.r>=r1&&o.r<=r2&&o.x>=x1&&o.x<=x2))continue;a.push([r1,x1,r2,x2])}return a});const sol=[];const s=idx=>{if(idx==cl.length)return vis.every(Boolean);for(const[r1,x1,r2,x2]of rects[idx]){let ok=1;for(let r=r1;r<=r2&&ok;r++)for(let x=x1;x<=x2;x++)if(vis[r*N+x])ok=0;if(!ok)continue;for(let r=r1;r<=r2;r++)for(let x=x1;x<=x2;x++)vis[r*N+x]=1;sol.push([r1,x1,r2,x2]);if(s(idx+1))return 1;sol.pop();for(let r=r1;r<=r2;r++)for(let x=x1;x<=x2;x++)vis[r*N+x]=0}return 0};if(!s(0))return alert('Keine Lösung!');const pos=e=>{const r=e.getBoundingClientRect();return[r.left+r.width/2,r.top+r.height/2]};const fire=(pt,mt,x,y)=>{const t=document.elementFromPoint(x,y)||document.body,o={bubbles:1,cancelable:1,view:window,clientX:x,clientY:y,pointerId:1,pointerType:'mouse',isPrimary:1,buttons:pt=='pointerup'?0:1};t.dispatchEvent(new PointerEvent(pt,o));t.dispatchEvent(new MouseEvent(mt,o))};for(let i=0;i<sol.length;i++){if(i==sol.length-1)await new Promise(r=>setTimeout(r,5e3));const[r1,x1,r2,x2]=sol[i],[ax,ay]=pos(c[r1*N+x1]),[bx,by]=pos(c[r2*N+x2]);fire('pointerdown','mousedown',ax,ay);for(let k=1;k<=4;k++){await new Promise(r=>setTimeout(r,35));fire('pointermove','mousemove',ax+(bx-ax)*k/4,ay+(by-ay)*k/4)}await new Promise(r=>setTimeout(r,35));fire('pointerup','mouseup',bx,by);await new Promise(r=>setTimeout(r,120))}})()
```

### 5. Zipextract (Zip)
Classless wall detection (scans computed borders > 2px across all child elements and pseudo-elements), solves the Hamiltonian path in strict number order, and clicks each cell sequentially.

```javascript
javascript:(async()=>{const cells=[...document.querySelectorAll('[data-cell-idx]')].sort((a,b)=>a.dataset.cellIdx-b.dataset.cellIdx);if(!cells.length)return alert('Kein Zip-Board!');const N=Math.round(Math.sqrt(cells.length)),nums=new Int32Array(cells.length);let maxNum=0,startIdx=-1;cells.forEach((c,i)=>{const t=(c.innerText||c.textContent||'').replace(/\s+/g,'');if(/^\d+$/.test(t)){const v=+t;nums[i]=v;if(v===1)startIdx=i;if(v>maxNum)maxNum=v}});const walls=cells.map(c=>{const w={u:0,d:0,l:0,r:0};for(const el of[c,...c.querySelectorAll('*')]){for(const p of[null,'::after','::before']){const s=getComputedStyle(el,p);if(p&&s.content==='none')continue;if(parseFloat(s.borderTopWidth)>2)w.u=1;if(parseFloat(s.borderBottomWidth)>2)w.d=1;if(parseFloat(s.borderLeftWidth)>2)w.l=1;if(parseFloat(s.borderRightWidth)>2)w.r=1}}return w});const vis=new Uint8Array(cells.length),path=[];const solve=(idx,next)=>{vis[idx]=1;path.push(idx);if(path.length===cells.length&&next>maxNum)return 1;const r=idx/N|0,c=idx%N,dirs=[[r-1,c,'u','d'],[r+1,c,'d','u'],[r,c-1,'l','r'],[r,c+1,'r','l']];for(const[nr,nc,d1,d2]of dirs){if(nr<0||nr>=N||nc<0||nc>=N)continue;const ni=nr*N+nc;if(vis[ni]||walls[idx][d1]||walls[ni][d2])continue;const v=nums[ni];if(v>0&&v!==next)continue;if(solve(ni,v===next?next+1:next))return 1}vis[idx]=0;path.pop();return 0};if(!solve(startIdx,2))return alert('Keine Lösung!');const pos=e=>{const r=e.getBoundingClientRect();return[r.left+r.width/2,r.top+r.height/2]};const clk=el=>{const[x,y]=pos(el),o={bubbles:1,cancelable:1,view:window,clientX:x,clientY:y,pointerId:1,pointerType:'mouse',isPrimary:1,buttons:1};el.dispatchEvent(new PointerEvent('pointerdown',o));el.dispatchEvent(new MouseEvent('mousedown',o));el.dispatchEvent(new PointerEvent('pointerup',{...o,buttons:0}));el.dispatchEvent(new MouseEvent('mouseup',{...o,buttons:0}));el.dispatchEvent(new MouseEvent('click',{...o,buttons:0}))};for(let i=0;i<path.length;i++){if(i===path.length-1)await new Promise(r=>setTimeout(r,5e3));clk(cells[path[i]]);await new Promise(r=>setTimeout(r,60))}})()
```

---

## Anti-Cheat & Troubleshooting

If you ever trigger a 0-second lock or break the board state, open DevTools (`F12` -> Console) and run:
```javascript
localStorage.clear(); sessionStorage.clear(); location.reload();
```
Your LinkedIn login (`li_at` cookie) stays intact.

## Disclaimer
For educational purposes and flexing on your LinkedIn connections. Use at your own risk.
