# matplotlib-skill

A Codex skill for creating publication-ready Matplotlib figures with project-level plotting configuration.

This skill helps agents generate consistent academic plots by separating:

- reusable plotting mechanics in `matplotlib_config.py`
- paper/project settings in `plot_config.toml`
- per-figure plotting choices in normal Matplotlib code

## What It Provides

The skill includes:

```text
matplotlib/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── assets/
    ├── matplotlib_config.py
    └── plot_config.toml
```

The bundled config encourages paper-friendly defaults:

- figure sizes derived from LaTeX `\columnwidth` and `\textwidth`
- font sizes configured globally
- line width and marker defaults configured globally
- color palettes configured in TOML
- constrained layout configured globally
- no `tight_layout`
- no `bbox_inches="tight"` for paper figures

## Installation

Clone this repository:

```bash
git clone https://github.com/zhuyk6/matplotlib-skill.git
```

Install globally for Codex:

```bash
mkdir -p ~/.codex/skills
cp -r matplotlib-skill/matplotlib ~/.codex/skills/matplotlib
```

Or install into a specific project:

```bash
mkdir -p .agents/skills
cp -r matplotlib-skill/matplotlib .agents/skills/matplotlib
```

## Usage

Invoke the skill in Codex:

```text
Use $matplotlib to create a publication-ready line plot.
```

When used in a project, the agent should copy the bundled files if they do not already exist. For `src` structure:

```text
assets/matplotlib_config.py -> src/devtools/matplotlib_config.py
assets/plot_config.toml -> plot_config.toml
```

Then edit `plot_config.toml` for the target paper or project.

## Example Plotting Code

```python
from matplotlib import pyplot as plt

from devtools.matplotlib_config import configure_matplotlib, get_latex_figsize

config = configure_matplotlib("plot_config.toml")

figsize = get_latex_figsize(config, width="column", fraction=0.95)
fig, ax = plt.subplots(figsize=figsize)

ax.plot(x, y, marker="o", label="Method A")
ax.plot(x, z, marker="s", label="Method B")
ax.set_xlabel("Epoch")
ax.set_ylabel("Accuracy")
ax.legend()

fig.savefig("figures/line.pdf")
```

## Configuration

Edit `plot_config.toml` for each paper or project:

```toml
[latex]
column_width_pt = 234.8775
text_width_pt = 487.8225
caption_font_size_pt = 9
body_font_size_pt = 10

[figure]
default_width = "column"
default_fraction = 0.95
default_height_ratio = 0.618
default_dpi = 300
default_format = "pdf"

[style]
palette = "nature"
line_width = 1.5
marker_size = 4
marker_edge_width = 1.0
errorbar_capsize = 3
```

Color palettes are also defined in `plot_config.toml`, so users can customize them without editing Python code.
