# QRAO NAE-3SAT

## NAE-SAT Problem
In this notebook, we attempt to solve the NAE-3SAT problem. Given a particular CNF, for example:

$(x_1 \lor x_2 \lor \neg x_3) \land (x_3 \lor \neg x_1 \lor \neg x_6) \land (\neg x_2 \lor x_4 \lor x_5) \land (\neg x_4 \lor \neg x_5 \lor x_6)$

we want to do two things:
1. Satisfy the CNF (i.e. assign values $\in \{0,1\}$ to each $x_i$) such that the CNF evaluates to True
2. For each clause the NAE (not-all-equal) constraint must hold, i.e. the clause operands cannot all evaluate to `True`.
3. If we don't know if a CNF is NAE-SAT or not, we would like to find out (without necessarily assigning values to the variables)

## Using QRAO
We use Quantum Random Access Optimization to solve the NAE-SAT problem, by encoding the problem into a Hamiltonian using (3,1)-QRAC, and finding the ground state of the Hamiltonian.
This is done by using the [prototype-qrao](https://github.com/qiskit-community/prototype-qrao/tree/main) library.

## Solving the Problem (more details in the notebook)
### Solving Point 3) Finding out if a CNF is NAE-SAT or not
1. Encode the CNF into a particular weighted graph
2. Solve Max-Cut on that graph using QRAO (without rounding)
3. Retrieve the relaxed Hamiltonian's ground-state energy value
4. Compute the cut value using that energy value
5. Compare the result to the theoretical cut value: If `relaxed_cut_value < theoretical_cut_value` then we know that the CNF is not NAE-SAT, otherwise we cannot conclude anything
### Solving Point 1) and 2) Assign variables such that the CNF is satisfied and the NAE constraint holds
1. Execute points 1 to 3 of the previous section
2. Perform rounding on the ground state of the Hamiltonian to retrive the optimal variable assignments $x_i, i\in \[1,2N\]$
3. Calculate the NAE-SAT error and the consistency error based on the CNF and the variable assignments
4. If the error is 0, we have a success and we know that the CNF is NAE-SAT.

## Summary:
- Goal: Compute the relaxed Max-Cut value (to determine if a CNF is NAE-SAT or not), or assign numbers in $\{0,1\}$ to $x$ (to find a solution that satisfies the CNF with the NAE constraint)
- Input: Number of variables (integer), and a CNF (string)
- Output: The relaxed Max-Cut value (`relaxed_maxcut_value = compute_max_cut_qrao(G, rounding_scheme, intermediate_results, should_round=False)`), or the variable assignments for $x$ (`x_values = compute_max_cut_qrao(G, rounding_scheme, intermediate_results, should_round=True)`)

## Example Execution 1 (determining if a CNF is NAE-SAT or not):
1) Suppose we want to determine if $(x_1 \lor x_2 \lor \neg x_3) \land (x_3 \lor \neg x_1 \lor \neg x_6) \land (\neg x_2 \lor x_4 \lor x_5) \land (\neg x_4 \lor \neg x_5 \lor x_6)$ is SAT-NAE or not (it is).
2) Run the cell containing `parsed_cnf, num_variables = parse_cnf()` with the inputs: `6` for the number of variables (as we have $\{x_1, x_2, x_3, x_4, x_5, x_6\}$ as variables for the CNF, and `0 1 n2,2 n0 n5,n1 3 4,n3 n4 5` for the CNF itself.
3) Execute the cells that create/display the graph and compute the theoretical Max-Cut value
4) Now we compute the relaxed Max-Cut value because we want to determine if the CNF is NAE-SAT or not
5) Based on that result, by checking `relaxed_maxcut_value > theoretical_maxcut_value` we can see that the CNF is maybe NAE-SAT

## Example Execution 2 (assigning the values $x$ for a CNF)
1) Suppose we want to assign values $x_i$ such that $(x_1 \lor x_2 \lor \neg x_3) \land (x_3 \lor \neg x_1 \lor \neg x_6) \land (\neg x_2 \lor x_4 \lor x_5) \land (\neg x_4 \lor \neg x_5 \lor x_6)$ is satisfied and NAE-SAT.
2) Run the cell containing `parsed_cnf, num_variables = parse_cnf()` with the inputs 
-`6` for the number of variables (as we have $\{x_1, x_2, x_3, x_4, x_5, x_6\}$ as variables for the CNF
-`0 1 n2,2 n0 n5,n1 3 4,n3 n4 5` for the CNF itself.
3) Execute the cells that create/display the graph and compute the theoretical Max-Cut value
4) Now we compute the actual values for $x$ because we want to assign discrete values, so we need to perform rounding with the cell containing `x_values = compute_max_cut_qrao(G, rounding_scheme, intermediate_results, should_round=True)`
5) Check if this satisfies the CNF with NAE constraint by calculating the error. If the error is 0, then the CNF is satisfied with NAE constraint, otherwise not.
6) If not, then we can retry by running again, and if we get no solution after some runs we can assume that the CNF is not NAE-SAT with some probability $p$.
