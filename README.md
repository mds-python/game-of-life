# Python Project 2: Conway's Game of Life

Create an application that simulates **Conway's Game of Life**, displays successive generations, and lets the user explore how
simple local rules produce complex patterns.

**You are free to use any popular external Python library, for example for graphics, user interaction, or data visualisation.**
Document your dependencies and how to install them. Implement the simulation logic yourself rather than using a ready-made Game of
Life engine.

This is the second programming project. You may use the **entire** [Programming for Modelling and Data Analysis
course](https://ftims.edu.p.lodz.pl/course/view.php?id=1990), including NumPy, data visualisation, special methods, inheritance, and
automated tests. Keep **`main.py`** as the entry point; choose the names of your other modules and classes yourself.

The basic mark remains achievable with a terminal interface. Higher marks require graphics, interaction, and well implemented
extensions. Every passing version must have correct simulation logic, a clear separation of responsibilities, meaningful classes,
and documented verification. **Automated unit tests are optional for every mark, including 5.0; manual checks are sufficient.**

## 1. What are you simulating?

Game of Life is a cellular automaton. Each cell on a rectangular grid is either **alive** or **dead**. After the initial
configuration is chosen, the next state is determined entirely by the current state; there are no players taking turns.

Each cell has eight neighbours: horizontal, vertical, and diagonal. **Do not count the cell itself.** The standard rules, often
written **B3/S23**, are:

| Current cell | Number of live neighbours | Cell in the next generation |
| ------------ | ------------------------- | --------------------------- |
| Alive        | 0 or 1                    | Dead: underpopulation       |
| Alive        | 2 or 3                    | Alive: survival             |
| Alive        | 4–8                       | Dead: overcrowding          |
| Dead         | Exactly 3                 | Alive: birth                |
| Dead         | Any other number          | Dead                        |

**All cells change simultaneously.** Compute every new state from the same old generation. Updating cells in place while still
counting neighbours changes the rules and is incorrect.

The initial configuration is **generation 0**. Advancing once produces generation 1, including when the board does not visibly
change.

Background explanations:

- **Written explanation:** [Wolfram MathWorld — Game of Life](https://mathworld.wolfram.com/GameofLife.html).
- **Video:** [Numberphile — Does John Conway hate his Game of Life?](https://www.youtube.com/watch?v=E8kUJL04ELA), featuring John
  Conway. Use this for context and explanation; it is not a programming tutorial.

## 2. Board size and boundaries

The mathematical game is usually described on an unbounded grid. This project uses a finite rectangular board with a precisely
defined boundary policy.

**Required default: periodic boundaries, also called a toroidal board.** The left and right edges join, and the top and bottom edges
join. Neighbours across a corner wrap in both directions. This is part of the simulation, not merely a visual scrolling effect.

- Support independently chosen width and height; do not assume a square board.
- Require width and height to be integers **at least 3**. This avoids ambiguity from multiple neighbour offsets mapping to the same
  cell on smaller periodic grids.
- Use a consistent coordinate convention. Recommended: `(row, column)`, counted from zero, with row 0 at the top. A NumPy array then
  has shape `(height, width)`.
- A loaded file defines the whole board, including all-dead rows and columns. Do not crop it or place it randomly inside a larger
  board.
- If you offer an additional larger-board placement option, document the placement rule and reject patterns that do not fit. The
  normal loading mode must still reproduce the file exactly.

A fixed outside-dead boundary policy is a possible extension. It must be selectable and labelled; silently mixing fixed and periodic
boundaries is incorrect.

## 3. Required functionality: the basic version

Your application must:

1. **Start with `python main.py`** or `python3 main.py`, depending on the system. Without additional arguments, show a useful
   prompt/menu or launch a documented default configuration. Command-line options are welcome but not compulsory.
2. **Offer two ways to initialise the board:** load a user-selected pattern file, or generate a random board with user-selected
   width and height.
3. For random boards, accept a finite live-cell probability **`p` between 0 and 1 inclusive** and an optional nonnegative integer
   random seed. Each cell is independently alive with probability `p`; do not promise exactly `p × width × height` live cells. The
   same seed and parameters must reproduce the initial board within the documented Python/NumPy environment. A seed is used at
   initialisation, not at every generation.
4. **Display generation 0 and subsequent generations.** ASCII output is sufficient for 3.0: use `#` for alive and `.` for dead, with
   a clear generation heading. The file format uses `X` rather than `#`, as explained below.
5. Let the user choose a nonnegative number **N** of generations to advance, and display generations 0 through N. A setting of N = 0
   must display the initial state without advancing it. Keep this finite-run option available in graphical versions too, for
   reproducible checks.
6. Report the **generation number and current number of live cells**. Provide a clear way to end the program. A terminal version may
   ask again between runs or finish after N steps; an indefinitely running graphical version must have a responsive stop/close
   action.
7. **Apply the standard rules and periodic boundaries correctly** to arbitrary valid boards, including patterns not supplied with
   the assignment. Do not stop solely because the population count has stayed the same: different boards can have the same
   population.
8. **Validate inputs and handle expected errors** with useful messages: invalid files, missing/unreadable files, invalid dimensions,
   probabilities, seeds, and generation counts. Do not show an unhandled traceback for these ordinary user errors. A failed load
   must not replace an existing valid board with a partially read one.

Changing display speed must not change the sequence of simulated states. Drawing a frame, resizing a window, or pausing must not
advance the simulation accidentally.

## 4. Pattern file format

Use the text format already present in the starter repository's `sample_patterns/` directory:

| Character | Meaning in a file |
| --------- | ----------------- |
| `X`       | Live cell         |
| `.`       | Dead cell         |

Each line is one row. For example, a horizontal blinker is:

```text
.....
.....
.XXX.
.....
.....
```

File rules:

- The board must be rectangular and at least 3 rows by 3 columns.
- Accept only uppercase `X`, dots, and line endings. Spaces, tabs, lowercase letters, comment lines, and blank rows are invalid.
- An empty file is invalid. A rectangular file containing only dots is valid.
- Accept both LF and CRLF line endings, with or without a final line ending.
- Remove line endings, but do not silently remove spaces or pad short rows. Report malformed input, ideally with the offending
  row/column.
- Preserve the dimensions and cell positions exactly. Loading must not advance the board.
- Do not modify the input file while simulating. If you implement saving, write `X` and `.` in this same format, with no headings or
  generation labels.

When the user loads a new file or generates a new random board, start a new run at generation 0. The initial state used for reset
must be an independent copy, not another reference to the evolving array.

## 5. Program design and course skills

### Separate model, file operations, and interface

Use at least **two meaningful application classes** and at least **three custom modules in addition to `main.py`**. Test classes do
not count towards the application-class requirement.

One possible organisation is:

| File                         | Responsibility                                                          |
| ---------------------------- | ----------------------------------------------------------------------- |
| `main.py`                    | Select configuration, create objects, and start the application         |
| `model.py`                   | Board state, neighbour counts, generation updates, and model operations |
| `pattern_io.py`              | Reading, validating, and optional writing of pattern files              |
| `interface.py`               | User input, messages, rendering, and interaction                        |
| `tests/test_*.py` (optional) | Automated tests that run without opening a graphics window              |

For example, a board class can own the state and update rules, while an application/display class manages interaction. This layout
and these names are suggestions; **only `main.py` has a mandatory name**.

- The model must not call `input()`, `print()`, open files, draw graphics, or depend on a graphics library. It should expose data
  and operations that can be used without the interface.
- File-operation code should return data or raise meaningful errors. The interface decides how to present errors.
- Keep `main.py` small. Importing a model or file module must not start the application or open a window.
- Store independent state on instances. Two board objects must not share mutable cell arrays or simulation counters accidentally.
- Avoid circular imports, global mutable state, duplicated rule implementations, and a single large class that does everything.
- Document whether operations mutate an object or return a new one. Copies, resets, saved snapshots, and retained history must
  behave as documented.

### Use NumPy meaningfully

Represent the logical board with a **two-dimensional NumPy array**, using Boolean values or an integer alive/dead representation.
Use array operations meaningfully; merely importing NumPy does not satisfy this requirement.

Vectorised neighbour counting and Boolean masks are encouraged. A clear loop-based implementation is acceptable for the basic mark,
provided the board is a NumPy array and you can explain the calculation. An efficient implementation should require work and
additional storage proportional to the number of board cells for each generation.

Do not confuse an array view with an independent copy. Use an integer accumulator for neighbour counts: it must represent values
from 0 to 8. If you later store cell ages, count **live cells**, not the sum of their ages.

### Use object-oriented features where they help

Special methods and inheritance are available tools, not a requirement to add unnecessary complexity. For example, `__str__` can
describe a board, and related renderers can share an interface or a base class. Use these only when their behaviour is natural and
documented. Composition is also appropriate. Inheritance or a large number of classes does not earn marks by itself.

## 6. Suggested simulation algorithm

For each generation:

1. Keep a stable view or snapshot of which cells are currently alive.
2. Count the eight live neighbours of every cell using the selected boundary policy.
3. Compute a separate next-generation array using the rule table: a cell is alive next if it has exactly three neighbours, or if it
   is currently alive and has exactly two.
4. Replace the current logical state only after all next states have been determined.
5. Increment the generation counter, then let the interface display the resulting state.

For periodic boundaries, shifted arrays are one way to align neighbours. [NumPy's `roll`
documentation](https://numpy.org/doc/stable/reference/generated/numpy.roll.html) explains its wraparound behaviour. If you use this
approach, specify the two spatial axes and omit the zero shift; flattening the board or including the centre cell gives incorrect
neighbours.

First verify one generation on small, hand-checkable boards. Then test repeated updates. Animation should call the same tested model
operation; do not implement a second version of the rules inside the drawing code.

## 7. Graphical and interactive versions

### Graphical animation: required above 3.0

Display the actual board as a changing two-dimensional image or grid. Clearly distinguish live and dead cells, show the generation
and population, and animate successive computed states. A still screenshot, a prerecorded animation, or coloured terminal characters
is insufficient.

Provide **pause/resume and a responsive close/quit action**. The finite-run option from Section 3 must stop automatic advancement
after N updates. Show the final board and make its generation unambiguous. If the user then explicitly requests additional manual
steps or another run, document how the limit is changed or restarted.

Matplotlib is suitable, especially [`imshow`](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.imshow.html) and
[`FuncAnimation`](https://matplotlib.org/stable/api/_as_gen/matplotlib.animation.FuncAnimation.html). Other graphics libraries are
allowed. Use a discrete, documented colour mapping so dead cells do not change appearance simply because a board becomes empty or
fully alive.

Keep the event loop responsive. Do not implement pause by blocking the graphics event loop, and do not rebuild an entire window for
every generation. Distinguish a requested simulation rate from rendering overhead; you do not need to guarantee identical wall-clock
performance on every computer.

### Interactive control package: required for 4.0 and above

Add all of the following:

- **Single-step while paused:** one action advances exactly one generation and remains paused.
- **Speed adjustment:** let the user change the requested generations per second; reject zero, negative, or non-finite rates.
- **Reset:** restore an independent copy of the configuration that began the current run, set generation to 0, and pause. Resetting
  a random run must restore its original cells rather than generate new ones.
- **Mouse editing while paused:** toggle or set individual cells reliably. Ignore clicks outside the grid. Convert screen positions
  to row/column correctly, including any margins or scaling you support.

For this project, editing starts a new run: treat the edited board as generation 0 and as the new reset baseline. Clear any obsolete
statistics, detected cycles, or age history. Document the controls. Loading a new pattern or randomising also establishes a new
baseline. An edit must take effect without closing and restarting the application.

## 8. Optional extension packages

For higher marks, complete packages from the list below. Each package counts **once**, irrespective of how many buttons or visual
options it contains. Features must work together with loading, random initialisation, reset, editing, and both finite and continuous
runs where applicable.

### A. Pattern management

Create and clear boards of chosen dimensions, load a new pattern within the application, and save the current configuration in the
specified format. Confirm before overwriting an existing file, handle read/write errors, and verify that save/reload preserves every
cell and the dimensions. These files store a configuration, not a full running session: loading them starts generation 0.

### B. Configurable Life-like rules

Accept rules in **`B.../S...`** notation, with the birth and survival counts selected from 0–8. Reject malformed strings, repeated
counts, and out-of-range values. Allow an empty birth or survival set; `B/S` is a valid rule in which all cells are dead after an
update. Preserve B3/S23 as the default.

Display the active rule, explain at least two additional rule sets, and verify cases that distinguish them from standard Life.
Changing rules pauses the simulation and starts a new run from the current configuration. Rule choice must affect the model, rather
than being a label in the interface. Your manual or automated checks must include birth at zero neighbours if you permit `B0`—this
is meaningful on the finite board.

### C. Boundary comparison

Offer both periodic boundaries and **fixed outside-dead boundaries**. In the latter policy, cells beyond the grid are always dead,
while edge cells inside the grid still evolve normally. Label the active policy, let the user choose it, and demonstrate a pattern
whose evolution differs between the two policies.

Changing policies pauses and starts a new run from the current configuration. Include manual or automated checks for edges and
corners under both policies. Reusing periodic shifts without removing wraparound contributions is not a correct fixed-boundary
implementation.

### D. Cell-age visualisation

Track how long each currently living cell has remained alive: initial live cells have age 1, births have age 1, surviving cells
increase their age by one per update, and dead cells have age 0. Display age with a readable colour scale and legend. You may cap
displayed colours, but document any cap and keep life/death rules independent of it.

Verify births, survival, death, reset, and editing, manually or with automated tests. Starting a new run resets all currently live
cells to age 1. Different age colours do not represent extra living states and must not change neighbour counts.

### E. Population analysis and pattern behaviour

Record and plot the live-cell population against generation and export it to CSV. Include generation 0 and one sample for every
update, including unchanged states. Rendering and pausing must not add samples.

Also identify extinction, unchanged states, and exact repeated board configurations within a documented, bounded history. Report the
detected period and distinguish a fixed point (period 1) from a longer oscillator. Use exact logical-state comparisons, not
population counts alone. If no repeat is found in the retained history, report that limitation; do not claim the pattern is
aperiodic. Reset the analysis when a new run begins.

### F. Interchangeable display implementations

Provide two independently selectable display implementations, at least one graphical, using the **same model API**. For example, a
terminal view and a Matplotlib view, or two graphical libraries. A terminal view must support finite runs; all graphical views must
support the graphical requirements for the claimed mark.

Select the view without editing source code, avoid importing an unused graphical backend when running the terminal view, and show
that both produce identical states for the same initial board and number of generations. Shared interfaces, composition, or suitable
inheritance are all acceptable. Duplicating the simulation in each view does not qualify.

Other ideas—pattern stamping, video export, sparse boards, or performance comparisons—are welcome, but do not replace these packages
without prior agreement with the instructor. Decorative effects alone are not an extension package.

## 9. Verification and sample patterns

**Automated unit tests are optional.** You may verify the program manually, with automated tests, or with a combination of both.
Document the inputs, expected outcomes, and actual results of your checks. Thorough manual verification can earn the full
verification score and does not limit the attainable mark.

If you choose to write unit tests, use `unittest`, as introduced in the lecture, or another documented framework. Automated tests
should run without prompts, sleeping, or opening a window. A suitable command for `unittest` projects is:

```bash
python -m unittest discover -s tests -p "test_*.py" -v
```

Use explicit, independently established expected results. If you write automated tests, do not call the same update code to
calculate both the expected and actual outcome. Cover the following behaviours through manual checks, automated tests, or both:

| Check group                   | Expected evidence                                                                                                     |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Rule table                    | Alive/dead centre cells at each neighbour count (0–8), manually or via parameterised tests                            |
| Empty board and isolated cell | An empty board stays empty; a lone cell dies after one update                                                         |
| Block                         | A 2×2 block with a dead margin is unchanged                                                                           |
| Blinker                       | A horizontal blinker becomes vertical after one update and returns after two; this also helps expose in-place updates |
| Glider                        | The supplied compact glider has the same shape translated one row down and one column right after four updates        |
| Boundaries and dimensions     | The corner example below, a non-square board, and rejected dimensions below 3                                         |
| Files                         | Valid/invalid symbols, row lengths, blank/empty, CRLF, missing files/no final newline; preserve dimensions/positions  |
| Random initialisation         | Dimensions; p = 0/1 extremes; reproducible seed; invalid p rejected                                                   |
| State independence            | Boards stay independent; snapshots survive updates; generation and population are correct                             |

Verify model and file-operation behaviour for any claimed extension, manually or with automated tests. Check graphical controls
manually and record those checks briefly. No particular test count or coverage percentage is required; all listed behaviours must be
verified. Do not check a random board by demanding an exact population near `p × size`, because a correct random sample can vary.

### Supplied files

The three original starter patterns are retained, and six compact patterns make debugging easier:

| File in `sample_patterns/` | Purpose                                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------------------- |
| `glider.txt`               | Original starter pattern on a larger board                                                     |
| `gosper-glider-gun.txt`    | Original starter pattern with moving structures; finite boundaries eventually affect behaviour |
| `pulsar.txt`               | Original starter pattern: a two-generation transient followed by a period-3 cycle              |
| `check_empty.txt`          | An all-dead 4×6 board; it remains empty under B3/S23                                           |
| `check_single.txt`         | One isolated cell; it dies after one update                                                    |
| `check_block.txt`          | A still life, unchanged after every update                                                     |
| `check_blinker.txt`        | A period-2 oscillator                                                                          |
| `check_glider.txt`         | A compact, padded glider for the four-update translation test                                  |
| `check_wrap.txt`           | A 5×5 board with live cells at (0,0), (0,4), and (4,0)                                         |

For `check_wrap.txt`, the periodic next generation has **exactly four live cells: all four corners**. Under fixed outside-dead
boundaries, all three original cells die and the board becomes empty. This is a useful check for both corner wrapping and the
optional boundary package.

All quoted pattern expectations assume B3/S23. Test only the specified number of updates when checking glider translation; a moving
pattern eventually encounters the finite board's boundaries. A finite torus may behave differently from an infinite plane,
particularly after moving structures wrap around and interact.

## 10. Assessment criteria

The final mark is the **highest row for which both the working-feature requirements and the code-quality threshold are met**.
Requirements are cumulative.

| Mark                | Required working features                                                                                | Minimum code-quality score |
| ------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------- |
| 3.0                 | Basic: rules, boundaries, files/random, NumPy, design, finite runs, ASCII, validation, documented checks | 10/20                      |
| **3.5**             | Basic requirements plus graphical animation, pause/resume, and responsive quit                           | 12/20                      |
| **4.0 — good**      | 3.5 plus single-step, adjustable speed, reset, and mouse editing                                         | 14/20                      |
| **4.5**             | 4.0 plus one complete extension package from Section 8                                                   | 16/20                      |
| **5.0 — very good** | 4.0 plus two different, integrated extension packages from Section 8                                     | 18/20                      |

An ASCII-only version is capped at 3.0. Implementing extra options does not compensate for incorrect rules, inadequate verification,
poor design, or unreadable code. Only features that can be demonstrated count. Unit tests are optional at every level; their absence
alone does not reduce the mark.

### Code-quality score: five categories, 0–4 points each

| Category                         | What is assessed                                                                              |
| -------------------------------- | --------------------------------------------------------------------------------------------- |
| Structure/design                 | Classes, separation, small entry point, state ownership, composition/inheritance              |
| Readability and documentation    | Names, manageable methods, consistent formatting, useful docs, run/control instructions       |
| Correctness and robustness       | Simultaneous updates, boundaries, validation, counters/reset, helpful errors                  |
| NumPy and algorithm design       | Array types/shapes, copies/views, neighbour counting, time/memory use                         |
| Verification and reproducibility | Manual/optional tests, independent expected states, reproducible random input, feature checks |

Each category uses the same scale: **0** = absent or fundamentally unsuitable; **1** = major weaknesses; **2** = adequate with
weaknesses; **3** = good with minor issues; **4** = clear, consistent, and well justified. No category may score 0 for a passing
mark. All basic requirements remain mandatory regardless of the total score. A project that does not meet them receives a
non-passing mark (**2.0**).

The assessor may use unfamiliar valid patterns and invalid inputs following this specification. You must be able to explain your
array operations, trace one generation, justify your design, and make a small modification during a short demonstration. More code,
more dependencies, 3D graphics, or advanced syntax do not earn points by themselves.

## 11. Submission and implementation notes

Submit one archive, or use the repository/submission system specified by the instructor. Include:

- `main.py`, all custom modules, and any required local assets.
- The supplied pattern files and at least **two additional valid patterns of your own**, with a brief explanation of what each tests
  or demonstrates.
- A concise record of verification inputs, expected outcomes, and actual results. If you wrote automated tests, include them and the
  exact command used to run them.
- An accurate `requirements.txt` listing third-party dependencies. NumPy is required; include your chosen graphics/test packages if
  applicable. Do not include standard-library module names.
- A project description in `README.md`, with an **Implementation notes** section containing your name/student identifier, Python and
  package versions used, installation and launch instructions, controls, supported sizes/limits, class/module responsibilities,
  algorithm/boundary choices, implemented extension packages, test results, known limitations, and consulted sources.

You may replace this assignment text with your own clear project description, but make your claimed functionality and usage easy to
assess. Describe what your version actually does.

Installation will normally use:

```bash
python -m pip install -r requirements.txt
python main.py
```

If the selected graphics backend needs an additional system component or interactive display, document it. If you include automated
model/I/O tests, they should run without a display. The submitted program must work after unpacking and following your instructions,
without paths specific to your computer or an internet service at runtime. A notebook or exported video alone is not a valid
submission.

Do not include virtual environments, installed packages, caches, generated test reports, or unrelated files. Keep dated working
versions or genuine version-control history so you can discuss your development process; Git is useful but not required. The
instructor announces the deadline and submission location separately.

Suggested order of work: verify the rule table and one-step update first; add file/random initialisation and manual checks or
optional automated tests; build a finite ASCII run; add graphics and controls; then implement and verify extensions. Keep a working
version at each stage.

## 12. Independent work: do not use AI

**This is an individual programming assignment. Do not use generative AI to design, write, complete, debug, refactor, or document
your solution.** This includes chatbots, AI coding assistants, and AI completion features in an editor. Disable these features while
working on the project.

You may use the starter repository, course material, official documentation, the linked explanations, and ordinary non-AI
documentation/search tools. Discussing general concepts is allowed, but you must write your own implementation. Do not copy or
closely adapt a complete Life implementation from websites, videos, repositories, other students, or previous submissions. Cite
consulted sources; citation does not make a copied implementation acceptable. NumPy operations and graphics APIs are allowed; a
ready-made cellular-automaton engine is not.

Submit only code you understand and can explain. Be ready to implement a small change without AI assistance. Ask the instructor for
help when you are stuck. Academic-integrity concerns are handled under the course and university rules.
