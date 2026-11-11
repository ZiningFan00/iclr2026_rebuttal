# iclr2026_rebuttal

## Reviewer nEJ4

### Critical Weaknesses:
1. The contribution is incremental relative to prior work. The stage-wise discriminators and accepting-state Q-value target follow Fan & Zhu (2025), and the automaton-level potential shaping follows Bagatella et al. (2024).

Thank you for highlighting the relationship to prior work. Our intended contribution is a demonstration-free, reachability-motivated reward-learning formulation for LTL control, rather than a claim that stage-wise discriminators, accepting-state value targets, or automaton potential shaping are individually new. We will make this distinction more explicit.
The central learning question is how to obtain informative feedback about progress toward acceptance from the agent’s own experience, without expert demonstrations. We study finite-horizon accepting-set reachability as a conceptual target for this feedback and develop a practical stage-wise reward-learning procedure motivated by it. The theoretical analysis characterizes the idealized reachability classifier under its stated sampling assumptions and identifies a sufficient uniform-recurrence condition connecting bounded-horizon returns to Büchi acceptance. These results explain the motivation and scope of the formulation; they do not establish that every update of the practical off-policy algorithm improves reachability or guarantees recurrence.
Relative to Fan and Zhu (2025), we retain the stage-wise discriminator architecture and accepting-state target, but train the progress rewards without expert demonstrations. We recognize that TiLoIL also uses agent-generated segments; therefore, the distinction is not the mere use of such segments, but the demonstration-free training procedure and its accompanying exploration mechanism.
Relative to Bagatella et al. (2024), we retain the automaton-MRP construction but modify its internal reward and discounting scheme to incorporate intermediate stage progress. Specifically, the potential is computed using stage-dependent rewards for intermediate automaton transitions, rather than only accepting-state rewards. This is a modification of the existing potential framework, not a claim to have introduced automaton-level shaping.
Accordingly, we will state our contribution as the specific combination of self-supervised stage-progress reward learning and stage-aware automaton exploration, motivated by the reachability analysis. We will separately identify inherited components, methodological modifications, and the assumptions of the idealized theory. The empirical importance of these modifications must be established through controlled comparisons and ablations, rather than inferred from the theoretical interpretation alone.

2. The theory does not analyze the algorithm that is implemented. Theorem 1 characterizes a single acceptance discriminator trained on on-policy data with discounted time sampling, and Theorems 2–3 concern the corresponding reachability objective. The implemented method instead uses component-wise discriminators with the staged reward in Eq. (9), whole-segment labels, and class-balanced sampling from replay data.

We agree that the current theoretical analysis does not directly characterize the complete implemented algorithm. The idealized classifier uses on-policy suffix-reachability labels under a specified sampling distribution, whereas the practical stage-wise heads use pooled replay data, whole-segment labels, and balanced sampling.
We will explicitly separate the idealized characterization from the practical algorithm. The implemented heads estimate stage-label probabilities under their training distribution and provide reachability-motivated progress rewards; we do not claim that they are calibrated estimates of current-policy acceptance probability. Likewise, the sufficient condition for recurrence is not a guarantee established by our training procedure.
To investigate this distinction empirically, we evaluated the frozen discriminators against future-progress outcomes from fresh current-policy trajectories on Fetch Avoid and Carlo. The measured calibration errors are non-negligible—for example, mean ECE is approximately 0.249 and 0.293 for Carlo’s two heads. These results support interpreting the practical rewards as progress signals rather than exact reachability probabilities. We will revise the theoretical claims accordingly.

3. The main evaluation metric, finite-rollout eventual-discounted return, is a proxy that cannot certify infinite-horizon satisfaction as the authors acknowledge. In addition, four of the eleven main tasks are non-recurrent. The extended-horizon evaluation in Appendix also covers only four tasks, at extended horizon. As a result, the experiments provide limited evidence on the infinite-horizon acceptance problem that motivates the paper.


To address the limited coverage of Appendix M, we completed an expanded checkpoint evaluation covering seven recurrent task families, with Fetch Avoid evaluated separately for post-goal safety. The accompanying table reports undiscounted accepting-state visits at H, 5H, and 10H, measured along uninterrupted executions without retraining or resets after acceptance. This substantially extends the original evaluation, which covered four tasks at approximately 1.5H–1.8H.

| Environment (\(H\) steps) | Cohort | Seed IDs | \(H\) | \(5H\) | \(10H\) |
|---|---|---|---:|---:|---:|
| PointZone (1,000) | Full | 0–4 | **2.20 ± 2.05** | 12.00 ± 11.02 | 24.00 ± 22.05 |
| | Selected | 1, 4 | **4.00 ± 0.00** | 21.00 ± 0.00 | 42.00 ± 0.00 |
| Carlo (500) | Full | 0–3 | **2.78 ± 1.88** | 11.83 ± 14.59 | 20.67 ± 32.06 |
| | Selected | 0, 3 | **3.93 ± 0.71** | 19.98 ± 18.64 | 37.66 ± 43.64 |
| FlatWorld Cycle (50) | Full | 0–11 | **1.00 ± 1.81** | 5.50 ± 9.95 | 11.25 ± 20.35 |
| | Selected | 6, 8, 10 | **4.00 ± 0.00** | 22.00 ± 0.00 | 45.00 ± 0.00 |
| Cheetah Flip (1,000) | Full | 0–13 | **3.78 ± 7.73** | 5.90 ± 12.08 | 6.59 ± 12.34 |
| | Selected | 0 | **24.66** | 42.98 | 42.98 |
| PointButton (1,000) | Full | 0–7 | **0.00 ± 0.00** | 0.00 ± 0.00 | 0.00 ± 0.00 |
| | Selected | 3 | **0.00** | 0.00 | 0.00 |
| Fetch PnP Button† (500) | Full | 0–3 | 1.97 ± 2.39 | 6.98 ± 12.33 | 13.23 ± 24.82 |
| | Selected | 1, 2 | 3.47 ± 2.79 | 13.47 ± 16.93 | 25.97 ± 34.61 |
| SixteenRooms‡ (1,000) | Full | 7–9 | 38.67 ± 33.49 | 195.33 ± 169.16 | 391.33 ± 338.90 |
| | Selected | 7, 8 | 58.00 ± 0.00 | 293.00 ± 0.00 | 587.00 ± 0.00 |
| Fetch Avoid§ (100) | Full | 0–3 | 88.25 ± 0.96 | 488.25 ± 0.96 | 988.25 ± 0.96 |
| | Selected | 0, 1, 3 | 88.67 ± 0.58 | 488.67 ± 0.58 | 988.67 ± 0.58 |

4. The paper contains several inconsistencies that raise questions about the reliability of the reported experimental details. For example, Appendix states that continuous-control runs take about 3 hours, while Table 1 reports substantially longer training times for many tasks, sometimes exceeding 50 hours.

5. The main text is so dense that the paper’s central claims are hard to identify. Moving some secondary theoretical explanations to the appendix and using the available space to provide more details about the experimental setup would improve readability and make the empirical results easier to assess.


### Questions For Authors:
1. How were the baseline methods tuned? Were all methods evaluated using the same episode horizons, environment-step budgets?
   
2. How does the fixed accepting-state Q-value boundary allow the policy to distinguish between safe and unsafe actions immediately after an accepting visit, given that the critic assigns the same target value to all actions at accepting states?
   
3. Can the authors provide additional evaluations that distinguish successful accepting rounds from subsequent safety violations, particularly over longer execution horizons?

We thank the reviewer for the careful assessment and for recognizing the breadth of our evaluation across navigation, robotic manipulation, and locomotion, including recurrent LTL tasks. We also appreciate the positive assessment of the detailed theoretical appendix and its explicit discussion of the guarantees’ limitations. Below, we address the concerns about novelty, the connection between theory and implementation, and the experimental evidence and reporting.

## Reviewer eWUJ
### Critical Weaknesses:
\textbf{1. The analysis does not establish recurrence for the trained method, and its surrogate and critic target can favor failure over it.} 

\textbf{2. The case for a small increment over TiLoIL rests on an evaluation that does not support it.} 

### Questions For Authors:
\begin{enumerate} \item (Clarification.) Which discount and episode lengths produce the eventual-discounted returns in Fig.4 at matched step budgets and episode lengths, together with TiLoIL with demonstrations plus RACE's automaton potential. Only the exploration term differs between these two, so the pair tests the claim that the gains come from exploration. 
\item (Requested experiment.) On the current benchmarks, how does RACE compare with a hand-designed staged reward (stage index, distance to the next region, and a penalty for unsafe regions) under the same SAC setup? 
\item (Requested experiment.) RACE needs one head per non-accepting stage, yet its high-dimensional tasks have at most three stages, the ten-goal scaling study uses a two-dimensional point agent, and the shared-head alternative, described as only slightly worse, stays near zero on Fetch Align (Fig.~6). How does RACE perform on a high-dimensional robot whose specification has eight or more non-accepting stages, or Until and conditional-response obligations? \end{enumerate}

## Reviewer mtfm

### Critical Weaknesses:
The evaluation protocol needs reconciliation. Appendix H specifies 50-step Fetch episodes, while Figure 12 reports horizons of 100 for RACE and 50 for TiLoIL. The evaluation discount or scaling also needs clarification: Figure 4 contains returns above 100, whereas the unit-reward metric in Appendix P.2 would be bounded by 100 if it used the reported training discount of 0.99. These may be reporting differences, but they currently limit interpretation of the comparisons.

Equation 21 also assigns the same Q target to every action at accepting states. In the ideal critic limit, Equation 22 then supplies only an entropy incentive at those states. How safe, recurrent continuation is learned under this boundary rule remains unclear.

### Questions For Authors:
1. Continuation after acceptance. Equation 21 gives every action at an accepting state the same target C. In the ideal critic limit, Equation 22 therefore leaves only an entropy incentive there. How does the actor learn to choose safe actions that begin subsequent rounds? Safety-violation rates and successful continuation rates would help interpret the accepting-round counts in Appendix M.
   
2. Connection between theory and implementation. The conditional optimality result concerns a global optimizer of an idealized objective, while the practical heads use balanced, pooled whole-segment labels (Appendices C.5 and G). Could a calibration check against current-policy future-progress probabilities, or a matched-suffix-label ablation, show how closely the practical rewards reflect the reachability motivation?
   
3. Continuous and discrete actions. How are enabled discrete automaton epsilon-actions selected alongside continuous SAC controls? Do any evaluated tasks require these choices, and what policy architecture or selection rule handles them?
