# matplotlib-skill

A Codex skill for creating publication-ready Matplotlib figures with scoped configuration and no persistent `rcParams` changes.

The design separates:

- static paper styling in `paper.mplstyle`
- paper dimensions and named palette definitions in `plot_config.toml`
- validated loading, figure-size calculation, and local context management in `matplotlib_config.py`
- per-figure size, palette selection, labels, and layout in plotting code

## Included files

```text
matplotlib/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── assets/
    ├── matplotlib_config.py
    ├── paper.mplstyle
    └── plot_config.toml
```

## Installation

Clone and install globally for Codex:

```bash
git clone https://github.com/zhuyk6/matplotlib-skill.git
mkdir -p ~/.codex/skills
cp -r matplotlib-skill/matplotlib ~/.codex/skills/matplotlib
```

Or install it into one project:

```bash
mkdir -p .agents/skills
cp -r matplotlib-skill/matplotlib .agents/skills/matplotlib
```

Invoke it with:

```text
Use $matplotlib to create a publication-ready line plot.
```

## Project setup

Copy the helper and style next to the plotting scripts, and place the TOML file at the project root. For example:

```text
project/
├── plot_config.toml
└── plots/
    ├── matplotlib_config.py
    ├── paper.mplstyle
    └── plot_results.py
```

The helper resolves `paper.mplstyle` relative to its own file, so these two files should remain together.

Dependencies are Python 3.11 or newer, Matplotlib, cycler, and Pydantic 2.

## Usage

```python
from pathlib import Path

from matplotlib import pyplot as plt

from matplotlib_config import get_latex_figsize, plot_context


def main() -> None:
    with plot_context("plot_config.toml", palette="nature") as config:
        figsize = get_latex_figsize(
            config,
            width="column",
            fraction=0.95,
            height_ratio=0.618,
        )
        fig, ax = plt.subplots(figsize=figsize)

        ax.plot(x, y, marker="o", label="Method A")
        ax.plot(x, z, marker="s", label="Method B")
        ax.set_xlabel("Epoch")
        ax.set_ylabel("Accuracy")
        ax.legend()

        output = Path("figures/line.pdf")
        output.parent.mkdir(parents=True, exist_ok=True)
        fig.savefig(output)
        plt.close(fig)
```

`plot_context` applies `paper.mplstyle`, optionally replaces `axes.prop_cycle` with a named palette from TOML, and restores the caller's previous Matplotlib settings on exit. Its optional `rc` mapping is an escape hatch for unusual context-wide overrides:

```python
with plot_context(
    "plot_config.toml",
    palette="nature",
    rc={"legend.frameon": False, "savefig.dpi": 600},
) as config:
    ...
```

Prefer normal Artist arguments when only one object differs:

```python
ax.plot(x, y, linewidth=2)
ax.legend(frameon=False)
```

## Static configuration

Edit `paper.mplstyle` for settings that remain stable across figures:

```ini
font.family: serif
font.serif: Noto Serif, DejaVu Serif
font.size: 9
axes.labelsize: 9
xtick.labelsize: 8
ytick.labelsize: 8
legend.fontsize: 8

lines.linewidth: 1.5
lines.markersize: 4
figure.constrained_layout.use: True
savefig.dpi: 300
savefig.format: pdf
text.usetex: False
```

For a LaTeX paper, match the figure font to the document where practical. Enable `text.usetex` and select a supported font in the style. Use `text.latex.preamble` only when the template requires specific font/math packages or custom commands; this makes the plotting environment depend on those TeX packages.

## Project metadata and per-figure choices

Edit `plot_config.toml` with widths from the active paper template and the available palette definitions:

```toml
[latex]
column_width_pt = 234.8775
text_width_pt = 487.8225

[palettes]
nature = ["#E64B35", "#4DBBD5", "#00A087", "#3C5488"]
binary = ["#1f77b4", "#ff7f0e"]
```

Choose the actual figure geometry and palette at the call site:

```python
with plot_context("plot_config.toml", palette="binary") as config:
    figsize = get_latex_figsize(
        config,
        width="text",
        fraction=0.95,
        height_ratio=0.45,
    )
```

Omit `palette` when the static style's default color cycle is sufficient.
