# BIOKT Lab Handbook, Chapter 1: Files, Folders and Project Organisation

**Status:** draft · **Last revised:** 2026-10-06

---

## 1. Why this document exists

You have probably already been told most of the rules below: use the standard
folder layout, don't call a file `test.png`, keep trajectories out of git. Being
told a rule doesn't mean you will follow it, especially when it seems like
bureaucracy and you're busy. So this document gives the **reason** for each rule.
If you understand the reason, you will apply it correctly in cases the rule never
anticipated.

All of the rules follow from one principle:

> **Any number or figure that ends up in a paper must be traceable back to the
> exact trajectory it came from, the inputs that produced that trajectory, and
> the code that analysed it, by someone other than you.**

"Someone other than you" matters. That person might be:

- a referee who asks, eighteen months after submission, whether the 300 K
  replicas used the same water model as the 280 K ones;
- the student who continues your project after you defend;
- the PI, writing a renewal while you're on holiday;
- you, two years from now, when you've forgotten everything that isn't written
  down.

If none of these people can answer "where did this figure come from?" in a few
minutes, the result can't be checked, which in practice means it can't be
reused. Simulations cost weeks of GPU time. Losing track of one is expensive.

---

## 2. Project layout

### 2.1 Where projects live

Every project lives at

```
~/Research/Projects/<Category>/<Project>/
```

on **every** machine you use: your desktop or laptop, and any GPU server or HPC
cluster you run on. `<Category>` is a broad research line and
`<Project>` is a short name for one scientific question.

There is no fixed list of categories, but use an existing one whenever it fits:

| Category | Research line |
|---|---|
| `IDPs` | Intrinsically disordered proteins: ensembles, force-field validation |
| `LLPS` | Biomolecular condensates, phase separation and aggregation |
| `Peptides` | Conformations and folding of short peptides |
| `Mechanics` | Protein mechanics and force spectroscopy |
| `MSM` | Markov state models and kinetics |
| `Quantum` | Quantum-chemical calculations |
| `Simulation` | Methods, benchmarks, force fields and tooling |
| `Colabs` | Projects led by, or done mainly for, external collaborators |

Create a new category only for a new research line that will hold several
projects, never for a single project. Fewer, broader categories are easier to
search than many narrow ones.

**Why:** if the relative path is the same everywhere, scripts, notebooks and
`rsync` commands can be moved between machines without editing paths. Nobody
needs to ask "where did you put it on the cluster?"

### 2.2 Inside a project

The layout is the same whatever simulation engine you use (GROMACS, Amber,
OpenMM, ...):

```
<Project>/
├── data/                # Raw simulation output. NOT in git
├── setup/               # Everything needed to (re)start a run. In git
├── scripts/             # How runs are launched: job submission, drivers. In git
├── analysis/
│   ├── notebooks/       # Jupyter notebooks. In git
│   ├── scripts/         # Python analysis scripts. In git
│   └── figures/         # Figures, named by the convention in §4. In git
├── README.md            # What this project is. In git
└── .gitignore           # Keeps data/ and large binaries out of git
```

What goes where, and why:

| Folder | Contents | In git? | Why |
|---|---|---|---|
| `setup/` | Everything needed to *regenerate* a run: starting structures, topologies, force-field files, engine input files (GROMACS `.mdp`, Amber `.in`, OpenMM `.py`/`.xml`, PLUMED `.dat`) | Yes | These files *define* the simulation. They are small, text-based, and the first thing a referee will ask about. |
| `scripts/` | Job submission (SLURM) and driver scripts | Yes | How a job was launched (resources, software version, flags) affects reproducibility and is otherwise forgotten. |
| `data/` | Raw output: trajectories, energies, checkpoints, logs, final structures (`.xtc`, `.edr`, `.cpt`, `.tpr`; `.nc`, `.rst7`; `.dcd`, `.chk` ...) | **No** | Too large for git, and git doesn't handle binaries well. These are *products*: given `setup/` + `scripts/` they can be regenerated. They still need a known home (see §5). |
| `analysis/scripts/` | Reusable Python (MDtraj, numpy, pandas) | Yes | Code that turns trajectories into numbers. If it isn't saved, the number can't be checked. |
| `analysis/notebooks/` | Exploratory and figure-making notebooks | Yes | Same reason. Keep the analysis logic in `analysis/scripts/` and import it, so it isn't buried in cells. |
| `analysis/figures/` | Figures | Yes | A figure is the visible end of the traceability chain. Tracking it in git records *when* it changed and *alongside which code*. |
| `README.md` | Project description (see §7) | Yes | The first file anyone opens. |

**Don't leave loose files in the project root.** Ad-hoc `.xvg` files, `ener.edr`,
`Untitled.ipynb` and PNGs dumped next to `README.md` are the most common way a
project becomes unreadable. If a file doesn't obviously belong in a folder above,
that tells you the folder structure needs a decision. Bring it up at your next
one-to-one instead of leaving it at the top level.

**Reuse before you write.** Before writing a new script, check whether an existing
one in `analysis/scripts/` or `setup/` can be given an extra argument instead.
One well-parameterised script is better than three near-copies (`rg.py`,
`rg_new.py`, `rg_300K_final.py`). Copies drift apart, and then you no longer know
which one produced a given figure.

### 2.3 Creating a project

New projects are created with the group script `new_simulation_project.sh`,
from the public repository
[BioKT/simulation-project-template](https://github.com/BioKT/simulation-project-template)
(its README explains how to install it):

```bash
new_simulation_project.sh <Category>/<Project>
```

It creates the folder structure above with a README to fill in and the group's
standard `.gitignore`, and initialises a git repository with an initial commit.
Add `--github` to also create a private repository `BioKT/<Project>` on GitHub;
without it, the script prints the command to add one later.

**Why use the script instead of `mkdir`:** the script guarantees that every
project starts identical, so anyone in the group can find their way around
anyone else's project. It also gets `.gitignore` right *before* the first
trajectory is written. Once a 4 GB trajectory has been committed, it stays in
the git history permanently.

### 2.4 Inside the folders: your choice, written down

The template fixes only the top-level folders. How you organise *inside*
`setup/` and `data/` depends on the engine and the protocol, and is up to you.
For example, a GROMACS project might use

```
setup/forcefield/   setup/prep/   setup/mdp/
data/em/   data/nvt/   data/npt/   data/md/
```

while an OpenMM project might have a single `setup/` with a structure and a
driver script, and one `data/` subfolder per replica. Either is fine, as long as
two things hold: inputs are tracked and raw outputs are not, and the README says
how the folders are organised (§7).

Number the scripts that must run in sequence (`01_prep.sh`, `02_em.sh`, ...). The
numbers show the order, so someone else can rebuild the system from scratch by
running the scripts one after another.

---

## 3. Naming simulation files

### 3.1 The anti-pattern

Tutorials for every simulation engine teach you to write names like these:

```
GROMACS:  topol.top   em.gro   npt.gro   md.tpr   md.xtc   md.edr
Amber:    system.prmtop   heat.rst7   prod.nc   prod.out
OpenMM:   system.xml   output.dcd   state.chk   log.txt
```

Inside a single folder that you made yourself today, these names are fine.
The problem is that **files don't stay in their folder.** Over the life of a project:

- the `.xtc` is copied from a cluster to a workstation for analysis;
- a notebook loads `md.xtc` from one system and `md.xtc` from another, and they
  differ only by a directory path that is typed by hand;
- someone `scp`s `npt.gro` to a colleague over Slack to "have a look";
- a run is extended, re-run with a different salt concentration, or repeated with
  a new force field, and the new `md.xtc` overwrites or is confused with the old one;
- the project is archived, and the folder structure that gave the names their
  meaning is flattened or lost.

**The file name is the only metadata guaranteed to travel with the file.**
Directory paths get lost, while names usually survive copying. If you find a file
called `md.xtc` in your Downloads folder, there is no way to tell which protein,
force field, temperature or replica it is. A file called
`as4_a99sb_300K_md_rep2.xtc` tells you without opening it.

There is also a quieter failure: loading the **wrong** file without noticing. If
two systems both produce `md.xtc`, one mistyped path in a notebook will give you a
perfectly plausible plot of the wrong system. Nothing crashes, and the error goes
into the paper.

### 3.2 The scheme

Simulation files are named to match the figure convention in §4:

```
<system>[_<variant>]_<stage>[_rep<N>].<ext>
```

| Field | Meaning | Examples |
|---|---|---|
| `<system>` | Short identifier for the molecular system. **The same identifier is used in figure names.** | `as4`, `ash1_wt`, `ubq` |
| `<variant>` | What distinguishes this run from its siblings: force field, temperature, salt, mutant ... Several may be chained in a fixed order. | `a99sb`, `300K`, `a99sb_300K` |
| `<stage>` | Workflow stage; choose the names per project and list them in the README | `em`, `nvt`, `npt`, `heat`, `equil`, `md`, `hremd` |
| `rep<N>` | Replica index, when there is more than one | `rep0`, `rep1` |

Examples:

```
GROMACS:  as4_a99sb.top          as4_a99sb_npt.gro       as4_a99sb_md_rep0.xtc
Amber:    as4_ff14sb.prmtop      as4_ff14sb_heat.rst7    as4_ff14sb_md_rep0.nc
OpenMM:   as4_a99sb.xml          as4_a99sb_md_rep0.dcd   as4_a99sb_md_rep0.chk
```

The extension already says which engine and which kind of file it is; the name
before it says which run.

Rules that go with it:

- **Lower case, underscores, no spaces.** Spaces and mixed case cause problems in
  shell scripts and between operating systems (macOS is case-insensitive, Linux
  clusters are not).
- **Fix the `<system>` identifier at the start of the project and never change
  it.** Write it in the README. Don't use `as4` in one place and `AS4_peptide`
  in another.
- **Fix the order of variant tags** (e.g. force field, then temperature), write
  it in the README and keep to it. `as4_300K_a99sb` and `as4_a99sb_300K` would
  sort and glob differently.
- **Replicas are numbered from 0** (`rep0`, `rep1`, ...). This matches how
  GROMACS multi-simulation and the group's H-REMD setup number them, so file
  names, run directories and analysis loops all agree.
- **Never overwrite; extend or version.** A re-run with changed inputs is a new
  variant or a new version, not a replacement under the same name.

### 3.3 Name files when they are written, not afterwards

Set the full name in the command that writes the file. Don't plan to rename
later: renaming afterwards depends on someone remembering to do it, and the
generic name will already have spread to scripts and notebooks. Every engine
lets you set the name of its output:

```bash
# GROMACS: one base name for all outputs of a stage
gmx mdrun -deffnm data/md/as4_a99sb_md_rep0

# Amber: name each output explicitly
pmemd.cuda -i setup/md.in -p setup/as4_ff14sb.prmtop -c data/equil/as4_ff14sb_equil.rst7 \
           -o data/md/as4_ff14sb_md_rep0.out -x data/md/as4_ff14sb_md_rep0.nc \
           -r data/md/as4_ff14sb_md_rep0.rst7

# OpenMM: the file names you give the reporters
simulation.reporters.append(DCDReporter("data/md/as4_a99sb_md_rep0.dcd", 5000))
```

Tools that write generic names (`topol.top`, `conf.gro`, `leap.log`) should have
their final products renamed as soon as they are final. The topology and
starting structure matter most (`as4_a99sb.top`, `as4_a99sb_start.gro`), because
they are the files most often copied to other projects.

---

## 4. Naming figures

Figures follow this convention:

```
<system>_<observable>[_<variant>]_v<N>.<ext>
```

| Field | Meaning | Examples |
|---|---|---|
| `<system>` | Same identifier as the trajectories (§3) | `as4`, `ash1_wt`, `ubq` |
| `<observable>` | What is plotted | `rg`, `rmsd`, `rmsf`, `fes`, `contact_map` |
| `<variant>` | Optional: force field, temperature, replica ... | `a99sb`, `300K`, `rep0` |
| `_v<N>` | Version number, incremented whenever the *underlying data or analysis* changes | `_v1`, `_v2` |
| `<ext>` | `pdf` for publication-quality vector figures; `png` for quick inspection | |

Examples: `as4_rg_a99sb_v1.pdf`, `ash1_wt_fes_300K_v3.pdf`, `ubq_rmsd_v2.png`.

Figures are saved in `analysis/figures/` and tracked in git.

**Never** save a figure as `figure.png`, `plot.pdf`, `test.png`, `final.png`,
`final2.png` or `Untitled.png`. Figures with these names are untraceable and won't
be accepted for group meetings, drafts or papers.

**Why the system prefix:** figures end up in slides, Overleaf, Slack and email,
far from the project folder. `fes_v2.png` dropped into a talk could be any of
five systems. `ash1_wt_fes_300K_v2.png` can only be one.

**Why version numbers instead of overwriting:** when you show `as4_rg_v3.pdf` in
group meeting and someone says "that looks different from last month", you can
put `v2` next to it and see what changed. Overwriting removes that history. When
you bump `v`, record *why* in your lab notebook (one line is enough: "v3: now
discarding the first 50 ns as equilibration").

**Why pdf vs png:** vector PDFs stay sharp at any size and are what journals
want. PNGs are fine for a quick look, but a PNG in a paper draft usually means
the figure will have to be remade later.

**Keep the code next to the figure.** Make it obvious which script or notebook
produced each figure. The simplest way is for the script that writes
`as4_rg_a99sb_v1.pdf` to be the only one whose code contains that file name, so
`grep` will find it.

**Check your `.gitignore` doesn't hide figures.** A blanket `*.pdf` or `*.png`
ignore rule (common for keeping literature PDFs or scratch plots out of git) also
silently excludes `analysis/figures/`. If you need such a rule, re-include figures
explicitly:

```
*.pdf
!analysis/figures/*.pdf
```

---

## 5. Several machines, one project

Most projects live on more than one computer: you prepare and analyse on your
own desktop or laptop and run on a GPU server or an HPC cluster. Clusters have
their own submission rules; read and follow them, they are outside the scope of
this document.

The same project can therefore exist in several places at once. Five rules:

1. **Same relative layout everywhere.** `~/Research/Projects/<Category>/<Project>/`
   with the same internal structure on every machine. A script that works on one
   machine should work on another after changing at most a module name.

2. **Inputs flow through git; data flows through `rsync`.** Once the project is
   on GitHub (§6), the tracked parts (`setup/`, `scripts/`, `analysis/`,
   `README.md`) are synchronised by pushing and pulling, never by copying files
   by hand. Before that, edit inputs on one machine only and copy them from
   there. Raw data in `data/` is moved with `rsync` (or `scp`), preserving the
   directory structure and file names.

3. **The canonical copy lives on your desktop.** Your own desktop computer
   holds the authoritative copy of the project, raw data included. Copies on
   clusters, GPU servers and laptops are working copies: once a run finishes,
   bring its output back to your desktop, and from then on treat the remote copy
   as disposable. If two machines hold different versions of "the same"
   trajectory, you have a problem that will appear in the paper.

4. **Back everything up, raw data too.** Back up the project, `data/` included,
   to the hard drives the group provides or to a group machine. Write in the
   README (§7) where the canonical copy and its backup are.

5. **Before you leave the group, hand your projects over.** Group computers,
   normally including your desktop, are reassigned or wiped when you leave, and
   a laptop of your own leaves with you. Either way, your projects must not
   depend on any machine you use. In
   the weeks before you leave, sit down with the PI and agree how each project
   is handed over: what goes into a final, complete backup, where it is stored,
   and what still needs writing down so someone else can continue the work. Do
   this while you still remember the details. There is no fixed procedure yet;
   it depends on the project, so talk about it early.

**Why:** clusters purge scratch, disks fail, laptops get stolen, and people leave. Trajectories
that exist only in a cluster scratch directory under a former member's account
are effectively lost. Once they are lost, every figure made from them can no
longer be checked.

---

## 6. Version control: the short version

Each project has its own git repository, created by `new_simulation_project.sh`
(§2.3). Exploratory projects can stay local: trying an idea should be cheap.
**Once a project is heading for a paper, it must be on GitHub**, as a **private**
repository in the BioKT organisation. To move it there, from the project folder:

```bash
gh repo create BioKT/<Project> --private --source . --remote origin --push
```

> Until the work is published, the repository is **private by default**.
> Going public earlier is sometimes the right choice: developing a tool in the
> open, or contributing code to someone else's software. **Always discuss it
> with the PI first.** The same applies to copying project files to a personal
> public repository, a public gist or any other public site.

**Why talk first:** an unpublished project contains unpublished results, and
often data or structures shared by collaborators under their own conditions.
Once something has been public, even for a few minutes, it may have been
cloned, cached or indexed, and making it private again does not undo that.
A short conversation beforehand settles what can be shared and what can't.

The repository contains `setup/`, `scripts/`, `analysis/` and `README.md`. `data/` and the large binary
files of the common engines (trajectories, energies, checkpoints, logs) are
excluded by the template's `.gitignore`. A single `main` branch is fine for
simulation projects.

- **Commit inputs before you run.** The input file and topology that produced a
  trajectory should be in a commit made *before* the job started. Then "which
  inputs made this run?" has a precise answer: that commit. If you change an
  input file and re-run, commit again.
- **Commit analysis together with the figures it produces.** The figure and the
  code that made it then share a commit.
- **Write commit messages for a stranger.** `update`, `fix`, `asdf` are useless.
  `Add 300K replicas of as4 with a99SB-disp; nsteps 250M` is useful.
- **Never commit raw data.** If you do it by accident, ask for help right away.
  Removing large files from git history gets harder with every later commit.
- **Commit little and often**, at least at the end of each working day on which
  something changed.

**Why:** git is the timestamped record of *what the inputs were at the time*.
Without it, the input file sitting in `setup/` today may not be the one that ran
eight months ago, and nobody can tell.

---

## 7. README and lab notebook

### 7.1 The project README

`README.md` is the first file anyone opens. The template's README has a table to
fill in; at minimum it should state:

- **Scientific question**, in two or three sentences.
- **Systems** and their `<system>` identifiers (§3), e.g. "`as4`: Ac-AAAA-NH2 peptide".
- **Force field and water model** for each system (e.g. ff99SBws-STQ with
  TIP4P/2005), including where any custom force-field files came from.
- **Simulation conditions**: temperature, salt concentration, box type,
  ensemble, run lengths, number of replicas, replica numbering (0-based by default).
- **Software versions**: the simulation engine (GROMACS, Amber, OpenMM ...) and
  any plugins (PLUMED), parameterisation tools, and the Python environment for
  analysis (MDtraj, numpy versions). Different machines in the group run
  different builds, so record which one produced which runs.
- **Naming choices**: the stage names and the order of variant tags (§3), and
  how `setup/` and `data/` are organised inside (§2.4).
- **Where the data lives**: machine and absolute path of the canonical copy of
  `data/` (§5), and of any backup or archive.
- **How to reproduce**: which scripts to run, in which order.

Update it whenever any of these changes. A README that was correct in month one
and is wrong in month ten is worse than none, because people will trust it.

### 7.2 The lab notebook

The README describes what the project **is**. The lab notebook records what you
**did** and what you **thought**: figures, mistakes, dead ends, references,
decisions and open questions, **written as the work happens** rather than
reconstructed afterwards.

**What matters is the habit, not the format.** A Markdown file in the project,
a Google Doc, a Word document: use whatever you will actually open every day.
The format is yours to choose. The habit of writing is not optional.

Writing is how you find out whether you understand something. "The radius of
gyration fluctuates around 1.5 nm" is an observation. Writing the next sentence, "which makes
sense because..." or "which can't be right, because...", is where the thinking
happens. Before you compute anything, write down what you expect to see. When
the result arrives, compare. A result that surprises you is either a discovery
or a mistake, and in both cases it deserves a paragraph.

Some things that make a notebook useful, wherever it lives:

- Date every entry. Newest-first keeps the latest state at the top.
- Record *decisions* and *why*: "Discarded rep3: box too small, protein
  interacted with its periodic image at 40 ns." This is exactly the information
  a referee asks for and a successor needs.
- Record mistakes. A note like "the first 300K set used the wrong salt
  concentration and was deleted" saves the next person from repeating the mistake.
- Refer to files by their full names (§3, §4). "Made a new FES plot" is useless;
  "made `as4_fes_300K_v3.pdf` from `as4_a99sb_300K_md_rep{0..3}.xtc`" lets
  someone retrace the step.
- Make sure it stays with the project when you leave.

The notebook is **not a monitoring tool** and nobody grades it. Its purpose is to
help you think by writing, to make progress visible while it happens instead of
only in a final paper, and to keep a record of things you would otherwise forget.
It is also the main reason anyone can pick up your project after you leave.

---

## 8. Bad and good: side by side

| Bad | Good | What the good version tells you |
|---|---|---|
| `md.xtc` | `as4_a99sb_300K_md_rep2.xtc` | System, force field, temperature, stage, replica |
| `npt.gro` | `ash1_wt_a99sb_npt.gro` | Which system was equilibrated, with what |
| `prod.nc`, `output.dcd` | `as4_ff14sb_md_rep0.nc`, `as4_a99sb_md_rep0.dcd` | The same rule, whatever the engine |
| `topol.top` (copied into another project) | `ash1_wt_a99sb.top` | Which system and force field the topology describes |
| `figure.png` | `as4_rg_a99sb_v1.pdf` | System, observable, variant, version, vector format |
| `fes_final_FINAL2.png` | `ash1_wt_fes_300K_v3.pdf` | Ordered, explicit version history |
| `rg.py`, `rg_new.py`, `rg_300K.py` | `analysis/scripts/rg.py --temp 300` | One script, parameterised; no doubt about which version ran |
| `Untitled3.ipynb` in project root | `analysis/notebooks/as4_rg_convergence.ipynb` | Where it belongs and what it does |
| `.xvg` and `.png` files scattered in project root | Everything in `data/`, `analysis/`, or deleted | A project anyone can navigate |
| Trajectories only in cluster scratch | Canonical copy recorded in README, backed up | Data survives purges and departures |
| Commit message: `update` | `Switch NPT barostat to Parrinello-Rahman; rerun npt` | What changed and why |
| README: "MD of a peptide" | README with system IDs, FF, water, versions, data location | A referee's question answered in one minute |

---

## 9. Self-check

Print this page and keep it next to your screen.

### Before you leave a project for the day

- [ ] Every new input file (engine inputs, topologies, scripts) is committed, with a
      message a stranger would understand.
- [ ] No new raw data has been committed. `git status` shows nothing large or binary.
- [ ] Every new file has a name that says what it is, with no `md.xtc`,
      `test.png` or `Untitled.ipynb`.
- [ ] No loose files in the project root.
- [ ] Today's lab-notebook entry is written, including anything that went wrong.
- [ ] If you started a run or moved data, the README says where the canonical copy is.

### Before you show a figure in group meeting

- [ ] The file name follows `<system>_<observable>[_<variant>]_v<N>.<ext>`.
- [ ] It is in `analysis/figures/` and committed together with the code that made it.
- [ ] If it replaces an earlier version, the version number was bumped and the
      reason recorded in the notebook.
- [ ] You can say which trajectories went into it without looking it up.

### Before you leave the group

- [ ] You have discussed with the PI, well before your last day, how each of
      your projects will be handed over.
- [ ] Every project, `data/` included, has a final complete backup in the
      agreed place, and its README says where.
- [ ] Each README and lab notebook is up to date enough for someone else to
      continue the work without you.

### The five-minute test

> **Pick any figure you have. Can you, in under five minutes, identify
> (1) the trajectory files it came from, (2) the input files that produced those
> trajectories, and (3) the script or notebook that made the figure?**

If you can, your project is in good shape. If you can't, fix it now, while you
still remember. The next person to try will be someone who doesn't.

---

*BIOKT Lab Handbook, BIOKT group (UPV/EHU – DIPC). Chapter 1.*
