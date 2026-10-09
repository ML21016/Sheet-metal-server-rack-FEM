# Sheet Metal Server Rack Shelf — FEM Analysis

FEA simulation of a sheet metal server rack shelf using Siemens NX/Nastran, 
including diagnosing and resolving mesh connectivity and solver singularity 
issues.

## Project Setup

This project involved building and analyzing a finite element model (FEM) of a 
sheet metal server rack shelf in Siemens NX, as part of a manufacturing/DfX 
coursework assignment.

**Model configuration:**
- **Constraints:** 8 fixed constraints representing screw mounting holes 
  (4 on each side of the shelf, matching the cabinet attachment points).
- **Load:** A force representing the server's weight plus a safety factor, 
  applied to simulate the shelf under service conditions.

**Initial issue:** After running the solver, the Results node in the 
Simulation Navigator displayed a status of **"Inferred"** rather than actual 
output — indicating the solve had not produced usable results. This kicked 
off a troubleshooting process to identify and resolve the underlying cause, 
detailed below.
![Initial simulation setup showing "Inferred" status](01-initial-setup-inferred.png)

## Troubleshooting Process

### 1. Meshing Warning
Running Model Setup Check flagged: *"Force is applied to geometry that is 
partially meshed or not meshed at all... due to a tiny edge."* The imported 
CAD contained a microscopic sliver edge smaller than the mesh element size, 
so no elements could form there — meaning the load had no valid path into 
the structure.
![Mesh warning showing Force applied to partially meshed geometry](02-mesh-warning.png)

### 2. Geometry Was Off-Limits
Attempted to fix the tiny edge via Examine Geometry / Idealize Geometry — 
both were unavailable. The imported bodies turned out to be **Polygon 
Bodies** (faceted geometry, not true CAD solids), so standard geometry-repair 
tools didn't apply.
![Polygon Body tree showing faceted geometry, not a CAD solid](03-polygon-bodies.png)

### 3. Workaround: Load Applied to Mesh, Not Geometry
Since the geometry couldn't be edited directly, the force load was deleted 
and recreated by selecting **mesh nodes** instead of a geometry face — this 
sidesteps the unmeshed-geometry problem entirely, since a node-based load 
doesn't depend on the underlying geometry being fully meshed.

![Force dialog with 101 nodes selected on the flat floor](04-node-force-selection.png)

### 4. Correcting the Load Physics
Selecting the right nodes required revisiting the original assignment brief, 
which clarified two things:
- The applied force (220.7 N) corresponds to 15 kg × 1.5 safety factor × 
  gravity — the server's weight plus safety margin.
- The ribbed section of the shelf was actually an **air-circulation louver 
  feature**, not a load-bearing surface — the servers don't rest on it.
- The true load points were on the flat floor near the front of the shelf, 
  matching the assignment's reference figure.

  ![Reference photo showing where the server unit actually contacts the shelf](06-product-reference-photo.png)

Two compact node clusters were selected on solid floor material (avoiding 
the perforated louver holes), and the total load was split into two 110.35 N 
forces to represent the two bearing points shown in the assignment figure.

![Two node clusters selected on solid floor material, avoiding the louver holes](05-node-clusters.png)

