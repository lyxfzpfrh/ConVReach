# How to Read the Results

```
results/
  dataset_name/
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

