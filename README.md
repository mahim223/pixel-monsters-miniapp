# pixel-monsters-miniapp
import React, { useState, useEffect, FC, useCallback } from 'react';
// Import the Farcaster Mini App SDK
// Make sure to install it: npm install @farcaster/miniapp-sdk
import Farcaster, { UserData } from '@farcaster/miniapp-sdk';

// --- 1. TYPE DEFINITIONS ---

type GameState = 'loading' | 'choosing' | 'battle' | 'won' | 'lost';

interface MonsterStats {
  maxHp: number;
  attack: number;
}

interface Monster {
  id: string;
  name: string;
  stats: MonsterStats;
  Sprite: FC<SpriteProps>;
}

interface SpriteProps {
  className?: string;
}

// --- 2. ORIGINAL MONSTER SPRITES ---
// We use inline, pixelated SVGs for an 8-bit feel without external files.

const commonSpriteStyles: React.CSSProperties = {
  imageRendering: 'pixelated',
  width: '96px',
  height: '96px',
};

// Grumble: A grumpy rock monster
const GrumbleSprite: FC<SpriteProps> = ({ className }) => (
  <svg viewBox="0 0 16 16" style={commonSpriteStyles} className={className}>
    <rect x="3" y="5" width="10" height="8" fill="#8B8B8B" />
    <rect x="4" y="6" width="8" height="6" fill="#ADADAD" />
    <rect x="6" y="8" width="1" height="1" fill="#222" />
    <rect x="9" y="8" width="1" height="1" fill="#222" />
    <rect x="5" y="13" width="2" height="1" fill="#8B8B8B" />
    <rect x="9" y="13" width="2" height="1" fill="#8B8B8B" />
  </svg>
);

// Sparklit: A small, fast-looking spark
const SparklitSprite: FC<SpriteProps> = ({ className }) => (
  <svg viewBox="0 0 16 16" style={commonSpriteStyles} className={className}>
    <rect x="7" y="4" width="2" height="1" fill="#FDE047" />
    <rect x="6" y="5" width="4" height="1" fill="#FDE047" />
    <rect x="5" y="6" width="6" height="4" fill="#FACC15" />
    <rect x="7" y="7" width="2" height="2" fill="#FFFFFF" />
    <rect x="4" y="7" width="1" height="2" fill="#FDE047" />
    <rect x="11" y="7" width="1" height="2" fill="#FDE047" />
    <rect x="6" y="10" width="4" height="1" fill="#FDE047" />
    <rect x="7" y="11" width="2" height="1" fill="#FDE047" />
  </svg>
);

// Fernling: A small sprout monster
const FernlingSprite: FC<SpriteProps> = ({ className }) => (
  <svg viewBox="0 0 16 16" style={commonSpriteStyles} className={className}>
    <rect x="7" y="4" width="2" height="5" fill="#4ADE80" />
    <rect x="6" y="6" width="1" height="2" fill="#22C55E" />
    <rect x="9" y="6" width="1" height="2" fill="#22C55E" />
    <rect x="5" y="7" width="1" height="2" fill="#4ADE80" />
    <rect x="10" y="7" width="1" height="2" fill="#4ADE80" />
    <rect x="6" y="9" width="4" height="3" fill="#654321" />
    <rect x="7" y="10" width="1" height="1" fill="#222" />
    <rect x="9" y="10" width="1" height="1" fill="#222" />
  </svg>
);

// Scamperat: A wild, rat-like creature
const ScamperatSprite: FC<SpriteProps> = ({ className }) => (
  <svg viewBox="0 0 16 16" style={commonSpriteStyles} className={className}>
    <rect x="4" y="6" width="8" height="6" fill="#94A3B8" />
    <rect x="5" y="7" width="6" height="4" fill="#E2E8F0" />
    <rect x="10" y="5" width="2" height="2" fill="#94A3B8" />
    <rect x="11" y="4" width="1" height="1" fill="#94A3B8" />
    <rect x="5" y="8" width="1" height="1" fill="#F87171" />
    <rect x="2" y="9" width="2" height="1" fill="#F472B6" />
    <rect x="5" y="12" width="2" height="1" fill="#94A3B8" />
    <rect x="9" y="12" width="2" height="1" fill="#94A3B8" />
  </svg>
);

// --- 3. MONSTER DATA ---

const STARTER_MONSTERS: Monster[] = [
  {
    id: 'grumble',
    name: 'Grumble',
    stats: { maxHp: 30, attack: 5 },
    Sprite: GrumbleSprite,
  },
  {
    id: 'sparklit',
    name: 'Sparklit',
    stats: { maxHp: 20, attack: 8 },
    Sprite: SparklitSprite,
  },
  {
    id: 'fernling',
    name: 'Fernling',
    stats: { maxHp: 25, attack: 6 },
    Sprite: FernlingSprite,
  },
];

const WILD_MONSTERS: Monster[] = [
  {
    id: 'scamperat',
    name: 'Scamperat',
    stats: { maxHp: 15, attack: 4 },
    Sprite: ScamperatSprite,
  },
  // We can add more wild monsters here
];

// --- 4. HELPER COMPONENTS ---

interface HPBarProps {
  current: number;
  max: number;
}

const HPBar: FC<HPBarProps> = ({ current, max }) => {
  const percentage = Math.max(0, (current / max) * 100);
  return (
    <div className="w-full h-4 bg-gray-700 border-2 border-black rounded-sm overflow-hidden">
      <div
        className="h-full bg-green-500 transition-all duration-500"
        style={{ width: `${percentage}%` }}
      />
    </div>
  );
};

interface RetroButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  children: React.ReactNode;
}

const RetroButton: FC<RetroButtonProps> = ({ children, className, ...props }) => (
  <button
    className={`bg-yellow-400 text-black font-bold py-3 px-6 rounded-lg border-2 border-black shadow-[4px_4px_0_0_#000] hover:bg-yellow-500 active:shadow-none active:translate-x-1 active:translate-y-1 transition-all disabled:bg-gray-400 disabled:shadow-none disabled:translate-x-0 disabled:translate-y-0 ${className}`}
    {...props}
  >
    {children}
  </button>
);

// --- 5. MAIN APP COMPONENT ---

const App: FC = () => {
  // --- STATE ---
  const [gameState, setGameState] = useState<GameState>('loading');
  const [farcasterUser, setFarcasterUser] = useState<UserData | null>(null);
  const [xp, setXp] = useState<number>(0);

  const [playerMonster, setPlayerMonster] = useState<Monster | null>(null);
  const [enemyMonster, setEnemyMonster] = useState<Monster | null>(null);
  const [playerHP, setPlayerHP] = useState<number>(0);
  const [enemyHP, setEnemyHP] = useState<number>(0);

  const [battleLog, setBattleLog] = useState<string[]>([]);
  const [isPlayerTurn, setIsPlayerTurn] = useState<boolean>(true);

  // --- SDK & LOCALSTORAGE INIT ---
  useEffect(() => {
    // Init Farcaster SDK
    Farcaster.init();
    
    // Get Farcaster user data
    Farcaster.getUser().then(setFarcasterUser).catch((err) => {
      console.warn('Farcaster user not found, running in standalone mode.', err);
    });

    // Load XP from LocalStorage
    try {
      const storedXp = localStorage.getItem('monstroclash_xp');
      setXp(storedXp ? parseInt(storedXp, 10) : 0);
    } catch (e) {
      console.warn('Could not read from localStorage.', e);
    }

    setGameState('choosing'); // Ready to start
  }, []);

  // --- GAME LOGIC ---

  const saveXP = (newXp: number) => {
    setXp(newXp);
    try {
      localStorage.setItem('monstroclash_xp', newXp.toString());
    } catch (e) {
      console.warn('Could not save to localStorage.', e);
    }
  };

  const selectMonster = (monster: Monster) => {
    setPlayerMonster(monster);
    setPlayerHP(monster.stats.maxHp);

    // Select a random wild monster
    const randomEnemy = WILD_MONSTERS[Math.floor(Math.random() * WILD_MONSTERS.length)];
    setEnemyMonster(randomEnemy);
    setEnemyHP(randomEnemy.stats.maxHp);

    setBattleLog([`A wild ${randomEnemy.name} appeared!`]);
    setIsPlayerTurn(true);
    setGameState('battle');
  };

  const playerAttack = useCallback(() => {
    if (!playerMonster || !enemyMonster || !isPlayerTurn) return;

    setIsPlayerTurn(false);
    const damage = playerMonster.stats.attack;
    const newEnemyHP = Math.max(0, enemyHP - damage);
    
    setBattleLog(prev => [...prev, `Your ${playerMonster.name} attacks for ${damage} damage!`]);
    setEnemyHP(newEnemyHP);

    if (newEnemyHP <= 0) {
      // Player wins
      setBattleLog(prev => [...prev, `Wild ${enemyMonster.name} fainted! You win!`]);
      saveXP(xp + 10);
      setTimeout(() => setGameState('won'), 1500);
    } else {
      // Enemy turn
      setTimeout(enemyAttack, 1000);
    }
  }, [playerMonster, enemyMonster, isPlayerTurn, enemyHP, xp]);

  const enemyAttack = useCallback(() => {
    if (!playerMonster || !enemyMonster) return;

    const damage = enemyMonster.stats.attack;
    const newPlayerHP = Math.max(0, playerHP - damage);

    setBattleLog(prev => [...prev, `Wild ${enemyMonster.name} attacks for ${damage} damage!`]);
    setPlayerHP(newPlayerHP);

    if (newPlayerHP <= 0) {
      // Player loses
      setBattleLog(prev => [...prev, `Your ${playerMonster.name} fainted! You lost...`]);
      setTimeout(() => setGameState('lost'), 1500);
    } else {
      setIsPlayerTurn(true);
    }
  }, [playerMonster, enemyMonster, playerHP]);

  const handleShare = () => {
    const text = `I just ${gameState === 'won' ? 'won' : 'finished'} a battle with my ${playerMonster?.name} in MonstroClash! My total XP is now ${xp}.`;
    
    // This will open the Farcaster client's cast composer
    Farcaster.cast(text);
  };

  const restartGame = () => {
    setPlayerMonster(null);
    setEnemyMonster(null);
    setPlayerHP(0);
    setEnemyHP(0);
    setBattleLog([]);
    setGameState('choosing');
  };

  // --- RENDER LOGIC ---

  const renderGame = () => {
    switch (gameState) {
      case 'loading':
        return <div className="text-center">Loading...</div>;

      case 'choosing':
        return (
          <div className="flex flex-col items-center">
            <h1 className="text-3xl font-bold mb-2">MonstroClash</h1>
            <h2 className="text-xl mb-4">Choose your monster!</h2>
            <p className="mb-4">Total XP: {xp}</p>
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4 w-full">
              {STARTER_MONSTERS.map((monster) => (
                <div
                  key={monster.id}
                  className="p-4 bg-gray-800 border-2 border-black rounded-lg flex flex-col items-center"
                >
                  <monster.Sprite />
                  <h3 className="text-lg font-bold mt-2">{monster.name}</h3>
                  <p>HP: {monster.stats.maxHp}</p>
                  <p>ATK: {monster.stats.attack}</p>
                  <RetroButton
                    className="mt-4 w-full"
                    onClick={() => selectMonster(monster)}
                  >
                    Choose
                  </RetroButton>
                </div>
              ))}
            </div>
          </div>
        );

      case 'battle':
        if (!playerMonster || !enemyMonster) return null;
        return (
          <div className="flex flex-col h-full">
            {/* Battle Scene */}
            <div className="flex-grow flex flex-col justify-between">
              {/* Enemy Side */}
              <div className="flex flex-col items-end p-4">
                <enemyMonster.Sprite className="transform -scale-x-100" />
                <div className="w-48 text-right">
                  <h3 className="font-bold text-lg">{enemyMonster.name}</h3>
                  <HPBar current={enemyHP} max={enemyMonster.stats.maxHp} />
                  <p className="text-sm">HP: {enemyHP} / {enemyMonster.stats.maxHp}</p>
                </div>
              </div>
              
              {/* Player Side */}
              <div className="flex flex-col items-start p-4">
                <playerMonster.Sprite />
                <div className="w-48">
                  <h3 className="font-bold text-lg">{playerMonster.name}</h3>
                  <HPBar current={playerHP} max={playerMonster.stats.maxHp} />
                  <p className="text-sm">HP: {playerHP} / {playerMonster.stats.maxHp}</p>
                </div>
              </div>
            </div>

            {/* UI & Log */}
            <div className="bg-gray-800 border-t-4 border-black p-4">
              <div className="h-24 w-full bg-black text-green-400 font-mono p-2 rounded-md overflow-y-auto mb-4 border-2 border-gray-600">
                {battleLog.map((log, i) => (
                  <p key={i}>{log}</p>
                ))}
              </div>
              <RetroButton
                className="w-full"
                onClick={playerAttack}
                disabled={!isPlayerTurn}
              >
                Attack!
              </RetroButton>
            </div>
          </div>
        );

      case 'won':
      case 'lost':
        return (
          <div className="flex flex-col items-center justify-center h-full text-center">
            <h1 className="text-4xl font-bold mb-4">
              {gameState === 'won' ? 'You Won!' : 'You Lost...'}
            </h1>
            {gameState === 'won' && <p className="text-lg mb-4">You gained 10 XP!</p>}
            <p className="text-xl mb-6">Total XP: {xp}</p>
            
            <div className="flex flex-col gap-4 w-full max-w-xs">
              {farcasterUser && (
                <RetroButton onClick={handleShare}>
                  Share Result
                </RetroButton>
              )}
              <RetroButton onClick={restartGame}>
                Play Again
              </RetroButton>
            </div>
          </div>
        );
    }
  };

  return (
    <div className="bg-gray-900 text-white font-mono min-h-screen w-full">
      <div className="max-w-md mx-auto h-screen p-4">
        {renderGame()}
      </div>
    </div>
  );
};

export default App;
