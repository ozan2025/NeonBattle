# NeonBattle - P2P Battleship PWA

## Project Overview
A fully peer-to-peer Battleship game as a Progressive Web App. Two players connect via WebRTC DataChannels using PeerJS and play in real-time. No backend server, no accounts. Static files hosted on GitHub Pages.

## Deployment
- **Repo**: https://github.com/ozan2025/NeonBattle
- **Live URL**: https://ozan2025.github.io/NeonBattle/
- **Current version**: v6
- Deploy: `git push`, then wait ~30s and check `gh api repos/ozan2025/NeonBattle/pages/builds/latest` for `"status":"built"`. Only manually trigger with `gh api repos/ozan2025/NeonBattle/pages/builds -X POST` if auto-deploy fails (don't trigger both — they race and cancel each other).
- On each deploy: bump version in title screen HTML AND `CACHE_NAME` in `sw.js`

## Architecture
- **Single-file app**: All CSS and JS inline in `index.html` (~70KB)
- **No external JS** except PeerJS (CDN)
- **No audio files** - Web Audio API only
- **Canvas 2D** for game grids and particle effects
- **PeerJS** for WebRTC signaling via free cloud server (`0.peerjs.com`)
- **Service worker**: Network-first for all assets (sw.js)

## File Structure
```
/
├── .claude/CLAUDE.md    (this file)
├── index.html           (complete game - all CSS/JS inline)
├── manifest.json        (PWA manifest)
├── sw.js                (service worker - network-first)
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── MainPrompt.md        (original spec/requirements)
```

## Key Technical Details

### Networking
- PeerJS peer IDs prefixed with `NB-` (e.g., `NB-XAEG` for host)
- Guest IDs: `NB-{code}-G-{random}`
- Room codes: 4 uppercase letters from `ABCDEFGHJKLMNPQRSTUVWXYZ` (no I/O)
- All game data flows as JSON via DataChannel after connection

### Message Protocol
- `{ type: 'ready' }` - Ship placement complete
- `{ type: 'attack', row, col }` - Fire at position
- `{ type: 'result', row, col, hit, sunk, cells }` - Attack result
- `{ type: 'chat', emoji }` - Emoji reaction
- `{ type: 'rematch', accept }` - Rematch request
- `{ type: 'sync' / 'sync-ack' }` - Reconnection sync

### Game Flow
1. Title screen → Create/Join game
2. PeerJS connection via room code
3. Ship placement (5 ships: Carrier/Battleship/Cruiser/Submarine/Destroyer)
4. Battle phase (host goes first, hit = extra turn, miss = pass)
5. Game over (all 5 ships sunk) → Stats + Rematch option

### Turn Management
- **Hit = extra turn**: attacker keeps firing on hits
- **Miss = turn passes**: `isMyTurn` flips to opponent
- Attacker fires → `isMyTurn = false` → waits for result
- On result: hit → `isMyTurn = true` (fire again), miss → `isMyTurn = false`
- Defender: hit → `isMyTurn = false` (opponent fires again), miss → `isMyTurn = true`

### Visual Style
- Dark ocean background (#0a0e1a) with twinkling stars + animated wave layers
- Cyan/teal for own board, magenta/pink for enemy board
- Fonts: Orbitron (titles), Rajdhani (body)
- Particle system for hit/miss/sunk effects (max 150 particles)
- Title entrance animation, button glow pulses, production credit neon glow
- Victory: pulsing gold glow; Defeat: shake animation

## iOS Safari Considerations
- `touch-action: manipulation` on interactive elements
- `overscroll-behavior: none` to prevent pull-to-refresh
- Custom A-Z keyboard for room code (avoids iOS virtual keyboard)
- `visualViewport.height` for accurate viewport on Safari (URL bar)
- `visibilitychange` + `focus` events for reconnection
- No `beforeunload` (unreliable on Safari)
- Compact battle UI elements for iPhone vertical space

## Development
```bash
# Serve locally
cd /Users/ozanselcuk/2026_Developer/P2PWebRTCBattleship
python3 -m http.server 8081

# Test with two browser tabs on localhost:8081
```

## Known Issues / Past Bugs
- **SW caching**: Now network-first. Bump `CACHE_NAME` in sw.js on each deploy.
- **Grid DPR scaling**: `getGridMetrics` must use logical canvas dimensions (`_logicW`/`_logicH`).
- **iPhone battle overflow**: Uses `visualViewport.height` + compact UI. May still need tweaking based on real device testing.

## TODO
- Continue real-device iPhone testing (user testing v4)
- Tweak battle screen fit if needed based on user feedback

## Rules
- Keep everything in a single `index.html` (inline CSS/JS)
- No external libraries except PeerJS
- No audio files - Web Audio API synthesis only
- Each player is authoritative about their own board (never trust opponent)
- Never send ship positions - only attack results
