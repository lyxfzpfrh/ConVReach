# ConVReach
This code repository is used to present the sample artifacts of the OOPSLA submitted paper "Determining the Unreachable: Constraint-Guided Reachability Analysis for Dependency Vulnerabilities".

# How to use

Use -i to specify the input folder, and use -o to specify the output folder.

Using -C allows for the analysis of vulnerabilities in the `Vulnerabilities.txt` file within the target project and outputs constraints in the target project folder.

Using -E enables compilation based on the constraints in the target project and tests the target project until an accessible path is output or it is determined to be unreachable.

Using -A will perform both of the above operations sequentially. 

Using `ConVReach -A -i ./datasets/Fplayer -o ./results` will execute all the processes for the instance project and output the reachable paths.

