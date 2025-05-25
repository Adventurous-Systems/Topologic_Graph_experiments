# Product Requirements Document - Digital Twin

## Summary

A modular Jupyter notebook will demonstrate a “golden-thread” diff of a BIM model over time by:

1. Loading an IFC into TopologicPy
2. Converting it to a discrete‐element graph
3. Serializing and timestamping the first snapshot
4. Editing the graph (removing and adding vertices)
5. Serializing and timestamping the second snapshot
6. Computing and visually highlighting the per-element differences

---

## Goals & Scope

**Goal:** Provide an end-to-end notebook showing how a BIM model evolves, with immutable snapshots and clear visual diff.

**Scope:**

* **IFC ingestion** via `Graph.ByIFCPath`
* **Graph export** to JSON + SHA-256 hash
* **Topology edits** (remove/add vertices)
* **Automatic diff** of two graph states
* **Visual highlighting** of added vs. removed elements

---

## User Stories

* **As a BIM engineer**, I want to load an IFC into Topologic so I can programmatically query its structure.
* **As a data analyst**, I want to export that graph to JSON and compute a content hash, so I can guarantee immutability.
* **As a developer**, I want to remove and add discrete elements using TopologicPy’s topology primitives.
* **As a QA engineer**, I want an automatic diff of before/after snapshots so I can audit exactly what changed.
* **As a project manager**, I want to see visual highlights of added vs. removed elements in the Topologic viewer.

---

## Functional Requirements

1. **Load IFC**

   * Notebook shall load an IFC file using

     ```python
     Graph.ByIFCPath(ifc_path, includeTypes=…, …)
     ```

     to produce a `Graph` object.

2. **Discrete Graph**

   * The IFC graph must consist of discrete vertices (spaces, walls, slabs, etc.), preserving GUIDs and IFC types.

3. **Timestamp First Graph**

   * Serialize the initial graph via

     ```python
     Graph.JSONString(g1)
     ```

     compute `SHA-256(json)`, and save both the JSON file and a metadata file containing `{hash, timestamp}`.

4. **Edit Graph**

   1. **Remove a vertex**

      * Use a subgraph or `Topology.RemoveVertices(...)` to drop a specified vertex.
   2. **Add a new vertex**

      * Create a new `Vertex` with `Vertex.ByCoordinates(...)`, assign a unique `"id"` dictionary, and insert it via `Graph.AddVertex(...)`.

5. **Timestamp Modified Graph**

   * Repeat serialization, hashing, and metadata write-out for the edited graph.

6. **Compute Diff**

   * Retrieve raw graph data with

     ```python
     Graph.JSONData(g1), Graph.JSONData(g2)
     ```
   * Compute sets:

     ```python
     added_vertices = v2 – v1  
     removed_vertices = v1 – v2  
     added_edges = e2 – e1  
     removed_edges = e1 – e2
     ```

7. **Visualize Diff**

   * Highlight removed vertices on the original graph and added vertices on the modified graph using

     ```python
     Topology.Show(selectors, graph, …)
     ```

---

## Non-Functional Requirements

* **NFR1:** Runs entirely in a standard Jupyter environment (Python 3.10+, Linux or Windows).
* **NFR2:** Depends on TopologicPy ≥ 0.8.28, ifcopenshell ≥ 0.7.9, pandas ≥ 1.4.2.
* **NFR3:** Notebook cells must be idempotent and re-runnable end-to-end.
* **NFR4:** All JSON snapshots are content-hashed with SHA-256 to ensure data integrity.
* **NFR5:** For IFC files ≤ 1 MB, end-to-end execution should complete within 30 s on a modern laptop.

---

## Notebook Architecture

1. **Cell 1:** Imports & helper functions (`timestamp_graph`, `diff_graphs`)
2. **Cell 2:** `build_graph()` → load IFC → `g1`
3. **Cell 3:** Timestamp & save `g1` JSON + metadata
4. **Cell 4:** Edit graph (remove + add vertices) → produce `g2`
5. **Cell 5:** Timestamp & save `g2` JSON + metadata
6. **Cell 6:** Compute diffs (`diff_graphs`) → added/removed sets
7. **Cell 7:** Display summary via pandas DataFrame
8. **Cell 8:** Visualize differences in the Topologic viewer

---

## Acceptance Criteria

* **AC1:** Two JSON exports (`graph1.json`, `graph2.json`) and two meta files are produced.
* **AC2:** The diff summary reports exactly one removed and one added vertex.
* **AC3:** The Topologic viewer highlights the removed element on the original graph and the added element on the modified graph.
* **AC4:** No unhandled exceptions occur during notebook execution.
* **AC5:** The notebook is fully documented with inline explanations per cell.

---

## References

* TopologicPy package overview and Graph module: [https://topologicpy.readthedocs.io/en/latest/topologicpy.html](https://topologicpy.readthedocs.io/en/latest/topologicpy.html)
* IFC import via ifcopenshell: [https://ifcopenshell.org/](https://ifcopenshell.org/)
* Topology editing in TopologicPy: [https://topologicpy.readthedocs.io/en/latest/topologicpy.Topology.html](https://topologicpy.readthedocs.io/en/latest/topologicpy.Topology.html)
* Graph export and JSON APIs: [https://topologicpy.readthedocs.io/en/latest/topologicpy.Graph.html](https://topologicpy.readthedocs.io/en/latest/topologicpy.Graph.html)
* Viewer usage: [https://topologicpy.readthedocs.io/en/latest/topologicpy.Topology.html](https://topologicpy.readthedocs.io/en/latest/topologicpy.Topology.html)
