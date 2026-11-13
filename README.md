# Taxonomy Explorer

An interactive tool for exploring subclass relations and comparing predictions against a reference dataset (*gold standard*). Visualize correct matches and errors, search for concepts, and calculate precision (P), recall (R), and F1.

The tool runs directly in your browser as a single HTML file. No installation, server, or external dependencies are required. It includes a **MatOnto Top 10%, epoch 1** example and supports other datasets, such as OBI.

## Repository contents

| Path | Contents |
| --- | --- |
| `taxonomy-explorer.html` | Interactive visualizer with an embedded example. |
| `results/` | Gold-standard and prediction JSON files. |
| `README.md` | Usage instructions and evaluation criteria. |

JSON files can have any name. The tool does not automatically load the `resultados/` folder: select files through the interface.

## Getting started

1. Download the repository and open `matonto-grafo-interactivo.html` in a modern browser.
2. Under **Load files**, select the reference JSON in **Gold (.json)** and the model output in **Predictions (.json)**.
3. Choose **Compare gold and predictions** and click **Load selection**.
4. Explore the graph, inspect the metrics, or export the visible relations to CSV.

You can also choose **Gold only** or **Predictions only** to explore a single file. P, R, and F1 are not calculated in these modes. **Restore example** reloads the embedded MatOnto Top 10%, epoch 1 example.

Selected files are processed locally in your browser.

## JSON format

Each file must contain an array of objects with non-empty `parent` and `child` strings:

```json
[
  {"parent": "material", "child": "metal"},
  {"parent": "metal", "child": "copper"}
]
```

- `parent`: superclass.
- `child`: subclass.
- `ID` and other additional fields are optional.
- If provided, `score` must be a finite number between 0 and 1. It is not used to filter predictions or weight the metrics.

Each prediction object counts as one predicted relation. The two files do not need to contain the same number of records.

## Visualization

Arrows point from **subclass → superclass**.

| Color | Meaning |
| --- | --- |
| Green | Relation present in both the gold standard and the predictions. |
| Dashed red | Gold-standard relation missed by the model. |
| Orange | Prediction absent from the gold standard. |
| Blue | Relation in **Gold only** mode. |
| Purple | Relation in **Predictions only** mode. |

A prediction absent from the gold standard is not necessarily semantically false. Node positions aid readability and do not represent taxonomic depth.

### Exploration modes

- **Main nodes:** displays the nodes with the most connections and the relations between the chosen nodes. Enabled automatically when the dataset exceeds 800 nodes or 2,000 unique relations.
- **Selected concept neighborhood:** displays the selected concept and its direct neighbors, along with relations between the included nodes. If no concept is selected, it uses the node with the most connections.
- **Full graph:** displays all relations allowed by the active filters.

Reduced views support a maximum of **50, 100, 200, or 300 nodes**. Node importance is determined by the number of connections across the full set of unique relations.

### Controls

- **Search for a concept:** filters the concept list by name, showing up to 200 matches ordered by connection count.
- **All:** a fixed option at the top of the concept list that clears the search and selection and returns to an overview.
- **Visible relations:** toggles correct matches, missed relations, and predictions absent from the gold standard.
- **Only connections of the selected concept:** restricts the view to that concept's direct relations.
- **Errors only / All:** changes the correct-match and error filters.
- **Show all names:** displays labels for all visible nodes.
- **Clear selection:** clears the selected concept and returns to an overview.
- **Fit view:** centers and scales the visible nodes.
- **Mouse wheel:** zooms in or out.
- **Drag the background / a node:** pans the view or moves a node.
- **Click a node or table row:** selects a concept.
- **Export visible CSV:** downloads the relations in the current view.

The table displays up to 500 relations. The CSV includes all visible relations, even when the table reaches that limit.

## Calculating P, R, and F1

Metrics follow the evaluator used for the results:

1. Convert `parent` and `child` to lowercase, without trimming whitespace.
2. Build sets of unique gold-standard and predicted pairs.
3. Count the pairs in their intersection as correct matches.
4. Divide the correct-match count by the **total number of records** in each file, including duplicates.

Let `C` be the number of unique correct matches, `N_pred` the number of prediction records, and `N_gold` the number of gold-standard records:

```text
P  = C / N_pred
R  = C / N_gold
F1 = 2 × P × R / (P + R)
```

P or R is 0 when its denominator is 0. F1 is 0 when P + R = 0. The interface displays three decimal places.

**Duplicates count toward the denominators, but each pair can contribute only one correct match and is drawn only once.** Consequently, the graph's unique-relation counts do not always reproduce the metric denominators. For example, if the gold standard contains three records representing two unique pairs, and the predictions contain each of those pairs once, P = 1, R = 2/3, and F1 = 0.8.

No transitive closure, relation inference, or additional name normalization is applied. Pair direction matters.

**Metrics always use all loaded records.** Search, filters, and node limits affect only the visualization.

## Limits

The loaded dataset can contain up to **10,000 concepts and 100,000 unique relations**. Use reduced views for large graphs; the full graph may be slower and harder to read.
