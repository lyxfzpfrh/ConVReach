# ConVReach
This code repository is used to present the sample artifacts of the OOPSLA submitted paper "Determining the Unreachable: Constraint-Guided Reachability Analysis for Dependency Vulnerabilities".

# How to use

## 📦 1. Environment and Dependencies

### (1) Prerequisites

- **Python ≥ 3.10**

### (2) Install local Python dependencies

```
pip install -r requirements.txt
```

If you use the built-in toy demo, the Python standard library is enough (no external dependencies).

## ⚙️ 2. Complete Project Framework

```
ConVReach/
├── datasets/                       # Raw or decompiled target projects
│   ├── <ProjectName>/              # Each dataset/project instance
│   │   ├── Vulnerabilities.txt     # Vulnerability and dependency specification
│   │   ├── source/                 # Decompiled source functions
│   │   └── metadata.json           # Optional metadata (target version, description)
│   └── README.md                   # Dataset documentation
│
├── results/                        # Output folder for all analyses
│   ├── <ProjectName>/              # Project-specific result folder
│   │   ├── constraints.json        # Extracted dependency constraints
│   │   ├── reachability.log        # Execution summary and stats
│   │   ├── reachable_paths.txt     # Reachable paths discovered
│   │   └── pruned_paths.txt        # Unreachable paths (pruned)
│   └── summary.csv                 # Aggregated reachability summary across datasets
│
├── convreach/                      # Core implementation (static + dynamic)
│   ├── icfg.py                     # ICFG data structure and loader (JSON)
│   ├── constraints.py              
│   ├── decomposition.py            
│   ├── satisfiability.py           
│   ├── aggregation.py              
│   ├── mutator.py                  
│   └── runner.py                   
│
├── AFLplusplus/                    # AFL++ integration layer
│   ├── custom_mutators/            # Custom AFL++ mutators for ConVReach
│   │   ├── Makefile                
│   │   ├── custom_mutator.c    	
│   │   ├── custom_parser.c    		
│   │   └── README.md           	
│   └── ...
│
├── config.ini                      # System configuration file (paths, AFL, modes)
├── convreach_main.py               
├── requirements.txt                
└── README.md                       # User guide (this file)
```



## 🚀 3. Running Commands

### Base Usage

You can execute ConVReach via the command line with the following parameters:

| Option | Description                                                  |
| ------ | ------------------------------------------------------------ |
| `-i`   | Specify the input dataset/project folder.                    |
| `-o`   | Specify the output results folder.                           |
| `-C`   | Perform **constraint analysis** (static phase). Reads a `Vulnerabilities.txt` file and extracts dependency constraints into the project folder. |
| `-E`   | Perform **execution and dynamic analysis** using AFL++ based on the extracted constraints. Stops when a reachable path is found or all paths are proven unreachable. |
| `-A`   | Perform **both** of the above operations sequentially.       |

### Example Commands

#### ① Constraint Extraction Only

```
convreach -C -i ./datasets/Fplayer -o ./results
```

This command analyzes the dependency graph and vulnerability specifications in `Fplayer/Vulnerabilities.txt`, generating constraint files in `./results/Fplayer/`.

#### ② Dynamic Execution Only

```
convreach -E -i ./datasets/Fplayer -o ./results
```

This will invoke AFL++ using the constraints found in the input project and start fuzzing/testing to determine reachability.

#### ③ Full Pipeline

```
convreach -A -i ./datasets/Fplayer -o ./results
```

This runs **static analysis → dynamic reachability testing** in sequence for the project `Fplayer`, and writes all logs and reachable paths to `./results/`.

You can capture all frequently used options in a `config.ini` 

```
[paths]
input = ./datasets/Fplayer
output = ./results
icfg = ./examples/toy_icfg.json

[modes]
# choose one of {C,E,A}; if omitted, CLI flags decide
mode = A

[afl]
afl_root = ./AFLplusplus
custom_mutator = %(afl_root)s/custom_mutators/libcustom_mutator.so
# number of parallel fuzzers and time budget (seconds)
instances = 4
budget_sec = 86400

[static]
# optional: vulnerability spec filename inside input project
vuln_file = Vulnerabilities.txt

[dynamic]
# unreachability thresholds (paper defaults)
U = 0.01
G = 0.95
relaxation-strategy = 1
Ur = 0.05
Gr = 0.90

[logging]
level = INFO
```



## 📊 4. Reading Results

After execution completes, check the output folder structure:

```
results/
  Fplayer/
    constraints.json     # Extracted constraints
    reachability.log     # Execution summary
    reachable_path.txt  # Reachable path detected
    reachable_path_input # The input triggers the vulnerability
    pruned_paths.txt     # Unreachable (pruned) paths
```

### Log interpretation

| File                   | Meaning                                                      |
| ---------------------- | ------------------------------------------------------------ |
| `constraints.json`     | List of extracted dependency constraints and normalized expressions. |
| `reachable_path.txt`   | A line corresponds to the reachable path; format: `Path_ID -> Constraints` |
| `reachable_path_input` | Inputs that can trigger vulnerabilities, the format depends on the project. |
| `pruned_paths.txt`     | Paths determined to be unsatisfiable after runtime estimation. |
| `reachability.log`     | Full pipeline logs: decomposition steps, satisfiability tracking, AFL++ execution stats. |

Example log excerpt:

```
[INFO] Constraint decomposition complete: 12 paths generated.
[INFO] Starting dynamic phase with 4 parallel instances.
[REACH] Path_1 satisfied all constraints → reachable.
[PRUNE] Path_2 flagged unreachable (U=0.01, G=0.9)
[PRUNE] Unsat constraint number: 3.
```

## 🧠 5. Typical Workflow Summary

Example end-to-end run:

```
convreach -A -i ./datasets/Fplayer -o ./results
```

Results will be under `./results/Fplayer/reachable_paths.txt`
