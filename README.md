# Life of Game 🧬

> A biology simulation game featuring single-cell organisms with genetic encoding, evolution, and emergent behavior.
> 
> *Working title: "Primordial Soup" (potential future rebrand)*

![Status](https://img.shields.io/badge/status-MVP-green)
![Version](https://img.shields.io/badge/version-0.1.0-blue)

## Overview

**Life of Game** is a browser-based biological simulation that goes beyond traditional cellular automata (like Conway's Game of Life). Instead of simple on/off cells following fixed rules, our organisms have:

- **True DNA** with 500 genes across 5 chromosomes
- **Phenotype expression** - genes map to visible/behavioral traits
- **Metabolism** - organisms must eat to survive
- **Reproduction with mutation** - evolution happens naturally
- **Emergent species** - populations diverge into distinct species over time

## Table of Contents

- [Getting Started](#getting-started)
- [Core Concepts](#core-concepts)
- [DNA & Genetics System](#dna--genetics-system)
- [Organism Life Cycle](#organism-life-cycle)
- [Food System](#food-system)
- [Features List](#features-list)
- [Controls](#controls)
- [Roadmap](#roadmap)

---

## Getting Started

Simply open `index.html` in any modern browser. No dependencies or build process required.

```bash
# Clone or download, then:
open index.html
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

### Inspiration vs. Implementation

| Cellular Automata | Life of Game |
|-------------------|--------------|
| Cells are on/off | Organisms have continuous properties |
| Rules are fixed | Behavior emerges from genetics |
| Patterns are discovered | Species evolve |
| No resources | Must acquire energy to survive |
| Instant state changes | Life cycle (birth → life → reproduction → death) |

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
| 0-19 | Size | 8-25 pixels |
| 20-39 | Membrane Thickness | 0.5-3.0 |
| 40-59 | Elasticity | 0.3-1.0 |

#### Chromosome 1: Movement
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-24 | Movement Type | none / cilia / flagellum / pseudopod |
| 25-49 | Base Speed | 0.2-2.0 |
| 50-74 | Turn Rate | 0.02-0.15 |
| 75-99 | Movement Efficiency | 0.5-1.5 |

#### Chromosome 2: Metabolism
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-24 | Can Eat Oxygen | Yes if avg > 100 |
| 25-49 | Can Eat Glucose | Yes if avg > 128 |
| 50-74 | Can Eat Amino Acids | Yes if avg > 180 |
| 75-99 | Metabolic Rate | 0.3-1.2 |

#### Chromosome 3: Reproduction
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-32 | Reproduction Threshold | 60-150 energy |
| 33-65 | Offspring Energy Share | 30-50% |
| 66-99 | Mutation Tendency | 1-15% |

#### Chromosome 4: Sensory/Behavior
| Gene Range | Trait | Output Range |
|------------|-------|--------------|
| 0-32 | Sensor Range | 30-120 pixels |
| 33-65 | Food Preference | 0-1 (bias) |
| 66-99 | Flocking Tendency | 0-0.5 |

### Gene-to-Trait Mapping

Genes are decoded using **range averaging**:

```javascript
// Example: Getting organism size from Chromosome 0, genes 0-19
const geneAverage = chromosome0.slice(0, 20).reduce((a,b) => a+b) / 20;
const size = mapRange(geneAverage, 0, 255, 8, 25);
```

For categorical traits (like movement type):

```javascript
// Movement type from Chromosome 1, genes 0-24
const avg = getGeneRange(1, 0, 25);  // 0-255
const typeIndex = Math.floor(avg / 64);  // 0, 1, 2, or 3
const movementType = ['none', 'cilia', 'flagellum', 'pseudopod'][typeIndex];
```

### Color as Species Indicator

Organism color is derived from **Chromosome 2 (Metabolism)**:

```javascript
const r = averageOf(chromosome2, 0, 33);
const g = averageOf(chromosome2, 33, 66);
const b = averageOf(chromosome2, 66, 100);
```

This means organisms with similar metabolisms appear as similar colors, creating **visual species clustering**.

### Mutation

During reproduction, each gene has a chance to mutate:

```javascript
if (Math.random() < mutationRate) {
    const change = randomInt(-15, 15);
    gene = clamp(gene + change, 0, 255);
}
```

- Mutations are **small** (±15 max)
- Mutation rate is influenced by both **global setting** and **organism's own mutation tendency gene**
- Over generations, mutations accumulate → species diverge

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
- **Energy drain**: Constant drain based on size and metabolic rate
- **Movement cost**: Moving organisms spend extra energy
- **Aging**: Age counter increases (currently cosmetic)

### Eating
- Organism detects food within **sensor range**
- Moves toward nearest **compatible** food (based on metabolism genes)
- On contact, food is consumed:
  - Energy gained = food.energy × metabolicRate
  - **Bonus**: 1.5x energy if food matches primary food type

### Reproduction
- Triggered when energy ≥ reproduction threshold
- **Asexual division**: One parent, one child
- Energy is split between parent and child
- Child DNA = Parent DNA + mutations
- Child spawns adjacent to parent

### Death
- When energy ≤ 0
- Organism is removed from simulation
- Death counter increments

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

**Strategy**: Organisms that can eat oxygen don't need to move much - it's everywhere.

### Glucose (G)
| Property | Value |
|----------|-------|
| Color | Yellow |
| Size | 6 pixels |
| Energy | 15 |
| Abundance | Medium |
| Lifespan | 400 ticks |
| Spawn rate | 2 per cycle |

**Strategy**: Good balance of availability and energy. Mobile organisms thrive on glucose.

### Amino Acids (A)
| Property | Value |
|----------|-------|
| Color | Pink |
| Size | 8 pixels |
| Energy | 30 |
| Abundance | Low |
| Lifespan | 300 ticks |
| Spawn rate | 1 per cycle |

**Strategy**: High risk, high reward. Only fast organisms with good sensors can reliably find amino acids.

### Food Dynamics

- Food particles **drift** slowly
- Food **decays** over time (fades and disappears)
- Food **spawns** continuously based on spawn rate setting
- Maximum caps prevent overpopulation of food

---

## Features List

### Currently Implemented (MVP)

#### Organisms
- [x] DNA with 5 chromosomes × 100 genes
- [x] Trait decoding from genes
- [x] Four movement types (none, cilia, flagellum, pseudopod)
- [x] Visual movement animations (flagellum wave, cilia wiggle, pseudopod ooze)
- [x] Energy system with drain and consumption
- [x] Food-seeking behavior with sensor range
- [x] Asexual reproduction with mutation
- [x] Visual species differentiation (color from metabolism)
- [x] Individual organism selection and inspection

#### Environment
- [x] Three food types (oxygen, glucose, amino acids)
- [x] Food spawning and decay
- [x] Boundary wrapping (toroidal world)

#### UI/UX
- [x] Real-time population statistics
- [x] Generation tracking
- [x] Birth/death counters
- [x] Food supply indicators
- [x] Species distribution visualization
- [x] Selected organism DNA/trait display
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
- [x] Dark, primordial aesthetic
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

---

## Controls

### Mouse
- **Click** on organism: Select and view its DNA/traits
- **Click** on empty space: Deselect

### UI Controls

| Control | Function |
|---------|----------|
| Food Spawn Rate slider | Adjust how fast food appears (0.1x - 3x) |
| Mutation Rate slider | Global mutation modifier (1% - 20%) |
| ⏸ button | Pause/resume simulation |
| 1x / 2x / 4x buttons | Simulation speed |
| 📊 View Statistics | Open stats overlay with charts and analytics |
| + Add 10 Organisms | Spawn 10 random organisms |
| + Add Food Burst | Spawn extra food of all types |
| Reset Simulation | Clear everything and restart |

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
- Single HTML file (~800 lines)

### Code Structure
```
index.html
├── CSS (embedded)
│   └── Dark bioluminescent theme
├── JavaScript (embedded)
│   ├── DNA class
│   ├── Organism class
│   ├── Food class
│   ├── Simulation loop
│   └── UI handlers
└── HTML
    ├── Canvas (simulation area)
    └── Side panel (controls/stats)
```

---

## Contributing

Ideas and contributions welcome! Key areas:

1. **New traits** - What other biological features could genes encode?
2. **Behaviors** - More complex organism decision-making
3. **Visualization** - Better ways to show evolution happening
4. **Performance** - Handle 1000+ organisms smoothly

---

## License

MIT License - Feel free to use, modify, and distribute.

---

## Acknowledgments

- Inspired by Conway's Game of Life and its 55+ years of engineering history
- Wolfram's work on cellular automata and computational irreducibility
- The real primordial soup that started it all ~4 billion years ago

---

*"In the beginning, there was chemistry. Then chemistry became biology. Then biology became... interesting."*
