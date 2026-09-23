---
name: write-explained-code
description: Write new or substantially rewritten code with concise contextual comments inside the source file. Use whenever the user asks Codex to create a script, write code from scratch, complete comments or pseudocode, fill a minimal skeleton, or add substantive code to an existing implementation.
---

# Write Explained Code

Make code understandable in place. Apply these rules to code added or
substantially rewritten during the task; do not annotate unrelated existing
code unless the user asks.

## No silent handling

Do not silently handle unexpected, unsupported, or unspecified cases. If
handling such a case requires a choice that could alter observable behavior or
materially affect the result, ask the user how to proceed. If the case is
intentionally unsupported or cannot be resolved through clarification, surface
it with an appropriate, clear error. Do not introduce implicit defaults,
fallbacks, coercion, retries, exception suppression, or automatic recovery
unless explicitly requested or required by the existing specification.

## Prefer simple execution flow

Prefer a simple, explicit top-to-bottom execution flow. Keep straightforward
logic inline by default, especially in one-off analysis scripts and launchers.

Extract a function only when it clearly improves at least one of the following:

- Reuse across multiple call sites.
- Readability of otherwise complex logic.
- Independent testing.
- Separation of distinct responsibilities.
- Isolation of complex branching or error handling.

Avoid thin helper functions that are called once and merely rename a short,
straightforward block. Do not introduce abstraction solely in anticipation of
possible future reuse.

## Comment meaningful operations

- Use a cohesive conceptual step or code block as the default unit of
  explanation. Add one concise comment immediately above each meaningful block,
  using the language's native comment syntax.
- Describe what the block accomplishes in the surrounding workflow and why it
  is needed when that is not obvious. Include important assumptions or
  consequences needed to understand the result; do not invent a rationale or
  translate syntax into prose.
- Do not comment on every line. When a block performs multiple distinct tasks,
  prefer splitting it into clearly named intermediate operations and comment
  each task separately. If the tasks must remain in one block, add internal
  comments only at the boundaries between the distinct subtasks. Do not add
  comments to blank, continuation-only, formatting-only, or closing-delimiter
  lines.
- Use module and function docstrings for overall purpose, inputs, outputs, and
  terminology. Use inline comments for key implementation steps.
- Preserve relevant comments supplied by the user.

For example:

```r
# Assemble one ordered table so diagnostics remain aligned with PC order.
model_diagnostics_df <- dplyr::bind_rows(model_diagnostic_rows)
```

```python
# Fit once so dynamic-programming subproblems can be reused across values of K.
dynamic_program = rpt.Dynp(
    model=cost_model,
    min_size=min_segment_size,
    jump=1,
).fit(signal)

# Find the optimal changepoint locations for every feasible changepoint count.
for k in range(max_changepoints + 1):
    breakpoints = dynamic_program.predict(n_bkps=k)
```

## Structure R and Python analysis scripts

- Use `# %%` cell headings to organize new or substantially rewritten R and
  Python analysis scripts. Begin with a concise description of the script's
  purpose, followed by an optional runtime-environment comment when applicable:

```text
# %% <Script purpose>
# env: <environment>

<imports or package loading>

# %% Parameters

# %% Functions

# %% ----- MAIN -----

# %% Load inputs

# %% Validate inputs

# %% Transform data

# %% Generate outputs
```

- Keep imports or package loading near the top without adding a `# %% Imports`
  heading.
- Omit the environment comment, `Functions`, or another section when it does
  not apply; do not retain empty sections.
- Under `MAIN`, add descriptive `# %%` headings for the major workflow stages.
  Name sections after their purpose rather than using generic numbered steps.
- Adapt the main-section names to the actual workflow; the example names are
  illustrative rather than mandatory.

## Anchor repository Python scripts

- For repository-bound Python scripts that use relative paths, import `os` and
  `subprocess`, then set the working directory to the Git repository root
  immediately after the imports and before parameters or file access:

```python
os.chdir(
    subprocess.check_output(
        ["git", "rev-parse", "--show-toplevel"], universal_newlines=True,
    ).strip()
)
```

- Use this as the default working-directory setup instead of relying on the
  directory from which the script was launched.
- Do not add process-wide working-directory changes to importable library
  modules unless the user explicitly requests them.

## R style

- Follow the [tidyverse style guide](https://style.tidyverse.org) for R code
  by default.
- Explicit user instructions override the guide where they conflict.
- In tidyverse data-masked expressions, use `.data` for columns and `.env` for
  external variables or function arguments. Validate consequential filters
  with multiple values to catch accidental self-comparisons such as
  `column == column`.
- As a deliberate exception to the guide, in an R script's `# %% Parameters`
  section, assign user-editable parameters with `=` rather than `<-`. Treat
  this distinction as a visual contract that identifies values the user is
  expected to configure.

- This parameter-assignment convention applies only to R. Do not add an
  analogous convention or explanatory note to Python or other languages; use
  their normal assignment style.

## Make generated results traceable to their code

- Whenever code generates outputs, save a machine-readable `run_metadata.json`
  in the same output directory. Its purpose is to record everything needed to
  rerun the analysis and reproduce the results: all input paths, parameter
  values actually used, including defaults, and any inputs selected through
  patterns or filters. Do not record passwords, access tokens, API keys, or
  other credentials.
- Name a generated results directory after the script that creates it, using
  the script basename without its extension.
- Place analysis-, contrast-, or dataset-specific identifiers beneath that
  script-named directory when multiple runs share the same engine. For example,
  `results/figures/prepare_module_ora_inputs/<analysis>` traces back to
  `prepare_module_ora_inputs.R`.
- Apply the same convention when a figure-specific launcher calls a generic
  engine: use the generating engine's basename for the results directory and
  retain the figure or analysis identity in the enclosing path or child name.
- When the output directory already identifies the analysis, contrast,
  dataset, or input, keep filenames within it generic. Do not hard-code or
  repeat that identifier in every filename; the directory provides the run
  context while generic filenames allow the same script to serve other inputs.
- Whenever code generates a plot, also write a CSV containing the final
  plot-ready data needed to recreate it directly without rerunning upstream
  transformations, aggregation, normalization, ordering, or other analysis.
  Include derived values and ordering or coordinate fields that affect the
  rendering. Name the CSV from the plot stem with the suffix `_plot_data.csv`;
  for example, `summary.pdf` or `summary.svg` uses `summary_plot_data.csv`.
  Multiple formats of the same plot share one companion CSV.
- Do not remove rows from a saved plot-data CSV solely because a display
  filter controls which elements are shown in the current plot. Save the
  complete pre-display-filter universe, add one clearly named boolean for each
  display filter plus an overall `included_in_plot` flag, and apply those flags
  only to the in-memory data passed to the plotting layer. This rule concerns
  display choices; an analysis-defining cohort, quality-control exclusion, or
  other scientific eligibility rule remains part of the analysis when the
  user specifies it as such.
- When a display filter changes an aggregation, normalization denominator,
  ranking, ordering, or another derived value, retain both the complete-data
  metric and the exact plot-specific metric in the unfiltered plot-data CSV.
  The CSV must contain enough information to reproduce the current rendering
  and to revise display filters later without recovering deleted rows.
- Whenever code generates a plot, include compact interpretation text in the
  plot itself. State the analysis decisions and contextual details a reader
  needs to interpret the figure without consulting the source code, such as
  the observation unit, metric definition, transformations or normalization,
  inclusion thresholds, missing-data or empty-category behavior, and
  non-obvious aesthetic encodings. Use a small subtitle or caption and split
  long text across multiple lines so it does not crowd the plotting area.
- Update launchers, downstream consumers, and documentation together whenever
  an output path changes.
- Preserve a user-specified path or established external output contract when
  it takes precedence over this default convention.
