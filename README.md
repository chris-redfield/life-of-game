# Life of Game 🧬

> A biology simulation game featuring single-cell organisms with genetic encoding, evolution, and emergent behavior.
> 
> *Working title: "Primordial Soup" (potential future rebrand)*

![Status](https://img.shields.io/badge/status-MVP-green)
![Version](https://img.shields.io/badge/version-0.2.1-blue)

## Overview

**Life of Game** is a browser-based biological simulation that goes beyond traditional cellular automata (like Conway's Game of Life). Instead of simple on/off cells following fixed rules, our organisms have:

- **True DNA** with 500 genes across 5 chromosomes
- **Phenotype expression** - genes map to visible/behavioral traits
- **Metabolism** - organisms must eat to survive
- **Reproduction with mutation** - evolution happens naturally
- **Emergent species** - populations diverge into distinct species over time
- **Trade-offs** - no "perfect" organism; every advantage has a cost

## Table of Contents

- [Getting Started](#getting-started)
- [Core Concepts](#core-concepts)
- [DNA & Genetics System](#dna--genetics-system)
- [Trade-offs System](#trade-offs-system)
- [Organism Life Cycle](#organism-life-cycle)
- [Food System](#food-system)
- [Features List](#features-list)
- [Controls](#controls)
- [Roadmap](#roadmap)

---

## Getting Started

Simply open `life-of-game.html` in any modern browser. No dependencies or build process required.

```bash
# Clone or download, then:
open life-of-game.html
# or
python -m http.server 8000  # then visit localhost:8000
```

---

## Core Concepts

### Philosophy

Unlike Conway's Game of Life where cells follow deterministic rules based on neighbor counts, **Life of Game** creates organisms that:

1. **Have agency** - They seek food, move with purpose
2. **Have identity** - Each organism has unique DNA determining its traits
3. **Evolve** - Mutations accumulate over generations, creating new species
4. **Compete** - Resources are limited; better-adapted organisms thrive
5. **Face trade-offs** - Being fast costs more energy, being big makes you slower

### Inspiration vs. Implementation

| Cellular Automata | Life of Game |
|-------------------|--------------|
| Cells are on/off | Organisms have continuous properties |
| Rules are fixed | Behavior emerges from genetics |
| Patterns are discovered | Species evolve |
| No resources | Must acquire energy to survive |
| Instant state changes | Life cycle (birth → life → reproduction → death) |
| No trade-offs | Every advantage has a cost |

---

## DNA & Genetics System

### Structure

Each organism has **5 chromosomes**, each containing **100 genes** (total: 500 genes).

Gene values range from **0-255** (8-bit integers).

```
DNA
├── Chromosome 0: Physical Traits
│   └── Genes 0-99
├── Chromosome 1: Movement
│   └── Genes 0-99
├── Chromosome 2: Metabolism
│   └── Genes 0-99
├── Chromosome 3: Reproduction
│   └── Genes 0-99
└── Chromosome 4: Sensory/Behavior
    └── Genes 0-99
```

### Chromosome Functions

#### Chromosome 0: Physical Traits
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-19 | Size | **5-40 pixels** |
| 20-39 | Membrane Thickness | 0.5-4.0 |
| 40-59 | Elasticity | 0.2-1.2 |

#### Chromosome 1: Movement
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| Gene 0 (single) | Movement Type | none / cilia / flagellum / pseudopod |
| 25-49 | Genetic Speed | **0.1-4.0** |
| 50-74 | Turn Rate | 0.01-0.25 |
| 75-99 | Movement Efficiency | 0.3-2.0 |

*Note: Actual speed = Genetic Speed × Size Penalty (see Trade-offs)*

#### Chromosome 2: Metabolism
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| Gene 0 (single) | Can Eat Oxygen | Yes if > 100 |
| Gene 25 (single) | Can Eat Glucose | Yes if > 128 |
| Gene 50 (single) | Can Eat Amino Acids | Yes if > 180 |
| 75-99 | Metabolic Rate | **0.2-2.0** |

*Note: Specialist bonus applies based on how many food types organism can eat (see Trade-offs)*

#### Chromosome 3: Reproduction
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-32 | Reproduction Threshold | **40-200 energy** |
| 33-65 | Offspring Energy Share | 25-55% |
| 66-99 | Mutation Tendency | **1-20%** |

#### Chromosome 4: Sensory/Behavior
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-32 | Sensor Range | **15-250 pixels** |
| 33-65 | Food Preference | 0-1 (bias) |
| 66-99 | Flocking Tendency | 0-0.8 |

### Gene-to-Trait Mapping

Traits are decoded using **2-gene averaging** for balance between stability and mutation sensitivity:

```javascript
// Example: Getting organism size from Chromosome 0
// Uses 2 key genes (start and middle of range)
const gene1 = chromosome0[0];   // First gene of range
const gene2 = chromosome0[10];  // Middle gene of range
const geneAverage = (gene1 + gene2) / 2;
const size = mapRange(geneAverage, 0, 255, 5, 40);
```

**Why 2 genes?** 
- 1 gene = too jumpy (single mutation causes huge change)
- 3+ genes = too stable (mutations barely visible)
- 2 genes = balanced (mutation affects 50% of trait value)

For **categorical traits** (movement type, can eat X), single genes are used so mutations can flip categories:

```javascript
// Movement type from single gene
const typeIndex = Math.floor(chromosome1[0] / 64);  // 0, 1, 2, or 3
const movementType = ['none', 'cilia', 'flagellum', 'pseudopod'][typeIndex];
```

### Color as Species Indicator

Organism color is derived from genes across **3 chromosomes** for visual diversity:

```javascript
const r = chromosome0[10];  // Physical
const g = chromosome1[10];  // Movement
const b = chromosome2[10];  // Metabolism
```

This creates visible species clustering - similar organisms have similar colors.

### Mutation System

During reproduction, each of the 500 genes has a chance to mutate:

```javascript
// Mutation rate = global setting + organism's mutation tendency
// Example: 8% (slider) + 10% (genes) = 18% chance per gene
const mutationRate = globalMutationRate + organism.mutationTendency;

if (Math.random() < mutationRate) {
    // 90% chance: gradual mutation (±50)
    if (Math.random() > 0.1) {
        gene += randomInt(-50, 50);
    } 
    // 10% chance: jump mutation (completely new value)
    else {
        gene = randomInt(0, 255);
    }
}
```

**Mutation features:**
- **Large mutations** (±50) for visible changes
- **Jump mutations** (10% of mutations) can create completely new trait values
- **Additive rate** - organism's mutation tendency adds to global rate
- At 18% rate with 500 genes: **~90 genes mutate per reproduction**

---

## Trade-offs System

The simulation enforces trade-offs so no single strategy dominates:

### 1. Size → Speed Penalty

Large organisms are slower (like in real life):

| Size | Speed Multiplier |
|------|------------------|
| 5 (tiny) | 100% |
| 22.5 (medium) | 70% |
| 40 (huge) | 40% |

```
Actual Speed = Genetic Speed × Size Penalty

Example: Genetic speed 3.0, Size 30
Penalty = 0.54 (54%)
Actual speed = 3.0 × 0.54 = 1.62
```

### 2. Movement Type Modifiers

Different movement types have speed and maneuverability trade-offs:

| Type | Speed Modifier | Turn Rate Modifier | Notes |
|------|----------------|-------------------|-------|
| **Flagellum** | ×1.1 | ×0.9 | Fast but less maneuverable |
| **Cilia** | ×1.0 | ×1.0 | Balanced |
| **Pseudopod** | ×0.7 | ×1.0 | Slow but efficient |
| **None** | N/A | N/A | Passive drift only |

### 3. Speed → Energy Cost (×1.5)

Fast organisms burn energy 1.5× faster:

```
Movement energy cost = baseSpeed × 0.012 × 1.5
```

A speed-4.0 organism burns energy ~3× faster than a speed-1.0 organism.

### 4. Sensor Range → Energy Drain

Good sensors cost a small amount of energy:

```
Sensor drain = sensorRange × 0.00004
```

At max range (250): ~0.01 energy/tick (small but adds up)

### 5. Specialist Bonus

Organisms that can eat fewer food types are MORE efficient at eating:

| Can Eat | Bonus |
|---------|-------|
| 1 type (specialist) | **1.8×** energy gain |
| 2 types | **1.3×** energy gain |
| 3 types (generalist) | **1.0×** energy gain |

This creates viable strategies:
- **Oxygen specialist**: Can only eat O₂, but gets 1.8× energy from it
- **Generalist**: Can eat everything, but gets base energy

### Viable Strategies

These trade-offs enable multiple winning strategies:

| Strategy | Size | Speed | Diet | Pros | Cons |
|----------|------|-------|------|------|------|
| **Tank** | Large | Slow | O₂ specialist | High energy efficiency, survives on ambient oxygen | Can't chase rare food |
| **Hunter** | Small | Fast | Glucose/Amino specialist | Catches rare high-energy food | Burns energy quickly |
| **Balanced** | Medium | Medium | 2 types | Good at both | Master of none |
| **Opportunist** | Small | Medium | Generalist | Can eat anything | Lower efficiency |

---

## Organism Life Cycle

```
    ┌─────────────────────────────────────────┐
    │                                         │
    ▼                                         │
  BIRTH ──► LIVING ──► REPRODUCTION ──────────┘
    │           │              │
    │           │              └──► CHILD (with mutations)
    │           │
    │           ▼
    │     ENERGY DEPLETED
    │           │
    │           ▼
    └───────► DEATH
```

### Birth
- Organism spawns with initial energy (50-80)
- DNA is either random (first generation) or inherited (with mutations)
- Traits are decoded from DNA immediately

### Living
- **Energy drain**: Constant drain based on size, speed, and sensors
- **Movement cost**: Moving organisms spend extra energy (speed × 1.5)
- **Aging**: Age counter increases

### Eating
- Organism detects food within **sensor range**
- Moves toward nearest **compatible** food (based on metabolism genes)
- On contact, food is consumed:
  - Energy gained = food.energy × metabolicRate × specialistBonus
  - **Primary food bonus**: +40% if food matches primary type

### Reproduction
- Triggered when energy ≥ reproduction threshold
- **Asexual division**: One parent, one child
- Energy is split between parent and child
- Child DNA = Parent DNA + mutations (~90 genes change)
- Child spawns adjacent to parent

### Death
- When energy ≤ 0
- Organism is removed from simulation
- Archived for statistics (top 100 dead organisms tracked)

---

## Food System

Three types of food particles exist in the simulation:

### Oxygen (O₂)
| Property | Value |
|----------|-------|
| Color | Blue |
| Size | 4 pixels |
| Energy | 5 |
| Abundance | High |
| Lifespan | 600 ticks |
| Spawn rate | 3 per cycle |

**Strategy**: Slow, large specialists can survive purely on oxygen.

### Glucose (G)
| Property | Value |
|----------|-------|
| Color | Yellow |
| Size | 6 pixels |
| Energy | 15 |
| Abundance | Medium |
| Lifespan | 400 ticks |
| Spawn rate | 2 per cycle |

**Strategy**: Balanced option. Worth chasing for medium-speed organisms.

### Amino Acids (A)
| Property | Value |
|----------|-------|
| Color | Pink |
| Size | 8 pixels |
| Energy | 30 |
| Abundance | Low |
| Lifespan | 300 ticks |
| Spawn rate | 1 per cycle |

**Strategy**: High risk, high reward. Only fast organisms with good sensors benefit.

### Food Dynamics

- Food particles **drift** slowly
- Food **decays** over time (fades and disappears)
- Food **spawns** continuously based on spawn rate setting
- Maximum caps prevent overpopulation of food

---

## Features List

### Currently Implemented

#### Organisms
- [x] DNA with 5 chromosomes × 100 genes
- [x] 2-gene trait decoding (balanced mutation sensitivity)
- [x] Single-gene categorical traits (movement type, diet)
- [x] Four movement types (none, cilia, flagellum, pseudopod)
- [x] Visual movement animations
- [x] Energy system with trade-off costs
- [x] Food-seeking behavior with sensor range
- [x] Asexual reproduction with significant mutation
- [x] Jump mutations for occasional big changes
- [x] Visual species differentiation (color from multiple chromosomes)
- [x] Individual organism selection and inspection

#### Trade-offs
- [x] Size → Speed penalty (big = slow)
- [x] Speed → Energy cost (×1.5)
- [x] Sensor range → Energy drain
- [x] Specialist bonus (fewer food types = more efficient)

#### Environment
- [x] Three food types (oxygen, glucose, amino acids)
- [x] Food spawning and decay
- [x] Boundary wrapping (toroidal world)
- [x] Lake/pond background aesthetic

#### UI/UX
- [x] Real-time population statistics
- [x] Generation tracking
- [x] Birth/death counters
- [x] Food supply indicators
- [x] Species distribution visualization
- [x] Selected organism DNA/trait display
- [x] Genetic vs Actual speed display
- [x] Specialist bonus display
- [x] Simulation speed controls (1x, 2x, 4x)
- [x] Pause functionality
- [x] Manual food burst
- [x] Manual organism spawning
- [x] Simulation reset

### Visual Polish
- [x] Bioluminescent glow effects
- [x] Pulsing cell animation
- [x] Movement appendage rendering
- [x] Energy bar per organism
- [x] Ambient particle overlay
- [x] Lake/pond background gradient

### Statistics & Analytics
- [x] Population timeline chart
- [x] Birth/death rate chart over time
- [x] Movement type distribution
- [x] Metabolism distribution
- [x] Top 5 longest-lived organisms
- [x] Top 5 most offspring producers
- [x] Average traits evolution chart
- [x] Generation distribution histogram
- [x] Size vs Speed scatter plot
- [x] Real-time trait averages comparison
- [x] Genetic speed vs Actual speed tracking

---

## Controls

### Mouse
- **Click** on organism: Select and view its DNA/traits
- **Click** on empty space: Deselect

### UI Controls

| Control | Function |
|---------|----------|
| Food Spawn Rate slider | Adjust how fast food appears (0.1x - 3x) |
| Mutation Rate slider | Global mutation modifier (1% - 30%) |
| ⏸ button | Pause/resume simulation |
| 1x / 2x / 4x buttons | Simulation speed |
| 📊 View Statistics | Open stats overlay with charts and analytics |
| + Add 10 Organisms | Spawn 10 random organisms |
| + Add Food Burst | Spawn extra food of all types |
| Reset Simulation | Clear everything and restart |

### Selected Organism Display

When you click an organism, you see:
- **Movement**: none / cilia / flagellum / pseudopod
- **Speed**: `1.24 (gene: 2.50)` - actual speed and genetic speed
- **Size**: Physical size in pixels
- **Primary Food**: Preferred food type
- **Can Eat**: `O₂ G (1.3x)` - what it can eat + specialist bonus
- **Energy**: Current energy level
- **Generation**: How many ancestors
- **Sensor Range**: Detection radius

---

## Roadmap

### Phase 2: Expanded Biology
- [ ] **Predation** - Carnivore organisms that eat other organisms
- [ ] **Cell walls/defense** - Structural genes that prevent being eaten
- [ ] **Toxin production** - Chemical warfare between species
- [ ] **Photosynthesis** - Autotrophs that generate energy from "light"

### Phase 3: Sexual Reproduction
- [ ] **Mating types** - A/B types that must combine
- [ ] **DNA crossover** - Child gets chromosomes from both parents
- [ ] **Sexual selection** - Preference for certain traits in mates

### Phase 4: Environment
- [ ] **Temperature zones** - Hot/cold areas affecting metabolism
- [ ] **Light gradient** - For photosynthesis mechanics
- [ ] **Obstacles** - Rocks, barriers organisms must navigate
- [ ] **Currents** - Water flow affecting movement

### Phase 5: Colony Behavior
- [ ] **Cell adhesion genes** - Organisms stick together
- [ ] **Differentiation** - Colonial organisms with different roles
- [ ] **Multicellularity** - First steps toward complex life

### Phase 6: Analytics & History
- [ ] **Phylogenetic tree** - Visual family tree of all species
- [ ] **Trait graphs over time** - How traits evolve
- [ ] **Extinction events** - Track species that died out
- [ ] **Save/load** - Persist simulations

---

## Technical Notes

### Performance
- Canvas-based rendering
- Efficient spatial queries for food detection
- Smooth 60fps target with speed multiplier

### Browser Support
- Modern browsers (Chrome, Firefox, Safari, Edge)
- No external dependencies
- Single HTML file (~2500 lines)

### Code Structure
```
life-of-game.html
├── CSS (embedded)
│   ├── Lake/pond theme
│   ├── Side panel styling
│   └── Stats overlay styling
├── JavaScript (embedded)
│   ├── DNA class (genetics, mutation, trait decoding)
│   ├── Organism class (behavior, movement, reproduction)
│   ├── Food class (spawning, decay)
│   ├── Trade-offs system
│   ├── Simulation loop
│   ├── Statistics tracking & charts
│   └── UI handlers
└── HTML
    ├── Canvas (simulation area)
    ├── Stats overlay
    └── Side panel (controls/stats)
```

---

## Changelog

### v0.2.1 - Movement Balance
- **Flagellum nerf**: Speed modifier reduced from ×1.3 to ×1.1, turn rate reduced to ×0.9
- **Movement type modifiers**: Documented speed and turn rate modifiers for all movement types

### v0.2.0 - Evolution Update
- **Trade-offs system**: Size→speed penalty, speed energy cost ×1.5, sensor drain, specialist bonus
- **Wider trait ranges**: Size 5-40, Speed 0.1-4.0, Sensors 15-250
- **Fixed mutations**: 2-gene averaging, single genes for categories, ±50 mutations, 10% jump mutations
- **Fixed mutation rate**: Now additive (global + tendency) instead of multiplicative
- **UI improvements**: Shows genetic vs actual speed, specialist bonus multiplier
- **Statistics overlay**: 10 different charts and analytics
- **Visual**: Lake/pond background instead of black

### v0.1.0 - Initial Release
- Basic DNA system with 5 chromosomes
- Four movement types
- Three food types
- Asexual reproduction
- Basic UI and controls

---

## Contributing

Ideas and contributions welcome! Key areas:

1. **New traits** - What other biological features could genes encode?
2. **Behaviors** - More complex organism decision-making
3. **Visualization** - Better ways to show evolution happening
4. **Performance** - Handle 1000+ organisms smoothly
5. **Balance** - Tune trade-offs for interesting dynamics

---

## License

MIT License - Feel free to use, modify, and distribute.

---

## Acknowledgments

- Inspired by Conway's Game of Life and its 55+ years of engineering history
- Wolfram's work on cellular automata and computational irreducibility
- The real primordial soup that started it all ~4 billion years ago

---

*"In the beginning, there was chemistry. Then chemistry became biology. Then biology became... competitive."*
