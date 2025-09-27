# WaterDetectionGame — An Interactive RL Mapping Agent

**WaterDetectionGame** is a playful, visual sandbox for teaching and tuning a learning agent that explores a world, **maps the environment**, and **survives periodic floods** by learning to climb trees. It blends three ideas:

1. **Reinforcement Learning (Q‑Learning)** — a tabular learner that chooses between *mapping* and *climbing* based on state and reward.
2. **Shaped rewards & curriculum** — discovery bonuses (new land, first tree, first‑time water sensing) and survival signals (safe vs. drowned during a flood).
3. **Continual environment mapping** — the agent builds a map each round and **remembers water locations across generations**, encouraging long‑term strategy.

Open the HTML file in a browser and watch the agent figure out how to explore efficiently **and** be on a tree when the water rises 🌊🌳.

---

## How it learns

### 1) Q‑Learning (Reinforcement Learning)
The agent uses **tabular Q‑learning** with an ε‑greedy policy. A compact state is derived from:
- Whether the agent is **on a tree** (safe perch).
- Whether there’s **nearby unseen** terrain (exploration opportunity).
- **Urgency** bucket (time remaining until the next flood).

**Actions:**  
- **MAP** — move toward unseen cells (or greedily explore).  
- **CLIMB** — if on a tree, gain altitude (safety for the upcoming flood).

**Update rule:**  
The agent collects a step reward, bootstraps from the next state’s best Q‑value, and updates the chosen action’s Q‑value with learning rate **α** and discount **γ**.

### 2) Reward Shaping
To make learning fun and fast, we shape the reward signal:
- `R_TIME`: small negative step cost (encourages efficiency).
- `R_NEW_LAND`: bonus for the **first visit** to a cell (progress).
- `R_VISIT`: small bonus for revisits (momentum without loops).
- `R_DISCOVER_TREE`: **boosted** bonus for first time stepping onto a tree (teaches “trees are valuable”).
- `R_SENSE_WATER`: bonus when the agent **first senses** a specific water tile (adjacent). This builds a water map.
- `R_FLOOD_SAFE`: reward when a flood hits **and** the agent is on a tree.
- `R_FLOOD_DEATH`: penalty if the agent gets flooded while not on a tree.
- `R_STUCK`: penalty for bumping / not making progress.

Together, these rewards create a curriculum: explore → find trees → be on a tree at flood time.

### 3) Mapping & Memory
- The agent maintains **`seen`** (cells visited this run) and **`waterSeen`** (tiles identified as water).  
- **Water memory is persistent across agent resets and generations**. When a flood wipes progress, the agent **keeps** its water knowledge and tries a new mapping plan with better priors. (Only a **Regenerate Map** clears water memory.)

This turns the problem into a mini‑**continual learning** loop: knowledge compounds across episodes while the policy improves.

---

## World & Dynamics

- Grid world **80×80** with land, water, and trees.  
- **Flood** occurs periodically. The agent must be **on a tree** to survive a flood tick (gets `R_FLOOD_SAFE`).  
- Flood spreads organically with row‑throttling to avoid instant full rows.  
- Trees are safe perches; water is a barrier; land is explorable.  
- The **AI Minimap** shows:
  - Green: visited land (this run)
  - Brown: tree (only after visited)
  - Blue: water (only after sensed)
  - Gray: unknown

---

## Sliders & Dials (live tuning)

### Flood & Loop
- **Flood period (s):** Time between flood ticks. Shorter means the agent must prioritize safety sooner.  
- **Learning speed (steps/frame):** More steps per animation frame = faster learning/acting (uses more CPU).

### Reinforcement Learning
- **ε (epsilon):** Exploration rate. Higher = more random choices. Try 0.25–0.40 early, then reduce.  
- **α (alpha):** Learning rate. Higher updates faster but can destabilize. 0.1–0.4 is common.  
- **γ (gamma):** Discount factor. Higher values value long‑term rewards (e.g., preparing for floods).

### Rewards (all editable on the fly)
- `R_TIME` (default **-0.01**): slight time cost.  
- `R_NEW_LAND` (default **1.0**): bonus for discovering new land tiles.  
- `R_VISIT` (default **0.1**): small revisit bonus.  
- `R_DISCOVER_TREE` (default **1.8**): **boosted** to make trees enticing.  
- `R_SENSE_WATER` (default **0.3**): reward for first‑time water detections (adjacent).  
- `R_FLOOD_SAFE` (default **6.0**): reward for being on a tree when flood hits.  
- `R_FLOOD_DEATH` (default **-8.0**): penalty for being caught on the ground.  
- `R_STUCK` (default **-2.0**): bumping / no progress.

> Tip: To strongly enforce “tree before flood,” **decrease Flood period** (e.g., 15–20s) and **increase `R_DISCOVER_TREE`** and/or **`R_FLOOD_SAFE`** magnitude.

---

## Controls

- **Reset Agent:** Start a new run **without** forgetting water memory. (Good for retrying with new hyperparameters.)  
- **Regenerate Map:** Entirely new world & snapshots — **clears water memory**.  
- **Pause/Resume:** Toggle the simulation loop.

---

## How to run

1. Save the provided HTML as `SurvivalGame.html` (or keep your filename).  
2. Double‑click to open in a modern browser (Chrome, Edge, or Firefox).  
3. Use the right‑sidebar dials to tune learning and rewards in real time.  
4. Watch the minimap fill in, and keep an eye on **“Next flood in”** to see whether the agent learned to climb.

No build tools. No server. Just open the file and play.

---

## Tuning Playbook

- **Too timid / not exploring?** Increase **ε** or **R_NEW_LAND**; reduce **R_STUCK**.  
- **Explores but drowns a lot?** Increase **R_DISCOVER_TREE** and **R_FLOOD_SAFE**, shorten the **Flood period**, and/or increase **γ**.  
- **Over‑explores and forgets safety?** Decrease **ε** and raise **`R_FLOOD_DEATH`** magnitude (more negative).  
- **Learns slowly?** Raise **α** a bit and increase **steps/frame**.  
- **Overfits to a lucky path?** Hit **Reset Agent** to keep water memory but reroll the run.

---

## Why this project is cool 😎

- It’s a **live, visual demo** of RL that you can **feel** by dragging sliders.  
- It shows how **reward shaping** and **memory** can turn a hard survival task into something learnable.  
- It’s compact: a single file you can tweak, hack, and extend.  
- It invites experiments: curriculum floods, new sensors, multi‑action policies, or switching to function approximation.

---

## Ideas & Extensions

- Replace the tabular Q‑table with **function approximation** (tile encodings → neural Q).  
- Add **intrinsic curiosity**: bonus for visiting novel states beyond the simple `R_NEW_LAND`.  
- Give the agent a **“plan” action** (wait strategically on trees near flood time).  
- Multi‑objective rewards: coverage speed vs. survival rate trade‑offs.  
- Procedural terrains (rivers, islands), variable tree heights, or temporary bridges.

---

## Troubleshooting

- **Minimap doesn’t show water I know exists:** The agent only marks water after **adjacent sensing**; it won’t pre‑reveal distant water.  
- **Agent forgets water after a reset:** Use **Reset Agent**, not **Regenerate Map**. Regenerate builds a brand‑new world and clears water memory by design.  
- **Performance:** Reduce **steps/frame** or the browser zoom. On slower laptops, 12–24 steps/frame is a sweet spot.

---

Have fun turning knobs and watching strategy emerge. If you make a neat variant, document your reward schedule and share a gif—this little agent loves new worlds! 🌍🧠
