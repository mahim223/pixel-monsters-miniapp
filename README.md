// Project file/folder structure /* farcaster-pixel-monsters/ ├─ public/ │  └─ farcaster.json ├─ src/ │  ├─ App.tsx              # Main React component (export default) │  ├─ index.tsx │  ├─ styles.css          # Retro pixel CSS │  └─ sprites.tsx         # Small helper to render pixel sprites ├─ package.json └─ README.md */

// ----------------------------- // public/farcaster.json // ----------------------------- /* { "name": "Pixel Monsters", "slug": "pixel-monsters-miniapp", "description": "A tiny pixel-art monster battle mini-game. Choose a monster, fight wild monsters, and share results to Farcaster.", "version": "1.0.0", "author": "YourName", "entry": "/index.html", "icons": { "48": "/icon-48.png" }, "permissions": ["cast"] } */

// ----------------------------- // src/sprites.tsx // Small helpers to draw pixel sprites as inline SVG (8x8 grid) so the art is original and tiny. // ----------------------------- import React from 'react';

type SpriteProps = { pixels: string[]; scale?: number; className?: string; alt?: string };

export const PixelSprite: React.FC<SpriteProps> = ({ pixels, scale = 10, className, alt }) => { // pixels: array of 8 strings each with 8 chars, where each char is a hex digit key into palette const palette: Record<string, string> = { "0": "#00000000", // transparent "1": "#2b2b2b", "2": "#f8f8f8", "3": "#ffcc00", "4": "#ff6b4a", "5": "#8ce07b", "6": "#4ab3ff", "7": "#d98cff", "8": "#ff8e3b", };

const size = 8 * scale;

return ( <svg className={className} width={size} height={size} viewBox={0 0 ${8} ${8}} style={{ imageRendering: 'pixelated' }} role="img" aria-label={alt} > {pixels.map((row, y) => row.split("").map((c, x) => { const fill = palette[c] ?? palette['1']; if (fill === '#00000000') return null; return <rect key={${x}-${y}} x={x} y={y} width={1} height={1} fill={fill} />; }) )} </svg> ); };

// Example monster sprites (completely original) export const SPARKLIT = [ '00003300', '00133330', '01332210', '01333310', '01332210', '00133330', '00003300', '00000000', ];

export const MAGMAFIN = [ '00004400', '00444440', '04444444', '04454444', '04444444', '00444440', '00004400', '00000000', ];

export const LEAFAWN = [ '00005500', '00555550', '05575550', '05555550', '05575550', '00555550', '00005500', '00000000', ];

export const WILD1 = [ '00006600', '00666660', '06666660', '06676660', '06666660', '00666660', '00006600', '00000000', ];

// ----------------------------- // src/styles.css // ----------------------------- export const styles = :root{ --bg:#0b1020; --panel:#0f1724; --accent:#7dd3fc; --muted:#9ca3af; } *{box-sizing:border-box} html,body,#root{height:100%} body{ margin:0; background:linear-gradient(180deg,var(--bg),#02040a); color:#e6eef8; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, 'Roboto Mono', 'Courier New', monospace; -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale; } .app{ max-width:480px; margin:0 auto; padding:12px; } .header{ display:flex;align-items:center;gap:8px;margin-bottom:10px; } .title{font-size:16px;font-weight:700;letter-spacing:1px} .panel{ background:linear-gradient(180deg,#0b1320,#071226); border:4px solid #111827; padding:10px;border-radius:8px;box-shadow: 0 4px 0 #000 inset; } .row{display:flex;gap:8px;align-items:center} .center{justify-content:center} .monster-card{flex:1;background:#071026;padding:8px;border-radius:6px;border:2px solid #0b1220;display:flex;gap:8px;align-items:center} .stat{font-size:12px;color:var(--muted)} .controls{display:flex;gap:8px;margin-top:10px} .button{flex:1;padding:12px;border-radius:6px;border:3px solid #081224;background:linear-gradient(180deg,#132033,#091623);text-align:center;font-weight:700} .button:active{transform:translateY(1px)} .pixel-sprite{width:72px;height:72px} .log{height:80px;overflow:auto;font-size:12px;padding:6px;background:#020812;border-radius:6px;margin-top:8px;border:1px solid #041126} .small{font-size:12px;color:var(--muted)} .footer{margin-top:12px;display:flex;gap:8px;align-items:center;justify-content:space-between} .share{padding:8px;border-radius:6px;background:#0b2b2a;border:2px solid #063636};

// ----------------------------- // src/App.tsx  (Main React component with battle logic) // ----------------------------- import React, { useEffect, useState } from 'react'; import { PixelSprite, SPARKLIT, MAGMAFIN, LEAFAWN, WILD1, } from './sprites'; // styles imported as string above; in a real project, put into styles.css and import normally

// Types type Monster = { id: string; name: string; hp: number; atk: number; sprite: string[]; };

const PLAYER_MONSTERS: Monster[] = [ { id: 'sparklit', name: 'Sparklit', hp: 30, atk: 6, sprite: SPARKLIT }, { id: 'magmafin', name: 'Magmafin', hp: 36, atk: 5, sprite: MAGMAFIN }, { id: 'leafawn', name: 'Leafawn', hp: 26, atk: 7, sprite: LEAFAWN }, ];

const makeWild = (): Monster => { // create random wild monster stat and slightly different sprite const base = WILD1; const rand = Math.random(); const hp = 20 + Math.floor(rand * 20); const atk = 4 + Math.floor(rand * 5); return { id: 'wild-' + Date.now(), name: Wild-${['Gnarl','Bram','Puff'][Math.floor(Math.random()*3)]}, hp, atk, sprite: base }; };

// LocalStorage keys const XP_KEY = 'pm_xp'; const BEST_KEY = 'pm_best_score';

export default function App(): JSX.Element { const [selected, setSelected] = useState<Monster | null>(null); const [wild, setWild] = useState<Monster | null>(null); const [playerHp, setPlayerHp] = useState(0); const [wildHp, setWildHp] = useState(0); const [log, setLog] = useState<string[]>([]); const [xp, setXp] = useState<number>(() => Number(localStorage.getItem(XP_KEY) || '0')); const [best, setBest] = useState<number>(() => Number(localStorage.getItem(BEST_KEY) || '0')); const [busy, setBusy] = useState(false);

useEffect(() => { localStorage.setItem(XP_KEY, String(xp)); }, [xp]); useEffect(() => { localStorage.setItem(BEST_KEY, String(best)); }, [best]);

useEffect(() => { // just insert CSS into head for this single-file example const el = document.createElement('style'); el.textContent = styles; document.head.appendChild(el); return () => { document.head.removeChild(el); }; }, []);

const startBattle = (m: Monster) => { setSelected(m); const w = makeWild(); setWild(w); setPlayerHp(m.hp); setWildHp(w.hp); setLog([You chose ${m.name}! Wild ${w.name} appeared!]); };

const append = (s: string) => setLog(prev => [s, ...prev].slice(0, 10));

const playerAttack = () => { if (!selected || !wild || busy) return; setBusy(true); // player deals damage = atk + rand(-1..+2) const dmg = Math.max(1, selected.atk + Math.floor(Math.random() * 4) - 1); const newWildHp = Math.max(0, wildHp - dmg); setWildHp(newWildHp); append(${selected.name} hits ${wild.name} for ${dmg} damage!);

setTimeout(() => {
  if (newWildHp <= 0) {
    // win
    append(`You defeated ${wild.name}! +10 XP`);
    const newXp = xp + 10;
    setXp(newXp);
    const newBest = Math.max(best, newXp);
    setBest(newBest);
    setBusy(false);
    // start a new wild after short delay
    setTimeout(() => {
      const next = makeWild();
      setWild(next);
      setWildHp(next.hp);
      setPlayerHp(selected.hp);
      append(`A new wild ${next.name} appears!`);
    }, 800);
  } else {
    // wild attacks
    const wdmg = Math.max(1, wild.atk + Math.floor(Math.random() * 3) - 1);
    const newPlayerHp = Math.max(0, playerHp - wdmg);
    setTimeout(() => {
      setPlayerHp(newPlayerHp);
      append(`${wild.name} strikes back for ${wdmg} damage!`);

      if (newPlayerHp <= 0) {
        append(`${selected.name} fainted! Game over.`);
        setBusy(false);
        // reset selection to allow replay
        setSelected(null);
        setWild(null);
      } else {
        setBusy(false);
      }
    }, 500);
  }
}, 300);

};

const flee = () => { if (!selected || !wild) return; append(You fled from ${wild.name}.); setSelected(null); setWild(null); };

const shareResult = async () => { // Compose a simple share message and try to call Farcaster mini app SDK. const message = ${selected ? selected.name : 'Player'} battled on Pixel Monsters — XP ${xp}.;

try {
  // try dynamic import so code still builds without the SDK installed locally
  const sdk: any = await import('@farcaster/miniapp-sdk');
  // hypothetical client creation — adapt if your SDK differs
  const client = sdk?.createClient?.() ?? sdk?.default?.createClient?.();
  if (client && typeof client.cast === 'function') {
    await client.cast({ text: message });
    append('Shared to Farcaster!');
    return;
  }
} catch (e) {
  // ignore dynamic import failure — fallback to window bridge
}

// fallback: if Farcaster object exists on window (hosts often provide a bridge)
const win: any = window as any;
if (win?.farcaster?.postCast) {
  try {
    await win.farcaster.postCast({ text: message });
    append('Shared to Farcaster (bridge)!');
    return;
  } catch (e) {
    // continue to fallback
  }
}

append('Share failed: Farcaster SDK not found in this environment.');
alert('Cannot share: Farcaster SDK not available in this demo environment.');

};

return ( <div className="app"> <div className="header"> <div className="title">Pixel Monsters — Mini Battle</div> <div className="small">XP: {xp} • Best: {best}</div> </div>

<div className="panel">
    {!selected && (
      <div>
        <div className="row">
          {PLAYER_MONSTERS.map(m => (
            <div key={m.id} style={{flex:1}}>
              <div className="monster-card">
                <PixelSprite pixels={m.sprite} scale={9} className="pixel-sprite" alt={m.name} />
                <div>
                  <div style={{fontWeight:800}}>{m.name}</div>
                  <div className="stat">HP {m.hp} • ATK {m.atk}</div>
                  <div style={{marginTop:8}}>
                    <button className="button" onClick={() => startBattle(m)}>Choose</button>
                  </div>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    )}

    {selected && wild && (
      <div>
        <div className="row" style={{marginBottom:8}}>
          <div className="monster-card">
            <div style={{textAlign:'center'}}>
              <PixelSprite pixels={selected.sprite} scale={9} className="pixel-sprite" alt={selected.name} />
              <div style={{fontWeight:800}}>{selected.name}</div>
              <div className="stat">HP {playerHp}/{selected.hp}</div>
            </div>
          </div>

          <div className="monster-card">
            <div style={{textAlign:'center'}}>
              <PixelSprite pixels={wild.sprite} scale={9} className="pixel-sprite" alt={wild.name} />
              <div style={{fontWeight:800}}>{wild.name}</div>
              <div className="stat">HP {wildHp}/{wild.hp}</div>
            </div>
          </div>
        </div>

        <div className="controls">
          <button className="button" onClick={playerAttack} disabled={busy}>Attack</button>
          <button className="button" onClick={flee} disabled={busy}>Flee</button>
        </div>

        <div className="log">
          {log.map((l,i) => <div key={i}>{l}</div>)}
        </div>
      </div>
    )}

    {!selected && (
      <div className="footer">
        <div className="small">Tap a monster to start a battle — mobile-first controls.</div>
        <div>
          <button className="share button" onClick={shareResult}>Share result</button>
        </div>
      </div>
    )}
  </div>

</div>

); }

// ----------------------------- // src/index.tsx (entry) // ----------------------------- /* import React from 'react'; import { createRoot } from 'react-dom/client'; import App from './App';

const root = createRoot(document.getElementById('root')!); root.render(<App />); */

// ----------------------------- // package.json (essential deps) // ----------------------------- /* { "name": "pixel-monsters-miniapp", "version": "1.0.0", "private": true, "dependencies": { "react": "^18.2.0", "react-dom": "^18.2.0", "@farcaster/miniapp-sdk": "^1.0.0" }, "devDependencies": { "typescript": "^5.0.0", "vite": "^5.0.0" }, "scripts": { "dev": "vite", "build": "vite build", "preview": "vite preview" } } */

// ----------------------------- // README.md (short instructions) // ----------------------------- /*
