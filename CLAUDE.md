# TANK BATTLE — Project Context for Claude Code

## เริ่มต้นบนเครื่องใหม่

```bash
# 1. Clone repo
git clone https://github.com/p2544/test.git tank
cd tank

# 2. เปิด Claude Code
claude

# 3. เล่นเกมได้ทันที — เปิด index.html ในเบราว์เซอร์
```

> ไม่ต้องติดตั้ง Node.js / Python — เกมเป็น single HTML file ทำงานได้ทันที

---

## ข้อมูล Repository

| รายการ | ค่า |
|--------|-----|
| GitHub URL | https://github.com/p2544/test |
| Branch หลัก | `main` |
| ไฟล์หลัก | `index.html` |
| Git user | p2544 |
| Git email | piriya2544@gmail.com |

### Push ขึ้น GitHub
```bash
git add index.html
git commit -m "v1.X - description"
git push
```

### Local Preview Server (Claude Code)
ใช้ Perl จาก Git Bash (ไม่มี Node/Python บนเครื่องเดิม):
- config: `.claude/launch.json`
- server script: `serve.pl`
- port: 8080
- รัน: `perl serve.pl` หรือผ่าน Preview Panel ใน Claude Code

---

## โครงสร้างเกม

### ไฟล์
```
index.html     — ทุกอย่างอยู่ในไฟล์เดียว (HTML + CSS + JS)
explosion.mp3  — เสียงระเบิดจริง (embed เป็น base64 ใน HTML แล้ว ไม่ต้องโหลดแยก)
CLAUDE.md      — ไฟล์นี้
```

### Constants สำคัญ
```javascript
CELL = 20        // ขนาด 1 grid cell (px)
COLS = ROWS = 26 // ขนาด map 26x26 cells
W = H = 520      // canvas 520x520 px
// รถถัง = 2x2 cells = 40x40 px
```

### Tile Types
```javascript
T.EMPTY = 0
T.BRICK = 1   // กำแพงอิฐ ทำลายได้
T.STEEL = 2   // เหล็ก ทำลายไม่ได้
T.WATER = 3   // น้ำ ผ่านไม่ได้
T.BUSH  = 4   // พุ่มไม้ ซ่อนตัวได้ (วาดทับสุด)
T.BASE  = 5   // ฐานทัพ ถูกทำลาย = game over
```

### Directions
```javascript
DIR = { UP:0, RIGHT:1, DOWN:2, LEFT:3 }
DX  = [0, 1, 0, -1]
DY  = [-1, 0, 1, 0]
```

---

## Game Logic สำคัญ

### Player
- Spawn position: `x: 4 * CELL, y: 24 * CELL` (col 4-5, row 24-25)
- เหตุผล: col 8 ทับกำแพงในแผนที่ — ย้ายมา col 4 ซึ่งโล่งทุก level
- Lives เริ่มต้น: **5 ชีวิต**
- Invincibility หลัง respawn: 180 frames

### Level Progression (สำคัญ!)
- ไป level ถัดไปได้ **ต่อเมื่อศัตรูหมดทุกตัวเท่านั้น**
- ใช้ flag `levelWon = true` เมื่อ `checkWin()` detect ศัตรูหมด
- ถ้า player ตายพอดีกับที่ศัตรูหมด → รอ respawn (1500ms) แล้วค่อยขึ้น Level Clear
- ถ้า `lives <= 0` → Game Over (ไม่ respawn)

```javascript
// Flow เมื่อ player ตาย:
loseLife()
  ├── lives <= 0  → showGameOver()
  └── lives > 0   → player = null → setTimeout 1500ms
                       ├── levelWon = true  → showLevelClear()
                       └── levelWon = false → player = createPlayer()
```

### Enemy Spawn
- Spawn points: col 0, col 12, col 24 (row 0)
- จำนวน enemies ต่อ level: `4 + level * 2`
- Active enemies สูงสุด: `min(4 + floor(level/2), 6)`
- Spawn delay: `max(60, 180 - level * 15)` frames

### Enemy Tiers
| Tier | สี | Speed | HP | คะแนน |
|------|----|----|-----|-------|
| 0 | แดง | 1.0 | 1 | 100 |
| 1 | ส้ม | 1.3 | 1 | 200 |
| 2 | ขาว | 1.8 | 2 | 300 |

### Powerups (18% chance เมื่อ enemy ตาย)
| Type | ผล |
|------|----|
| star | +speed, +bullet power |
| shield | invincible 600 frames |
| grenade | ระเบิดศัตรูทั้งหมด |
| life | +1 ชีวิต (max 5) |
| gun | +1 max bullet |

---

## Map Rules (สำคัญ!)

### กฎที่ต้องรักษาเสมอ
1. **ช่องทางกว้างอย่างน้อย 2 cells** (รถถัง = 2x2)
2. กลุ่มกำแพงในแถวเดียวกัน: ช่องว่างระหว่างกลุ่ม ≥ 2 cols
3. ระหว่างแถวกำแพง: ว่าง 2 แถวเสมอ
4. **cols 4-5 ต้องโล่งใน rows 21-25** (player spawn path)
5. **row 0 ต้องโล่งทั้งแถว** (enemy spawn)
6. Base อยู่ที่: row 23-24, cols 11-13 (`BBB` / `B*B`)

### Map Layout (แถว × 26 ตัวอักษร)
- แต่ละ level มี 26 แถว แถวละ 26 ตัวอักษร
- ตัวอักษร: `.`=empty, `B`=brick, `S`=steel, `W`=water, `G`=bush, `*`=base

### 3 Level ปัจจุบัน
- **Level 1**: กำแพงอิฐล้วน, ง่าย
- **Level 2**: ผสม BB + SS
- **Level 3**: SS หนาแน่น, ยาก
- (วนซ้ำเมื่อผ่าน Level 3)

---

## Audio

### เสียงระเบิด (explosion.mp3)
- ไฟล์ต้นฉบับ: `explosion-in-tight-space-gamemaster-audio-no-reverb-tail-1-00-00.mp3`
- Embed เป็น base64 ใน HTML โดยตรง
- เหตุผล: `fetch()` ถูก block เมื่อเปิดผ่าน `file://` protocol
- ใช้ Web Audio API: `audioCtx.decodeAudioData(bytes.buffer)`
- Fallback: synthesized noise burst ถ้า decode ไม่สำเร็จ

### เสียงอื่น ๆ (synthesized via Web Audio API)
- `shoot`: square wave 800→200Hz
- `metal`: square wave เมื่อกระสุนชนเหล็ก
- `powerup`: sine wave 440→660→880Hz

---

## Bugs ที่เคยแก้แล้ว

| Version | Bug | สาเหตุ | วิธีแก้ |
|---------|-----|--------|--------|
| v1.1 | Canvas ดำทั้งหมด หลัง Start | `map = []` ทำให้ `draw()` throw TypeError ตั้งแต่แรก → game loop หยุด | เปลี่ยนเป็น `Array.from({length:ROWS}, () => Array(COLS).fill(T.EMPTY))` |
| v1.2 | รถถังขยับไม่ได้ | Player spawn ที่ col 8-9 ทับกำแพงอิฐ → `canMove()` return false ทุกทิศ | ย้าย spawn มา `x: 4 * CELL` |
| v1.3 | ปืนหันผิดทิศ | `ctx.rotate()` คำนวณมุมซ้อนกัน 2 ครั้ง | เปลี่ยนเป็น `ctx.rotate(angles[dir])` |
| v1.4 | รถถังติดกำแพง | ช่องทางในแผนที่แคบกว่า 2 cells | ออกแบบแผนที่ใหม่ทั้งหมด |
| v1.5 | ตายแล้วข้าม level | `checkWin()` trigger ขณะ player = null → ข้ามไป level clear | เพิ่ม `levelWon` flag + แยก `showLevelClear()` / `showGameOver()` |
| v1.6 | เสียง MP3 ไม่ออก | `fetch()` ถูก block บน `file://` | Embed MP3 เป็น base64 ใน HTML |
| v1.7 | เกมแข็งค้างหลังตาย | Player bullets บินอยู่หลัง `player = null` → `player.bulletCount` throw TypeError → game loop หยุด | เพิ่ม `if (player)` ทุกจุด + try-catch ใน game loop |

---

## Version History

| Version | รายละเอียด |
|---------|-----------|
| v1.0 | สร้างเกมครั้งแรก |
| v1.1 | เพิ่มเสียงระเบิด synthesized (3 layers) |
| v1.2 | แก้ canvas ดำ + รถถังขยับไม่ได้ |
| v1.3 | แก้ปืนหันผิดทิศ |
| v1.4 | ออกแบบแผนที่ใหม่ ช่องทาง min 2 cells |
| v1.5 | ใช้เสียง MP3 จริง + แก้ level progression + 5 ชีวิต |
| v1.6 | Embed MP3 เป็น base64 แก้เสียงไม่ออกบน file:// |
| v1.7 | แก้เกมแข็งค้าง: null check + try-catch game loop |
| v1.8 | ปรับปรุงแผนที่ด่าน 1 และอัปเดตกราฟิกรถถัง/กำแพงให้เป็นสไตล์ 8-bit คลาสสิก |

---

## การตั้งค่า Git บนเครื่องใหม่

```bash
git config user.email "piriya2544@gmail.com"
git config user.name "p2544"
```

> ถ้า push แล้วถามรหัสผ่าน — ใช้ **Personal Access Token** จาก https://github.com/settings/tokens แทน password
