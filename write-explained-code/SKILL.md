---
name: write-explained-code
description: Write new or substantially rewritten code with concise contextual comments inside the source file. Use whenever the user asks Codex to create a script, write code from scratch, complete comments or pseudocode, fill a minimal skeleton, or add substantive code to an existing implementation.
---

# Write Explained Code

Apply these rules when adding new code or substantially editing existing code. Do not apply them to existing code that is only read, reviewed, or debugged.

### General implementation rules

##### No silent handling

Do not silently resolve unexpected, unsupported, or unspecified cases when the choice could materially affect the result or observable behavior. Ask the user when their intent is needed to make that choice. If a case is unsupported or cannot be resolved, fail clearly rather than hiding the problem.

Do not introduce implicit defaults, fallbacks, coercion, retries, exception suppression, or automatic recovery unless explicitly requested by the user.

##### Prefer simple implementations

- Use the simplest clear implementation that meets the current requirements.
- Keep the execution flow explicit and easy to follow from top to bottom,
  especially in one-off analysis scripts and launchers.
- Keep short, one-use logic inline. Extract a function when it clearly improves
  reuse, readability, testing, separation of responsibility, or handling of
  genuine complexity.
- Avoid thin one-use helpers. Do not add classes, abstraction layers,
  configuration, generalisation, or extension points for hypothetical future
  needs.
- Do not create auxiliary outputs beyond those required by the traceability
  rules below unless the user requests them or they are clearly needed for
  reproducibility, validation, or downstream use. Keep any such outputs minimal
  and non-redundant.
- Prefer the clearer implementation when both are practical at the expected
  data scale. Add complexity for performance only when runtime or memory
  benefits are meaningful.

##### Comment meaningful operations

- Add concise comments above meaningful blocks of code.
- Write comments so that reading only the section headings and block comments
  is enough to understand the script's overall flow, major steps, and
  progression from inputs to outputs.
- Within each section, comment the meaningful intermediate steps needed to
  follow how that section works. Do not rely on a single comment at the start
  of a long section when several distinct operations occur within it.
- Describe each block's purpose concisely, using concrete, plain language that
  a reader unfamiliar with the workflow can understand. Name the actual data
  objects and relationships, and avoid vague workflow terminology.

  For example, avoid:

  `# Resolve paired inputs and validate their metadata contracts.`

  Prefer:

  `# Load the raw and transformed feature tables with their metadata. Require`
  `# every sample in each table to appear in its corresponding metadata.`
- Include assumptions, decisions, or consequences when they help explain
  interpretation or why a step is necessary.
- Group closely related statements under a single comment. Avoid redundant
  comments that merely translate obvious code into prose.
- Use module and function docstrings for overall purpose, inputs, outputs, and
  important terminology.
- Preserve relevant comments supplied by the user.

For example:

```python
# %% Validate runs before combining them

# Require one run per analysis configuration so no configuration receives
# extra weight in the pooled result.
duplicate_runs = run_parameters_df.duplicated(configuration_columns, keep=False)
if duplicate_runs.any():
    raise ValueError("Duplicate analysis configurations found.")

# Hold the remaining analysis settings constant so the combined runs differ
# only in the parameters being compared.
for column in fixed_parameter_columns:
    if run_parameters_df[column].nunique(dropna=False) != 1:
        raise ValueError(f"Runs use multiple values of {column}.")
```

```python
# %% Summarise stability

# Count each feature once per resample before calculating how often it recurs
# across data perturbations.
feature_counts_df = (
    resample_results_df.groupby("feature", as_index=False)
    .agg(n_selected_resamples=("resample_id", "nunique"))
)

# Use every successful resample as the denominator so absence from a resample
# contributes to the reported selection frequency.
feature_counts_df["selection_frequency"] = (
    feature_counts_df["n_selected_resamples"] / n_successful_resamples
)
```

Avoid comments that merely repeat the syntax:

```python
# Group by feature.
feature_groups = results_df.groupby("feature")
```

Prefer comments that explain the operation's role in the workflow:

```python
# Combine observations for each feature before calculating stability across
# resamples.
feature_groups = results_df.groupby("feature")
```

### Analysis-script and output conventions

Apply these sections when creating or substantially editing analysis scripts
or code that generates result files.

##### Structure R and Python analysis scripts

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

##### Make generated results traceable to their code

- For each generated results directory, if requested by user, save a machine-readable
  `run_metadata.json` containing the input paths, parameter values actually used
  including defaults, and inputs selected through patterns or filters. Do not
  record credentials.
- Name results directories after the generating script using its basename
  without the extension. When the same script serves multiple analyses,
  contrasts, or datasets, place those identifiers beneath the script-named
  directory.
- When the directory already provides the run context, keep filenames generic
  rather than repeating the same identifiers.
- For each plot, save one `<plot_stem>_plot_data.csv` containing the final data
  needed to recreate or revise the plot without rerunning the upstream analysis,
  for example when preparing the figure for publication.
- Preserve rows excluded only by display filters in the plot-data CSV and record
  those filters with clearly named boolean flags, including `included_in_plot`.
  If a display filter changes a derived metric, retain both the complete-data
  and plot-specific values.
- Include concise explanatory text in each plot so it can be fully understood
  and interpreted without consulting the code. Add only the contextual details
  needed to interpret the figure correctly, such as the metric, transformation,
  thresholds, or non-obvious encodings.
- Preserve user-specified paths and existing external path contracts instead of
  overriding them with these defaults.

### Language-specific conventions

Apply only the subsection relevant to the language and type of code being edited.

##### Anchor Python relative paths to the repository root

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

##### R style

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
