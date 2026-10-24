## Functional Design Document — Manufacturing Line (CPG)

Purpose
-------

This functional design document (FDD) defines the functional requirements, architecture, interfaces, data flows, control logic, safety considerations, test and deployment plans for a manufacturing line in a Consumer Packaged Goods (CPG) facility. It is intended for engineering, automation, operations, QA, IT (MES/ERP), and stakeholders responsible for design, integration, commissioning, and maintenance.

How to use this template
------------------------

- Fill in plant- and line-specific information in each section. Use the "Examples / Notes" hints when helpful.
- Replace diagram placeholders with actual diagrams (Mermaid, draw.io, Visio exports, or images). Keep file references or links in the "Diagrams" section.
- For PLC/SCADA I/O and MES interfaces, append detailed tables or CSV attachments as needed.

1. Document metadata
--------------------

- Version: 0.1
- Author:
- Date:
- Reviewers:
- Status: Draft / Review / Approved
- Related documents: (e.g., P&ID, Electrical Single Line, Standards, Safety regs)

2. Executive summary
--------------------

- Short description of the line (product family, throughput targets, packaging type)
- Business goals and KPIs (OEE, throughput, yield, changeover time, scrap rate)

3. Scope
--------

- In scope: (e.g., filling station, capping, labeling, case packing, palletizing)
- Out of scope: (e.g., utilities, warehouse systems)
- Assumptions (e.g., existing MES present, network infrastructure available)

4. Stakeholders & roles
-----------------------

- Business owner / Product owner
- Manufacturing engineer
- Automation engineer (PLC, HMI)
- Integration / MES engineer
- QA / Quality engineer
- IT / Network
- Safety / EHS
- Maintenance

5. Background & context
-----------------------

- Plant layout reference (link to floor plan)
- Existing automation or legacy systems to integrate with
- Known constraints (space, budget, regulatory)

6. Goals and success criteria
-----------------------------

- High-level goals (e.g., achieve 99% on-time production; reduce manual inspection by X%)
- Success metrics and acceptance criteria (numeric, measurable). Example:
  - Throughput: >= 1200 units/hour measured over 8-hour shift
  - OEE target: >= 85% after 90 days of operation
  - Quality: defect rate < 0.5% for critical defects

7. System overview / Architecture
---------------------------------

- Short text overview describing major subsystems:
  - Field layer: sensors, actuators, drives, safety relays
  - Control layer: PLC(s), safety PLC, HMI/Operator panels
  - Supervisory layer: SCADA, Historian
  - Execution layer: MES (production orders, genealogy, traceability)
  - Enterprise layer: ERP
  - Network and cybersecurity

- Diagram placeholders (replace with images or Mermaid):

  - Topology diagram (physical & logical)

  Example Mermaid placeholder:

  ```mermaid
  graph LR
    A[ERP] --> B[MES]
    B --> C[SCADA/Historian]
    C --> D[PLC]
    D --> E[Field Devices]
    C --> F[HMI]
  ```

8. Functional components and responsibilities
-------------------------------------------

- Subsystem: Conveyors
  - Purpose: move product between machines, singulate, buffer
  - Inputs: start/stop, speed setpoints, product detected sensors
  - Outputs: start/stop commands, encoder feedback, fault signals
  - Acceptance: belt runs at set speed ±5% under full load

- Subsystem: Filling station
  - Purpose: dispense correct volume with accuracy
  - Inputs: product ID, recipe, start/stop, level sensors
  - Outputs: valve commands, dispensed volume log
  - Controls: closed-loop control for volumetric dosing

- Subsystem: Capping/Sealing
  - Purpose, inputs, outputs, quality checks (torque, presence)

- Repeat for Labeling, Inspection (vision), Case packing, Palletizing, etc.

9. Control strategies and sequences
----------------------------------

- Provide high-level PLC state machine(s) and key sequences such as:
  - Line start/stop, emergency stop handling
  - Product changeover sequence
  - Recipe download and verification
  - Manual/automatic modes

- Example state machine (Mermaid):

  ```mermaid
  stateDiagram-v2
    [*] --> Idle
    Idle --> Starting: operator start
    Starting --> Running: checks OK
    Running --> Paused: operator pause
    Running --> Fault: fault
    Fault --> Idle: reset
  ```

10. Data model and information flows
-----------------------------------

- Events and messages to MES/ERP/SCADA (examples):
  - Production start/stop (timestamped)
  - Batch/lot assignment
  - Per-unit traceability data (serial IDs), if applicable
  - Quality inspection results
  - Downtime and reason codes

- Suggested data schema table (example):

  | Field | Type | Description |
  |---|---|---|
  | productionOrderId | string | MES order identifier |
  | productCode | string | SKU |
  | timestamp | ISO8601 | event time |
  | deviceId | string | PLC or station id |
  | status | enum | start/stop/fail/complete |

- Data retention and historian policy (e.g., keep raw sensor samples at 1s for 90 days)

11. Interfaces and integration
------------------------------

- PLC <-> SCADA/OPC UA
  - Protocol: OPC UA / Modbus-TCP / Ethernet/IP
  - Tags/messages: provide a mapping table (append PLC I/O list)

- SCADA <-> MES
  - Protocol: REST API / MQTT / OPC UA A&E / Custom socket
  - Payloads: production events, genealogy, quality measurements

- MES <-> ERP
  - Expected interactions: Material issue, finished goods confirmation

- Table template: Interface / Endpoint / Protocol / Message format / Direction / SLA / Retry

12. PLC & I/O mapping
---------------------

- Summarize PLC racks, CPU part numbers, I/O modules, and network addresses.
- Provide or link to a detailed I/O table: signal name, type (DI/DO/AI/AO), address, description, safety critical?

13. Safety and EHS requirements
-------------------------------

- Safety devices (light curtains, E-stops, safety relays, safety PLC) and their placement
- How safety logic interacts with control logic (e.g., safe stop vs immediate stop)
- Interlocks, lockout/tagout procedures
- Regulatory standards to satisfy (OSHA/ISO/IEC 61508/62061 etc.)

14. Alarms & HMI requirements
----------------------------

- Alarm categories: Warning, Fault, Safety
- Example alarm table: code / text / severity / operator actions
- HMI screens needed: Overview/Dashboard, Alarms, Recipes, Maintenance, Diagnostics

15. Quality and inspection
-------------------------

- Vision system checks (presence, print/readability), inline sensors, and sampling points
- Acceptance criteria and gating rules (e.g., reject after X consecutive fails)
- Data capture for each inspected unit (which fields are captured to MES/historian)

16. Testing & validation plan
----------------------------

- Unit tests for PLC programs (function blocks), test cases for each sequence.
- FAT (Factory Acceptance Test) checklist: functional sequences, safety, interface tests, performance.
- SAT (Site Acceptance Test) checklist: as-built verification, environmental tests, integration with MES/ERP.
- Acceptance sign-off criteria and responsible signers.

17. Deployment & commissioning
------------------------------

- Staging environment (FAT lab) and what is required to replicate production.
- Rollout plan: phases, fallback/rollback steps, expected downtime.
- Backup and version control (PLC source, HMI screens, configurations). Recommend tagging versions in a repository.

18. Monitoring, telemetry & maintenance
-------------------------------------

- KPIs to monitor (OEE, line speed, downtime, mean time between failures)
- Alerts & dashboards (SCADA + enterprise dashboards)
- Routine maintenance tasks and intervals, spare parts list

19. Security
-----------

- Network segmentation and firewall rules between enterprise and OT networks
- User roles and authentication for HMI/SCADA/MES
- Secure update process for PLC/HMI firmware and application code

20. Reliability, availability & scalability
----------------------------------------

- Availability targets, redundant components (redundant PLC, dual network paths)
- Expected failure modes and mitigation strategies

21. Risk assessment and mitigations
----------------------------------

- List major risks (supply chain, single-vendor PLC, single point of failure) and proposed mitigations

22. Open issues / decisions required
----------------------------------

- Decision: PLC vendor and programming standard
- Decision: MES vendor & integration pattern
- Pending mechanical layout sign-off

23. Appendix & artifacts
------------------------

- Diagrams (links or file names):
  - Floorplan — /diagrams/floorplan.png
  - Network topology — /diagrams/network_topology.png
  - PLC rack layout — /diagrams/plc_rack.pdf
  - I/O matrix — /docs/plc_io.csv

- Glossary of terms
- Compliance and reference documents

24. Fill-in checklist (quick)
---------------------------

Complete before FAT:

- [ ] PLC I/O table attached
- [ ] MES interface spec attached
- [ ] Safety plan reviewed and signed
- [ ] HMI screens mock-ups provided

25. Next steps & recommended deliverables
----------------------------------------

1. Populate the "System overview" and "PLC & I/O mapping" sections with plant-specific data.
2. Produce the diagrams referenced in the Appendix. Prefer vector exports and link them here.
3. Prepare FAT scripts and test cases (use sections 9 and 16 to build them).
4. Schedule stakeholder review and update version & sign-off fields.

---

Notes and examples
------------------

- Use this template as the canonical design artifact for integration. Small facilities may compress sections; larger projects should expand each subsystem into its own design sub-document (e.g., "Filling station FDD").
- If you want, I can generate: a) a populated example for a small bottling line, b) a CSV PLC I/O template, or c) Mermaid sequence diagrams for changeover and start/stop workflows.
