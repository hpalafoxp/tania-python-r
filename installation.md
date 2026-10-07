# macOS (Apple Silicon) Python + R + Scientific GIS Setup Manual

**Audience:** beginner Python/R learner with a GIS / biogeography background  
**Platform:** Mac with an Apple Silicon processor (M1/M2/M3/M4/M5-family, `arm64`)  
**Last verified:** 2026-10-07  
**Goal:** a clean scientific-computing environment that feels as close as practical to a Windows + Git Bash + VS Code workflow, while keeping Python/R GIS libraries stable and providing a reliable path to scientific notebooks, R Markdown, equations, HTML, and PDF output.

---

## 1. What this setup is designed to give you

At the end of this guide, you will have:

- **VS Code** for Python scripts, general text editing, Markdown, and later Jupyter notebooks.
- **RStudio Desktop** as the main R IDE.
- A **Bash terminal** so that most commands match a Windows Git Bash workflow.
- **Miniforge** as the Python/Conda distribution.
- **conda-forge** as the package source, with strict channel priority.
- A clean **Python 3.14** teaching environment for modern scientific and GIS libraries.
- **R 4.6.1** installed natively from CRAN, independently of Conda.
- A practical R package set for `tidyverse`, plotting, vector GIS, raster GIS, and interactive spatial visualisation.
- **JupyterLab** and notebook export tools when you are ready to introduce notebooks.
- **Quarto + TinyTeX** for scientific publishing, equations, citations, and PDF/HTML rendering.
- A setup appropriate for work with spatial vector data, raster data, environmental grids, NetCDF files, spatial interpolation, mapping, and scientific reports.

This guide intentionally does **not** install Docker, PostgreSQL, Stan, deep-learning frameworks, financial-data libraries, forecasting packages, course autograders, or desktop GIS applications. The focus is Python/R scientific work and the libraries that can be used directly from those languages.

---

## 2. Version choices used in this guide

As of 2026-10-07:

| Component | Recommended version / line | Why |
|---|---|---|
| Python | **3.14.x** | Current stable feature line. The scientific/GIS stack below has current Python 3.14 support. |
| Python build | **Normal CPython / GIL build** | Avoid the experimental free-threaded build for a beginner scientific environment. |
| Miniforge | Current stable Apple-Silicon build | Native `arm64`, small install, conda-forge first. |
| R | **4.6.1** | Current stable CRAN release for Apple Silicon at the time of verification. |
| RStudio Desktop | Current stable release | Stable R IDE and bundled Pandoc support for R Markdown. |
| Quarto CLI | Current stable release | Scientific publishing from R, Python/Jupyter, VS Code, RStudio, and Terminal. |
| TinyTeX | Current TeX Live/TinyTeX release | Lightweight LaTeX distribution suitable for R Markdown, Quarto, and notebook PDF export. |

### Why Python 3.14?

Python **3.14.8** is the current stable release as of this guide. Python 3.15 is still a release candidate and is scheduled for final release on 2026-10-09, so it is too new for this teaching environment.

More importantly, the packages recommended in this guide have been checked specifically for the 3.14 ecosystem. The main compiled dependencies—NumPy, SciPy, pandas, Matplotlib, Shapely, pyproj, pyogrio, Rasterio, netCDF4, and related GIS libraries—now support Python 3.14. OSMnx explicitly added Python 3.14 support in its 2.1 line, rioxarray supports Python 3.12–3.14, and conda-forge provides an Apple-Silicon Python 3.14 build of PyKrige 1.7.3.

For teaching, create the environment with:

```bash
python=3.14
```

rather than pinning a patch release such as `3.14.8`. Conda can then update within the 3.14 line.

### One important pandas/NumPy detail

Current pandas metadata requires **NumPy >= 2.3.3 on Python 3.14**. You normally do not need to manage this yourself: Conda's solver will choose a compatible NumPy when pandas is installed. This is one reason to install the core scientific stack through conda-forge rather than manually mixing binary packages from multiple sources.

---

## 3. Before installing anything: confirm the Mac is using Apple Silicon

Open **Terminal** from `Applications > Utilities > Terminal` and run:

```bash
uname -m
```

Expected result:

```text
arm64
```

Check macOS:

```bash
sw_vers
```

Inspect the current default shell:

```bash
echo $SHELL
```

A normal modern Mac will usually return:

```text
/bin/zsh
```

That is expected. We will switch to Bash later, and the change is reversible.

> **If `uname -m` says `x86_64` on an M-series Mac:** you may be running the terminal under Rosetta. Stop before installing Miniforge or R. Mixing Intel (`x86_64`) and Apple-Silicon (`arm64`) binaries is a common source of scientific-package problems.

---

## 4. Install Apple command-line developer tools

You do **not** need the full Xcode application.

In Terminal:

```bash
xcode-select --install
```

Follow the macOS prompt and let the installation finish.

Verify:

```bash
xcode-select -p
git --version
clang --version
```

The exact versions can differ. What matters is that the commands work.

These tools provide Git, a compiler, and system utilities that R/Python packages may occasionally need.

---

## 5. Use Bash for teaching parity with Windows Git Bash

### 5.1 Why use Bash here?

macOS uses **Zsh** by default, while a typical Windows data-science setup may use **Git Bash**. For basic teaching, the commands are almost identical:

```bash
pwd
ls
cd
mkdir
cp
mv
rm
cat
grep
which
```

### 5.2 macOS Bash caveat

The Bash bundled with macOS is an older Bash 3.2 build. It is not ideal for advanced Bash scripting, but is entirely adequate for:

- navigation;
- file manipulation;
- Python and R;
- Conda;
- Git;
- launching VS Code;
- introductory shell teaching.

There is no need to install Homebrew solely to replace Bash.

### 5.3 Change the default shell to Bash

Run:

```bash
chsh -s /bin/bash
```

Enter the Mac login password when asked.

Quit every Terminal window completely, reopen Terminal, then verify:

```bash
echo $SHELL
bash --version
```

The first command should show:

```text
/bin/bash
```

### 5.4 Return to normal macOS Zsh later

If Conda has already been installed, initialise it for Zsh first:

```bash
conda init zsh
chsh -s /bin/zsh
```

Quit Terminal completely and reopen it.

If Conda has not yet been installed:

```bash
chsh -s /bin/zsh
```

To switch back to Bash later:

```bash
conda init bash
chsh -s /bin/bash
```

Again, restart Terminal afterwards.

---

## 6. Install Visual Studio Code

Download the current macOS build from:

<https://code.visualstudio.com/download>

Choose **Apple Silicon** when an architecture choice is shown.

Move **Visual Studio Code.app** into **Applications**.

### 6.1 Make the `code` command available

Open VS Code and open the Command Palette:

```text
Cmd + Shift + P
```

Search for:

```text
Shell Command: Install 'code' command in PATH
```

Run it, restart Terminal, and test:

```bash
code --version
```

### 6.2 Recommended VS Code extensions

Install:

1. **Python** — Microsoft
2. **Pylance** — Microsoft
3. **Jupyter** — Microsoft
4. **Quarto** — Posit, once you begin scientific reporting
5. **R** — optional, because RStudio will remain the main R editor

### 6.3 Make Bash the VS Code terminal

In VS Code:

1. `Cmd + Shift + P`
2. **Terminal: Select Default Profile**
3. Choose **bash**
4. Open a new integrated terminal

Verify:

```bash
echo $SHELL
```

---

## 7. Optional: make VS Code the terminal editor

Open your Bash profile:

```bash
code ~/.bash_profile
```

Add:

```bash
# Use VS Code when a terminal program asks for a text editor
export EDITOR="code --wait"
export VISUAL="$EDITOR"
```

Save it.

Then open:

```bash
code ~/.bashrc
```

Add this **only if it is not already present**:

```bash
if [ -f ~/.bash_profile ]; then
    source ~/.bash_profile
fi
```

Do not make `.bash_profile` source `.bashrc` as well, or you can create a loop.

---

# Part I — Python, Conda, and scientific GIS

## 8. Install Miniforge

### 8.1 Why Miniforge?

Miniforge is a small Conda distribution built around **conda-forge**. It supports native Apple Silicon and avoids unnecessary mixing with the Anaconda `defaults` channel.

Do **not** install the full Anaconda Distribution for this setup.

### 8.2 Download the Apple-Silicon installer

```bash
curl -fsSLo "$HOME/Downloads/Miniforge3.sh" \
  "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh"
```

Install it:

```bash
bash "$HOME/Downloads/Miniforge3.sh" -b -p "$HOME/miniforge3"
```

Initialise Conda:

```bash
source "$HOME/miniforge3/etc/profile.d/conda.sh"
conda activate base
conda init bash
```

Quit Terminal completely and reopen it.

You should see `(base)` near the prompt.

### 8.3 Verify architecture

```bash
conda --version
python --version
conda info
```

Look for:

```text
osx-arm64
```

If you see `osx-64`, stop before installing packages: that indicates an Intel/Rosetta Conda installation.

---

## 9. Lock Conda to conda-forge

Miniforge already uses conda-forge, but explicitly set strict priority:

```bash
conda config --set channel_priority strict
```

Check:

```bash
conda config --show channels
conda config --show channel_priority
```

If `defaults` appears, remove it:

```bash
conda config --remove channels defaults
```

### Rule for this setup

Use **conda-forge only** for the general scientific/GIS environment.

Avoid mixing `defaults`, `pytorch`, and conda-forge unless a future project specifically requires it.

---

## 10. Keep `base` clean

The `(base)` environment should mainly operate Conda itself.

Do not install the whole scientific stack into `base`.

Create a separate teaching environment instead. If it becomes messy, you can delete and recreate it without damaging Conda.

---

## 11. Create the Python teaching environment

Create an environment with Python 3.14 and `pip`, but no data-science libraries yet:

```bash
conda create -n py-gis python=3.14 pip
```

Activate it:

```bash
conda activate py-gis
```

Verify:

```bash
python --version
which python
```

The executable should live under something like:

```text
~/miniforge3/envs/py-gis/
```

### Why start nearly empty?

It makes the following concepts visible:

- Python vs. a Python package;
- Conda environment vs. the computer itself;
- package installation;
- package removal;
- dependency solving;
- environment reproducibility.

---

## 12. Connect `py-gis` to VS Code

```bash
mkdir -p ~/Documents/python-practice
cd ~/Documents/python-practice
code .
```

In VS Code:

1. `Cmd + Shift + P`
2. **Python: Select Interpreter**
3. Select `py-gis`

Create `hello_python.py`:

```python
name = "Python"
print(f"Hello from {name}!")
```

Run:

```bash
python hello_python.py
```

---

## 13. Python packages to install later as teaching exercises

Always activate the environment first:

```bash
conda activate py-gis
```

### 13.1 Core numerical/data stack

```bash
conda install numpy pandas scipy
```

The Conda solver will select a NumPy version compatible with pandas on Python 3.14.

### 13.2 Plotting and visualisation

```bash
conda install matplotlib seaborn altair plotly
```

Suggested learning order:

1. Matplotlib
2. Seaborn
3. Altair
4. Plotly

### 13.3 Core vector GIS

```bash
conda install geopandas shapely pyproj pyogrio mapclassify
```

Purpose:

- `geopandas` — spatially-aware tabular data
- `shapely` — geometry construction and operations
- `pyproj` — coordinate-reference systems and transformations
- `pyogrio` — fast vector-file I/O through GDAL/OGR
- `mapclassify` — classification methods for choropleth maps

GeoPandas already reads and writes GeoJSON, so a standalone `geojson` library is optional rather than necessary.

If you specifically want to teach the GeoJSON object format/API:

```bash
conda install geojson
```

### 13.4 Interactive/context maps

```bash
conda install folium contextily
```

### 13.5 Raster, environmental, and gridded data

Particularly useful for biogeography, climate, remote-sensing, and environmental datasets:

```bash
conda install rasterio rioxarray xarray netcdf4
```

- `rasterio` — GeoTIFF/raster I/O and raster operations
- `xarray` — labelled N-dimensional arrays
- `rioxarray` — geospatial raster extensions for xarray
- `netcdf4` — NetCDF scientific/environmental data support

### 13.6 Modern columnar/geospatial interchange — optional

For Parquet/GeoParquet workflows:

```bash
conda install pyarrow
```

This is not required for the first GIS lessons, but it is increasingly useful for modern spatial datasets and efficient interchange.

### 13.7 OpenStreetMap and spatial interpolation — later

```bash
conda install osmnx pykrige
```

- `osmnx` — OpenStreetMap networks and geospatial features
- `pykrige` — kriging / spatial interpolation

These are better introduced when a lesson needs them rather than on day one.

---

## 14. Python 3.14 compatibility validation

The recommendation to use Python 3.14 is not based only on the language being current. The stack above was checked against current upstream metadata, release notes, and conda-forge availability.

| Package / layer | Python 3.14 status used for this guide |
|---|---|
| NumPy | Current 2.4/2.5 releases explicitly support Python 3.14. |
| pandas | Explicit Python 3.14 support; requires NumPy >= 2.3.3 under Python 3.14. |
| SciPy | Python 3.14 binaries supported since SciPy 1.16.1; current 1.18 line is newer. |
| Matplotlib | Current project metadata explicitly lists Python 3.14. |
| Seaborn | Pure/high-level layer on supported NumPy/pandas/Matplotlib stack. |
| Altair | High-level/pure-Python visualisation layer; no low-level GIS binary ABI burden. |
| Plotly | Python package supports Conda installation and has no GDAL/GEOS-style native stack to manage. |
| GeoPandas | Current 1.2 line requires Python >= 3.11 and current NumPy/pandas/Shapely/pyproj stack. |
| Shapely | Current metadata explicitly lists Python 3.14. |
| pyproj | Current metadata explicitly lists Python 3.14. |
| pyogrio | 0.12+ publishes Python 3.14 wheels and works through GDAL/OGR. |
| Rasterio | Python 3.14 support added in 1.4.4; current 1.5 line requires Python >= 3.12. |
| rioxarray | 0.20+ explicitly supports Python 3.12–3.14. |
| xarray | Current 2026 release line; compatible with the current scientific stack. |
| netCDF4 | Python 3.14 wheels added in the 1.7.3 line. |
| OSMnx | 2.1.0 explicitly added Python 3.14 support. |
| PyKrige | conda-forge currently provides a `py314` build for **macOS-arm64** (v1.7.3). |
| JupyterLab / nbconvert | Current Jupyter stack works on modern supported Python versions; install it from conda-forge inside the same environment. |

### Practical conclusion

**Keep Python 3.14 for `py-gis`.** There is no current reason to downgrade the main teaching environment to 3.13 for the package set in this manual.

The important safeguards are:

1. use the normal CPython/GIL build;
2. use conda-forge with strict priority;
3. install compiled scientific/GIS packages with Conda rather than mixing binary sources;
4. let Conda solve compatible package versions instead of pinning old course versions;
5. use a separate compatibility environment if a future niche library has not caught up.

Example fallback only if a future package needs it:

```bash
conda create -n py-gis313 python=3.13 pip
```

Do **not** downgrade `base`.

---

## 15. Conda commands worth teaching early

```bash
# See all environments
conda env list

# Activate the teaching environment
conda activate py-gis

# Leave the current environment
conda deactivate

# See installed packages
conda list

# Install a package
conda install PACKAGE_NAME

# Remove a package
conda remove PACKAGE_NAME

# Search for a package
conda search PACKAGE_NAME

# Export only explicitly requested packages
conda env export --from-history > environment.yml

# Re-create an environment
conda env create -f environment.yml

# Delete an environment
conda env remove -n py-gis
```

### Conda vs. pip

For this GIS-oriented environment, prefer:

```bash
conda install package-name
```

This is especially important for packages around:

- GDAL;
- GEOS;
- PROJ;
- NetCDF/HDF;
- compiled NumPy/SciPy extensions.

Use `pip` when a package is unavailable on conda-forge or a project explicitly requires a PyPI release.

Good rule:

1. install as much as possible with Conda first;
2. use `pip` afterwards only when needed;
3. avoid repeatedly replacing the same low-level dependency between Conda and pip.

---

## 16. Add Jupyter when scripts are comfortable

Once notebook concepts are useful:

```bash
conda activate py-gis
conda install jupyterlab ipykernel nbconvert pandoc
```

Register the environment:

```bash
python -m ipykernel install --user \
  --name py-gis \
  --display-name "Python (py-gis)"
```

Launch JupyterLab:

```bash
jupyter lab
```

Or open an `.ipynb` directly in VS Code and choose **Python (py-gis)** as the kernel.

`nbconvert` is included here because it enables notebook export to formats such as HTML and PDF. `pandoc` is included because nbconvert uses it for several document conversions.

---

# Part II — R and RStudio

## 17. Install R natively — not through Conda

Go to:

<https://cran.r-project.org/bin/macosx/>

For an M-series Mac, download the current **arm64** package. At the time this guide was prepared:

```text
R-4.6.1-arm64.pkg
```

Run the installer using its defaults.

Verify:

```bash
R --version
```

The output should identify an Apple-Silicon platform such as `aarch64-apple-darwin` / `arm64`.

> Do not install `r-base` into the `py-gis` Conda environment.

---

## 18. Install RStudio Desktop

Download RStudio Desktop from:

<https://posit.co/download/rstudio-desktop/>

Install it into **Applications**.

Open RStudio and run:

```r
R.version.string
R.version$platform
R.home()
```

RStudio should be using the native Apple-Silicon R installed above.

### Rosetta note

If RStudio itself explicitly requests Rosetta 2, it can be installed with:

```bash
/usr/sbin/softwareupdate --install-rosetta --agree-to-license
```

That does **not** mean you should install Intel versions of R or Miniforge. Keep the scientific runtime native `arm64`.

---

## 19. Install the initial R package set

Unlike the Python environment, it is reasonable to make RStudio useful immediately for tables, plots, spatial work, and scientific writing.

In the **RStudio Console**:

```r
install.packages("pak")
```

Then:

```r
pak::pkg_install(c(
  "tidyverse",
  "janitor",
  "readxl",
  "here",
  "sf",
  "terra",
  "stars",
  "tidyterra",
  "tmap",
  "leaflet",
  "mapview",
  "rnaturalearth",
  "rnaturalearthdata",
  "ggspatial",
  "knitr",
  "rmarkdown",
  "tinytex",
  "bookdown"
))
```

### General data work

- `tidyverse` — `dplyr`, `tidyr`, `readr`, `ggplot2`, `purrr`, `tibble`, etc.
- `janitor` — convenient data-cleaning helpers
- `readxl` — Excel files
- `here` — project-relative paths

### Spatial work

- `sf` — vector spatial data
- `terra` — raster/vector spatial analysis
- `stars` — spatiotemporal arrays and raster-like data
- `tidyterra` — `ggplot2` integration for `terra`
- `tmap` — thematic mapping
- `leaflet` — interactive mapping
- `mapview` — rapid interactive spatial inspection
- `rnaturalearth` / `rnaturalearthdata` — convenient reference geography
- `ggspatial` — scale bars, north arrows, and spatial helpers for `ggplot2`

### Scientific writing

- `knitr` — executable R code in documents
- `rmarkdown` — `.Rmd` rendering
- `tinytex` — R-side TinyTeX management/integration
- `bookdown` — cross-references, equations, figures, tables, citations, and longer scientific documents

You do **not** need to install `ggplot2` separately because it is part of `tidyverse`.

### Quick R test

```r
library(tidyverse)
library(sf)
library(terra)
library(rmarkdown)

sessionInfo()
```

---

## 20. Do we need XQuartz?

Not for the baseline setup in this manual.

Older R/macOS guides often installed XQuartz proactively. Current R and the packages above normally work through native CRAN macOS binaries without XQuartz for everyday use.

Install XQuartz later only if a specific package explicitly requires X11 support.

---

# Part III — Scientific publishing: R Markdown, Jupyter, equations, HTML, and PDF

## 21. Why add Quarto + TinyTeX?

For a scientist, this is worth installing even if the first programming lessons use plain scripts.

The publishing stack gives you:

- `.Rmd` reports from RStudio;
- `.ipynb` notebooks from Python/Jupyter;
- `.qmd` documents later if you use Quarto directly;
- inline equations such as `$E = mc^2$`;
- display equations such as

```text
$$
\hat{\beta} = (X^T X)^{-1}X^T y
$$
```

- bibliographies and citations;
- HTML reports;
- PDF reports through LaTeX;
- reproducible documents that combine prose, code, output, tables, and figures.

### Important distinction

**TeX is needed for LaTeX-based PDF generation. It is not required just to see equations in HTML/notebook previews.**

- Jupyter notebook equation display uses MathJax in the frontend.
- R Markdown HTML uses MathJax by default.
- Quarto can use MathJax/KaTeX for HTML.
- TinyTeX/XeLaTeX becomes important when producing PDF output.

---

## 22. Install the standalone Quarto CLI

RStudio includes Quarto support, but installing the standalone CLI is useful because it makes the same publishing tool available from:

- Terminal;
- VS Code;
- Python/Jupyter projects;
- RStudio.

Download the current macOS installer from:

<https://quarto.org/docs/get-started/>

Use the normal macOS installer.

Restart Terminal, then verify:

```bash
quarto --version
quarto check
```

---

## 23. Install TinyTeX for the whole scientific workflow

Use Quarto to install TinyTeX and place its executables on your `PATH`:

```bash
quarto install tinytex --update-path
```

Restart Terminal afterwards.

Verify:

```bash
xelatex --version
tlmgr --version
quarto check
```

This approach is useful here because the same TinyTeX installation can then be used by:

- Quarto;
- R Markdown / the R `tinytex` package;
- Jupyter `nbconvert`.

In RStudio, verify that R can see it:

```r
library(tinytex)
tinytex::is_tinytex()
```

Expected:

```r
[1] TRUE
```

### Why TinyTeX instead of full MacTeX?

MacTeX is excellent, but is much larger. TinyTeX is a lighter TeX Live distribution and is particularly convenient for R/Quarto because missing LaTeX packages can be installed automatically.

Quarto's PDF engine can automatically install missing TeX packages when using TinyTeX. The R `tinytex` package can do the same when rendering R Markdown.

---

## 24. Add the LaTeX packages commonly required by Jupyter `nbconvert`

Jupyter's LaTeX PDF exporter uses **XeTeX/XeLaTeX**. Unlike R Markdown and Quarto, nbconvert itself does not provide the same automatic missing-package installation workflow.

TinyTeX already contains many common packages and `tlmgr` resolves dependencies automatically. To proactively cover the current nbconvert LaTeX templates, run:

```bash
tlmgr install \
  adjustbox \
  booktabs \
  caption \
  enumitem \
  enumerate \
  eurosym \
  fancyvrb \
  float \
  fontspec \
  geometry \
  grffile \
  hyperref \
  mathrsfs \
  parskip \
  soul \
  tcolorbox \
  titling \
  ucs \
  ulem \
  unicode-math \
  upquote \
  xcolor
```

Do **not** worry if `tlmgr` reports that some are already installed.

The current nbconvert template directly uses packages including `caption`, `float`, `xcolor`, `geometry`, `upquote`, `eurosym`, `fontspec`/`unicode-math`, `fancyvrb`, `grffile`, `adjustbox`, `hyperref`, `titling`, `booktabs`, `enumitem`, `ulem`, `soul`, `mathrsfs`, `tcolorbox`, and `parskip`.

### If nbconvert later reports a missing `.sty` file

Search for the TeX package that owns it:

```bash
tlmgr search --global --file "/missing-file.sty"
```

Then install the package it reports:

```bash
tlmgr install PACKAGE_NAME
```

This is preferable to installing a multi-gigabyte full TeX distribution just in case.

---

## 25. Test R Markdown HTML + PDF output

In RStudio, create `scientific_test.Rmd`:

````markdown
---
title: "Scientific Publishing Test"
output:
  html_document: default
  pdf_document:
    latex_engine: xelatex
---

# Equation

Inline: $E = mc^2$.

Display equation:

$$
\hat{\beta} = (X^T X)^{-1}X^T y
$$

# R output

```{r}
library(ggplot2)

ggplot(mtcars, aes(wt, mpg)) +
  geom_point() +
  theme_minimal()
```
````

Use the **Knit** menu in RStudio to render HTML and PDF, or run:

```r
rmarkdown::render("scientific_test.Rmd", output_format = "html_document")
rmarkdown::render("scientific_test.Rmd", output_format = "pdf_document")
```

If both work, the R Markdown + equation + TeX stack is ready.

---

## 26. Test Jupyter HTML + PDF output

Make sure the Python environment contains the notebook/export tools:

```bash
conda activate py-gis
conda install jupyterlab ipykernel nbconvert pandoc
```

Create a notebook such as `scientific_test.ipynb` and add a **Markdown** cell containing:

```markdown
# Equation test

Inline: $E = mc^2$.

$$
\int_a^b f(x)\,dx
$$
```

The equation should render directly in JupyterLab or VS Code without requiring LaTeX.

Export to HTML:

```bash
jupyter nbconvert --to html scientific_test.ipynb
```

Export to PDF through XeLaTeX:

```bash
jupyter nbconvert --to pdf scientific_test.ipynb
```

The PDF route uses the TinyTeX installation configured above.

### Optional WebPDF route

If a notebook has complex HTML/JavaScript output that does not translate well through LaTeX, nbconvert also has a browser-based WebPDF exporter. This requires Playwright/Chromium and is best installed only if you actually need it.

For ordinary scientific notebooks with equations, tables, Matplotlib/Seaborn figures, and static outputs, the LaTeX PDF route is a good first approach.

---

## 27. Quarto later: one publishing system for both languages

Quarto is worth learning after `.R`, `.py`, `.Rmd`, and `.ipynb` basics are comfortable.

It can render documents that execute:

- R through `knitr`;
- Python through Jupyter;
- notebooks directly;
- equations, citations, figures, and tables;
- HTML and PDF from similar source documents.

Check the installation at any time with:

```bash
quarto check
```

TinyTeX status can be inspected with:

```bash
quarto list tools
```

---

# Part IV — Cross-platform teaching notes

## 28. Commands that are effectively the same in Windows Git Bash and macOS Bash

```bash
pwd
ls
ls -la
cd
cd ..
cd ~
mkdir
mkdir -p
cp
mv
rm
rm -r
cat
head
tail
grep
find
which
clear
history
```

Conda commands are also essentially the same:

```bash
conda env list
conda activate py-gis
conda deactivate
conda install pandas
conda list
```

Git commands are the same once Git is configured.

---

## 29. Mac/Windows differences you are most likely to notice

| Task | Windows Git Bash | macOS Bash |
|---|---|---|
| Home directory | `/c/Users/name` | `/Users/name` or `~` |
| Show current path | `pwd` | `pwd` |
| Path separator in Bash | `/` | `/` |
| Open current folder graphically | `explorer .` | `open .` |
| Open file with default app | often Explorer / `start` | `open file` |
| GUI shortcut modifier | `Ctrl` often | `Cmd` often |
| Copy command output to clipboard | e.g. `clip.exe` | `pbcopy` |
| Read clipboard in pipeline | varies | `pbpaste` |
| Default OS shell | PowerShell/CMD; Git Bash added | Zsh; Bash included |

The biggest conceptual difference is usually **path location**, not Bash syntax.

---

# Part V — Validation and troubleshooting

## 30. Final installation check

Open a fresh Terminal and run:

```bash
uname -m
echo $SHELL
xcode-select -p
git --version
code --version
conda --version
conda config --show channels
conda config --show channel_priority
conda env list
quarto --version
xelatex --version
tlmgr --version
```

Then:

```bash
conda activate py-gis
python --version
which python
python -c "import platform, sys; print(platform.machine()); print(sys.executable)"
```

You want:

- `arm64` architecture;
- `/bin/bash` as the teaching shell;
- conda-forge with strict priority;
- Python 3.14.x;
- the active Python executable inside `~/miniforge3/envs/py-gis/`;
- Quarto working;
- XeLaTeX/TinyTeX working.

Check R:

```bash
R --version
```

Then in RStudio:

```r
R.version.string
R.version$platform
tinytex::is_tinytex()
```

---

## 31. Python scientific/GIS smoke test

After the relevant package lessons:

```bash
python - <<'PY'
import numpy
import pandas
import scipy
import matplotlib
import seaborn
import altair
import plotly
import geopandas
import shapely
import pyproj
import pyogrio
import rasterio
import rioxarray
import xarray
import netCDF4

print("Scientific + GIS Python imports: OK")
PY
```

If OSMnx and PyKrige have also been installed:

```bash
python - <<'PY'
import osmnx
import pykrige
print("Additional spatial packages: OK")
PY
```

---

## 32. R smoke test

In RStudio:

```r
library(tidyverse)
library(sf)
library(terra)
library(tmap)
library(leaflet)
library(rmarkdown)

cat("R + spatial + publishing packages: OK\n")
```

---

## 33. Common problems

### `conda: command not found`

```bash
source "$HOME/miniforge3/etc/profile.d/conda.sh"
conda init bash
```

Restart Terminal.

---

### `code: command not found`

In VS Code:

```text
Cmd + Shift + P
Shell Command: Install 'code' command in PATH
```

Restart Terminal.

---

### VS Code runs the wrong Python

Use:

```text
Cmd + Shift + P
Python: Select Interpreter
```

Choose `py-gis`.

Verify:

```bash
which python
python --version
```

---

### Conda solver problems or strange GIS conflicts

Check:

```bash
conda config --show channels
conda config --show channel_priority
```

The environment should use conda-forge with strict priority.

If an environment becomes badly tangled:

```bash
conda deactivate
conda env remove -n py-gis
conda create -n py-gis python=3.14 pip
```

Then reinstall only what is actually needed.

---

### A future package does not support Python 3.14

Do not downgrade `base` or randomly replace the main interpreter.

Create a compatibility environment:

```bash
conda create -n py-gis313 python=3.13 pip
conda activate py-gis313
```

---

### RStudio starts but uses Intel R

In RStudio:

```r
R.version$platform
```

If it identifies an Intel/x86_64 R installation on an M-series Mac, reinstall the current **arm64** R build from CRAN.

---

### `jupyter nbconvert --to pdf` reports a missing `.sty` file

Find the package:

```bash
tlmgr search --global --file "/missing-file.sty"
```

Install the package reported by the search:

```bash
tlmgr install PACKAGE_NAME
```

Then retry the export.

---

### R Markdown PDF reports a missing LaTeX package

The R `tinytex` package usually installs it automatically. If not, inspect the log:

```r
tinytex::parse_install("document.log")
```

or reinstall/update TinyTeX if the TeX Live year is obsolete.

---

## 34. Recommended learning progression

### Phase 1 — Terminal + scripts

- `pwd`, `ls`, `cd`, paths
- `conda activate` / `deactivate`
- Python variables, types, lists, dictionaries, control flow, functions
- plain `.py` scripts in VS Code
- plain `.R` scripts in RStudio

### Phase 2 — Tables and plotting

Python:

- NumPy
- pandas
- Matplotlib / Seaborn
- Altair / Plotly

R:

- `dplyr`
- `tidyr`
- `ggplot2`

### Phase 3 — Spatial data

Python:

- GeoPandas
- Shapely
- CRS concepts with pyproj
- GeoJSON / GeoPackage
- Rasterio / xarray / rioxarray
- NetCDF
- OSMnx / PyKrige when relevant

R:

- `sf`
- `terra`
- `stars`
- `tidyterra`
- `tmap` / `leaflet`

### Phase 4 — Notebooks and reproducible scientific reports

- JupyterLab / `.ipynb`
- Markdown equations
- notebook HTML export
- notebook PDF export
- R Markdown / `.Rmd`
- citations, figure/table numbering, cross-references
- Quarto / `.qmd`

This sequence keeps programming fundamentals separate from reporting complexity while still leaving a high-quality scientific publishing stack ready when it becomes useful.

---

# Appendix A — Future reference Conda environment

Do **not** use this on day one if the purpose is to practise installing packages manually. It is a reproducibility reference for later.

```yaml
name: py-gis
channels:
  - conda-forge
dependencies:
  - python=3.14
  - pip
  - numpy
  - pandas
  - scipy
  - matplotlib
  - seaborn
  - altair
  - plotly
  - geopandas
  - shapely
  - pyproj
  - pyogrio
  - mapclassify
  - folium
  - contextily
  - rasterio
  - rioxarray
  - xarray
  - netcdf4
  - osmnx
  - pykrige
  - jupyterlab
  - ipykernel
  - nbconvert
  - pandoc
```

Optional additions:

```yaml
  - geojson
  - pyarrow
```

Create it later with:

```bash
conda env create -f environment.yml
```

---

# Appendix B — Why several packages from the original spatial-course environment were removed

The older course environment contained useful GIS packages but also unrelated or course-specific tooling.

**Kept or suggested later:**

- `pandas`
- `matplotlib`
- `seaborn`
- `plotly`
- `geopandas`
- `osmnx`
- `pykrige`
- JupyterLab / IPython kernel

**Removed from the default recommendation:**

- PyTorch / `torch-summary`
- LightGBM
- Prophet
- sktime
- PyOD
- yfinance
- cryptocompare
- otter-grader
- keplergl
- old course-specific version pins

Those packages are not inherently problematic; they simply do not belong in a beginner's general scientific/GIS environment unless a project requires them.

---

# Appendix C — Sources used for the 2026 compatibility check

## Python and scientific stack

- Python releases: <https://www.python.org/downloads/>
- Python 3.14.8: <https://www.python.org/downloads/release/python-3148/>
- Miniforge: <https://github.com/conda-forge/miniforge>
- conda-forge: <https://conda-forge.org/docs/user/>
- NumPy: <https://numpy.org/>
- pandas: <https://pandas.pydata.org/>
- SciPy: <https://scipy.org/>
- Matplotlib: <https://matplotlib.org/>
- GeoPandas: <https://geopandas.org/>
- Shapely: <https://shapely.readthedocs.io/>
- pyproj: <https://pyproj4.github.io/pyproj/>
- pyogrio: <https://pyogrio.readthedocs.io/>
- Rasterio: <https://rasterio.readthedocs.io/>
- rioxarray: <https://corteva.github.io/rioxarray/>
- xarray: <https://docs.xarray.dev/>
- OSMnx: <https://osmnx.readthedocs.io/>
- PyKrige: <https://github.com/GeoStat-Framework/PyKrige>
- PyKrige conda-forge builds: <https://anaconda.org/conda-forge/pykrige>
- netCDF4: <https://github.com/Unidata/netcdf4-python>

## R and scientific publishing

- R for macOS: <https://cran.r-project.org/bin/macosx/>
- RStudio Desktop: <https://posit.co/download/rstudio-desktop/>
- R Markdown: <https://rmarkdown.rstudio.com/>
- TinyTeX: <https://yihui.org/tinytex/>
- Quarto: <https://quarto.org/>
- Quarto PDF/TinyTeX guidance: <https://quarto.org/docs/output-formats/pdf-engine>
- Jupyter nbconvert: <https://nbconvert.readthedocs.io/>

---
