# How to Use This Docs

## Doc Layers

```
components  ──►  patterns  ──►  config-paths
   (atoms)        (molecules)     (recipes)
```

### 1 · Components (`/10-components`)

Self-contained descriptions of a single capability or service feature.  
**Read them when** you need to understand what something is, how to configure it in isolation, and how to validate it.  
Components do **not** prescribe how multiple capabilities are wired together.

### 2 · Patterns (`/20-patterns`)

Opinionated, composable architecture designs that combine 2+ components.  
**Read them when** you need to understand how pieces connect and what the trade-offs are.  
Patterns include Mermaid architecture diagrams and validation checklists.

### 3 · Config Paths (`/30-config-paths`)

Thin, numbered guides that assemble specific components and patterns into a step-by-step path for a named scenario (e.g., "OOB connector + public network + mcpgenie").  
**Read them when** you know your target configuration and just need the ordered steps.  
Config paths contain **links, not repeated text**.

### 4 · Decision Guides (`/40-decision-guides`)

Flowcharts and decision trees to help you choose between options.  
**Start here** if you are unsure which connector type or network path to use.

### 5 · Matrix (`/50-matrix`)

A single table that maps every combination of dimensions to the relevant config path.  
**Use it** for a quick orientation or to verify that your scenario is supported.

## Navigation Tips

- Every component and pattern file uses consistent headings so you can scan by section.
- All links are **relative** — they work on GitHub and any static site renderer.
- If a value is not yet known or confirmed, it is marked `TODO: confirm`.

## Contributing

- Add a new component when a reusable building block is missing.
- Add a new pattern when a new combination of components is validated.
- Add a new config path when a specific end-to-end scenario is confirmed working.
- Update the matrix whenever a new config path is added.
