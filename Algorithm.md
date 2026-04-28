# 📦 Hack the Flow: **Inditex** Logistics Challenge

## 🏭 Project Overview
This repository contains our algorithmic solution for **Hack the Flow**, an Inditex hackathon challenge focused on supply chain optimization. At the forefront of modern logistics, distribution centers operate like living organisms, relying on automated silos as their central lungs to manage goods shipped to thousands of stores. 

This massive infrastructure holds hundreds of thousands of boxes, presenting a complex technical challenge: orchestrating the perfect choreography of incoming and outgoing box movements to minimize response times and maximize system agility.

---

## 🏗️ Architecture & Constraints

### The Silo Grid
The warehouse silo is organized as a high-density three-dimensional grid where each unit load must be perfectly located.
* **Aisles:** 4 total aisles.
* **Sides:** 2 per aisle (Left: 01, Right: 02).
* **X-Coordinate (Length):** 60 positions (from 1 to 60) extending from the head (entrance).
* **Y-Coordinate (Height):** 8 vertical levels.
* **Z-Coordinate (Depth):** 2 depth positions per shelf (1: shallow, 2: deep).

**Position Tracking:** Each location is defined by an 11-digit string formatted as `AISLE_SIDE_X_Y_Z` (e.g., `01_02_003_04_01`). Boxes are tracked via a 20-digit code containing their Source, Destination, and Bulk number.

### Operational Rules
* **The Z-Rule:** You cannot place a box at z=2 if z=1 is occupied. Similarly, retrieving a box from z=2 requires relocating the box at z=1 first.
* **Shuttles:** There is exactly one shared shuttle per Y-level (height) that handles both incoming (storage) and outgoing (retrieval) boxes.
* **Time Cost:** The time to execute a movement is calculated as `t = 10 + d`, where 10 seconds is the fixed handling time (pick/drop) and `d` is the distance traveled along the X-axis.
* **Palletizing:** Boxes are shipped by consolidating them into pallets. A complete pallet consists of 12 boxes with the exact same destination. 
* **Robot Capacity:** The system uses 2 robots that can manage up to 8 active pallets simultaneously.

---

## 🧠 The Algorithm: Interleaved "On-the-Way" Routing

Our core strategy replaces standard sequential processing with an **Interleaved Storage-Delivery** model. To drastically reduce empty shuttle trips ("dry runs") and minimize the `t = 10 + d` time penalty, we enforce a strict 1:1 ratio: **for every box stored, one box is delivered.**

### Execution Flow
1. **Initial Pickup:** The shuttle begins at the head (X=0) and picks up an incoming box.
2. **Instant Pallet Check:** The system instantly evaluates if the box's destination matches any of the 8 active pallets currently managed by the robots.
   * *If Match:* The shuttle immediately delivers the box to the pallet queue, bypassing storage entirely.
   * *If No Match:* The box must be stored.
3. **Targeted Delivery:** The system identifies the next outgoing box that needs to be retrieved from the silo.
4. **"On-the-Way" Drop-off:** Instead of making a dedicated trip to store the incoming box, the shuttle carries it toward the target retrieval box. It drops the incoming box off *on the way* using a strict priority heuristic.

### Storage Priority Heuristics
When deciding exactly where to drop the incoming box "on the way" to the retrieval target, the algorithm prioritizes the following slots:

* **Priority 1 (Avoid Relocation):** Store in z=1 *only if* the box currently in z=2 shares the same destination. This guarantees we won't have to waste time relocating the z=1 box later.
* **Priority 2 (Deep Fill):** Store in an empty z=2 position (provided z=1 is also empty/can be bypassed) to keep the shallow z=1 slots open for future agility.
* **Priority 3 (Forward Search):** If no ideal spots exist *before* the retrieval target on the X-axis, look for slots fitting Priority 1 or Priority 2 slightly *after* the retrieval target. 
* **Priority 4 (Proximity Fallback):** If all else fails, drop the box in the closest available z=1 slot relative to the retrieval target to minimize the travel distance (`d`) between the drop-off and the immediate next pick-up.

---

## 🚀 Success Metrics

This routing logic is designed to maximize the hackathon's core evaluation criteria:
* **Total Operation Time:** Minimized by coupling storage and retrieval actions in single trips.
* **Throughput:** Increasing the number of pallets completed per unit of time by dynamically prioritizing boxes that finish active pallets.
* **Full Pallets:** Ensuring a high percentage of successfully consolidated 12-box sets by actively checking incoming destinations.
