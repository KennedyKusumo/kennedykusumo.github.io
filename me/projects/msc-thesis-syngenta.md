# MSc Thesis: Harvest Schedule Optimisation for Syngenta

## Summary
Developed a two-stage MILP optimisation tool in AIMMS to help Plant Managers schedule
hybrid corn seed harvesting operations. The tool cut typical planning time from ~6 days
away from optimal to ~2 days, saving hundreds of skilled man-hours per season.

---

## Business Context
- **Client:** Syngenta (agri-seed company)
- **User:** Plant Managers scheduling annual harvest campaigns
- **Scale:** 5–10 growing areas × 80–100 fields each = 400–1,000 fields; 3–5 processing plants
- **Crop:** Hybrid corn seed (perishable — hours to days after harvest)
- **Planning problem:** Every field grows a specific hybrid; harvest window is narrow
  (defined by moisture content dropping below a threshold before freeze); pickers
  must be contracted and allocated in advance

## Planning Stages
| Stage | Lead time | Data quality |
|-------|-----------|-------------|
| S1 (initial plan) | ~6 months | Rough yield and harvest-start estimates (area × hybrid level) |
| S2 (daily revision) | 2–4 weeks | More accurate estimates (field × hybrid level); revises S1 plan |

---

## Model Structure

### Model type
Two-stage MILP (mixed-integer linear programme), implemented in AIMMS 4.34.

### Sets & Indices
| Set | Index | Description |
|-----|-------|-------------|
| Field | f | Individual fields (80–100 per area) |
| GrowingArea | g | Geographic clusters of fields |
| Hybrid | h | Corn hybrid varieties |
| Day | d | Calendar days (planning horizon) |
| Picker | p | Contracted harvesting crews |
| ProcessingPlant | pp | Drying/processing facilities |

### Key Parameters
| Parameter | Units | Description |
|-----------|-------|-------------|
| LandUsage(g,f,h) | Acres | Land area planted with hybrid h in field f |
| Yield(g,f,h) | Bu/acre | Estimated yield at each planning stage |
| EstimatedHarvestStart(g,h) / (g,f,h) | Date | S1 by area×hybrid; S2 by field×hybrid |
| EstimatedFreezeStart(g) | Date | Freeze date by growing area |
| LogisticsCost(g,pp) | $/Bu | Transport cost from area to plant |
| PickerMaxCapacity(p) | Bu/day | Max daily harvest capacity per picker |
| PlantMaximumCapacity(pp) | Bu/day | Max daily processing capacity per plant |
| TransportMaximumLoad(g,pp) | Bu/trip | Max load per truck trip |
| TransportNumberOfTripsPerDay(g,pp) | — | Trips per day |
| PickedCost | $/Bu | $75 per bushel (contracted harvest cost) |
| FreezeRiskCost | $ | $1,000,000 penalty per day harvesting after freeze |
| HarvestDelayCost | $/Bu/day | Penalty for each bushel-day of delay past optimum |
| UnharvestedCost(h) | $/Bu | Penalty for crop left unharvested at end of window |

### Decision Variables (S1 — initial plan)
| Variable | Domain | Units | Description |
|----------|--------|-------|-------------|
| S1HarvestPlan(g,f,h,p,d) | within harvest window | Bu | Allocation of picker p to field (g,f,h) on day d |
| S1TransportPlan(g,pp,d,h) | planning horizon | Bu | Crop transported from area g to plant pp on day d |
| S1ProcessingPlan(pp,d,h) | planning horizon | Bu | Crop processed at plant pp on day d |
| S1HarvestBinary(g,f,h,p,d) | binary | — | 1 if picker p is assigned to field on day d |
| S1HarvestDayBinary(d) | binary | — | 1 if any harvesting occurs on day d |
| S1UnharvestedCrop(g,f,h,d) | | Bu | Cumulative unharvested crop at end of day d |

### S2 Extensions (daily revision)
S2 takes the S1 plan as a starting point and optimises deviations:
- S2HarvestPlanAdded / S2HarvestPlanRemoved — changes to harvest allocation
- Same pattern for transport and processing
- Uses field-level harvest-start estimates (more accurate than S1's area-level)
- Carries forward actual harvested amounts from past days

### Objective Function (minimise)
```
OverallCost = 
  PickingCost           (PickedCost × Σ HarvestPlan / Yield)
+ LogisticsCost         (LogisticsCost × Σ TransportPlan × Bu2KgConvFactor)
+ ProcessingCost        (ProcessingCost × Σ ProcessingPlan)
+ FreezeRiskCost        ($1M × extra days after freeze)
+ DelayInHarvestCost    (HarvestDelayCost × HybridWeight × Σ UnharvestedCrop per day)
+ UnharvestedCost       (UnharvestedCost(h) × Σ UnharvestedCrop at end of window)
+ NumberOfHarvestDaysCost (CostPerDay × Σ HarvestDayBinary)
```

Alternative objective (S2 only): **minimise number of harvest days** (fastest harvest mode).

### Key Constraints
| Constraint | Description |
|------------|-------------|
| S1CropBalance(g,f,h) | Harvested + Unharvested = LandUsage × Yield |
| S1PickerCapacityConstraint(p,d) | Σ HarvestPlan ≤ PickerMaxCapacity(p) |
| S1PickerFairnessOverall(p,p') | Overall capacity difference between any two pickers ≤ tolerance |
| S1PickerFairnessDaily | Daily capacity equalisation (non-essential, toggleable) |
| S1HarvestBeforeFreeze(g,f,h,p,d) | No harvesting after EstimatedFreezeStart + buffer |
| S1HarvestAfterForecast(g,f,h,p,d) | No harvesting before EstimatedHarvestStart (moisture window) |
| S1TransportBalance(g,h,d) | All harvested on day d must be transported same day |
| S1ProcessingBalance(pp,d,h) | All transported must be processed same day |
| S1PlantMaxCapacity(pp,d) | Σ ProcessingPlan ≤ PlantMaximumCapacity(pp) |
| S1OverallMaximumTransportLoad(g,pp,d) | Transport ≤ MaxLoad × Trucks × Trips |
| S1MinimumTransportLoad | Minimum load required if transport is used (binary) |
| S1MinimumPickingCapacity | Minimum picking if picker is assigned to a field |

Bottleneck detection: model automatically identifies whether harvest, transport, or
processing is the binding constraint.

---

## Case Studies
- 5 case studies run (Case Studies 1–5 in Excel files)
- Each has separate Harvest SO, Processing SO, Transport SO output sheets
- Variables between case studies: [to confirm — likely scale, number of fields/pickers, objective]

---

## Key Result
- **Average timeliness:** improved from ~6 days away from optimal (manual) to ~2 days with the tool
- **Worst case:** the real manual plan was up to 17 days away from the optimal date; the tool brings this down to a maximum of 6 days — a ~3× reduction in worst-case deviation
- Hundreds of skilled man-hours saved per season
- Model solved in reasonable time for realistic problem size (AIMMS with CPLEX)

---

## Technical Notes
- Bushels-to-kg conversion: Bu2KgConvF = MoistureContent × 0.6336 + 21.275
- BigM = 1e12 (for binary-continuous linking constraints)
- MoistureDelayTolerance: configurable window of days past optimum harvest date
- Constraints split into "essential" (always active) and "non-essential" (user-toggleable via GUI)
- Data imported from Excel workbooks via AIMMS SpreadSheet library

---

## Source Files
| File | Content |
|------|---------|
| `Submission/Thesis.pdf` | Full submitted dissertation |
| `Thesis.docx`, `Thesis Updated.docx` | Word drafts |
| `Initial Problem Statement.docx` | Full problem definition with sets/parameters/constraints |
| `Model Description 26 Jun.docx` | Formal nomenclature document |
| `More Constraints.docx` | Additional constraints (hybrid priority, contamination, plant types) |
| `Harvest optimization_Problem Specification.pptx` | Problem spec slides |
| `Poster Presentation.pptx` | Summary poster |
| `Dissertation/12 Jul - DelayReformulation/` | Final AIMMS model (most complete) |
| `Case Studies/` | 5 Excel case study output files |
| `Syngenta Meeting/` | Meeting notes from client meetings |
| `Literature Review/Final/` | Literature review document |
| `16 Jun 2017.docx`, `20 Jun 2017.docx`, etc. | Supervisor meeting notes |
