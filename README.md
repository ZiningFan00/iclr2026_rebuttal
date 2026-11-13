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

We thank the reviewer for identifying this inconsistency. The three-hour statement in Appendix P.4 is not an accurate summary of the task-specific runtimes in Table 1, and we will remove it. We will reconcile the table against available run logs and clarify the training budgets, hardware, and timing scope. Since hardware loads varied, these measurements represent observed computational costs, not a controlled efficiency comparison. We apologize for the inconsistent reporting.

5. The main text is so dense that the paper’s central claims are hard to identify. Moving some secondary theoretical explanations to the appendix and using the available space to provide more details about the experimental setup would improve readability and make the empirical results easier to assess.

We thank the reviewer for this constructive suggestion. We will reorganize the main text around the reachability-based formulation, the practical stage-wise algorithm, and the empirical findings, explicitly distinguishing theoretical guarantees from implementation assumptions. We will move secondary derivations to the appendix and use the freed space to clarify training budgets, episode horizons, evaluation metrics, and baseline tuning, making the comparisons easier to assess and reproduce.


### Questions For Authors:
1. How were the baseline methods tuned? Were all methods evaluated using the same episode horizons, environment-step budgets?

Response to R1 Q1
Thank you for requesting these details. We agree that the implementation, hyperparameters, and comparison budgets should be stated explicitly. The baselines share a common SAC backbone in our continuous-control implementation, but not every historical run used identical horizons or training settings. We clarify the original setup and the additional controlled comparisons below.
Baseline implementation and tuning
We implemented the continuous-control baselines as follows:
- LCER: SAC with counterfactual replay. Physical transitions are reused with relabeled automaton states, and the corresponding automaton successors and rewards are recomputed.
- DRL²: SAC with an automaton-level transition model and a learned potential-based exploration bonus.
- CycLER: a SAC-based adaptation using accepting-cycle reward relabeling. Candidate cycle rewards are computed along each trajectory, and the highest-scoring cycle supplies the replay rewards.
- TiLoIL: its task-specific discriminator and demonstration-based training procedure. The additional TiLoIL_w_potential variant retains the same demonstrations, discriminator, and policy-learning setup, adding RACE’s automaton potential.
For LCER, DRL², and CycLER, the shared network architecture uses two 256-unit hidden layers, twin critics, and automaton-augmented observations. Common settings include learning rate \(3\times10^{-4}\), batch size 64, replay capacity \(10^6\), training discount parameter 0.99, and automatic entropy tuning.

Additional controlled comparison
To address the comparison concern, we completed 30 new runs: five tasks × two TiLoIL variants × three seeds, using all seeds 0–2 without performance-based selection.
Within each task, both variants use the same physical environment implementation, specification, horizon, online interaction budget, demonstrations, warm-up, and evaluation procedure. Shared optimizer settings are learning rate \(3\times10^{-4}\), batch size 64, replay capacity \(10^6\), \(\tau=10^{-4}\), and automatic entropy tuning. The potential-augmented variant differs by the added potential term. Demonstrations remain an additional information resource and are not counted as online interactions.
The common evaluator uses:
- Ten evaluation episodes every 5% of the training budget.
- The fixed final checkpoint evaluated on 100 held-out episodes.
- Identical evaluation seed banks and horizon \(H\).
- Task-performance metrics excluding discriminator rewards and potential bonuses.
These controls establish a matched comparison between the two newly trained TiLoIL variants. They do not retroactively establish identical training conditions for every archived RACE curve.

| Environment | Steps | RACE | TiLoIL | TiLoIL_wo_demo | TiLoIL_w_potential |
|---|---:|---:|---:|---:|---:|
| Doggo Avoid | 400K | 365.16 ± 50.03 | 319.66 ± 69.57 | 381.95 ± 45.58 | — |
| Doggo Navigate | 400K | 286.32 ± 77.26 | 298.58 ± 95.71 | 331.52 ± 48.25 | — |
| Carlo | 1M | 2.55 ± 0.62 | 3.88 ± 1.17 | 0 ± 0 | 4.18 ± 0.57 |
| Fetch Avoid | 400K | 91.79 ± 1.77 | 76.31 ± 5.43 | 86.91 ± 0.41 | 61.49 ± 18.48 |
| Fetch Align | 2M | 47.82 ± 35.07 | 33.40 ± 35.51 | 37.91 ± 38.39 | 36.87 ± 35.08 |
| Cheetah Flip | 1M | 14.50 ± 8.62 | 22.74 ± 2.07 | 0 ± 0 | 12.07 ± 9.20 |
| FlatWorld Cycle† | 2M | 3.64 ± 1.49 | 1.33 ± 1.89 | 0± 0 | 0.00 ± 0.00 |
| Point Zone | 2M | 1.51 ± 1.10 | — | 0 ± 0 | — |
| Point Button | 2M | 2.13 ± 2.27 | — | 0 ± 0 | — |

2. How does the fixed accepting-state Q-value boundary allow the policy to distinguish between safe and unsafe actions immediately after an accepting visit, given that the critic assigns the same target value to all actions at accepting states?

Thank you for this question. The fixed accepting-state boundary is a credit-assignment mechanism for completing the current round, not a constraint guaranteeing safe continuation. Under an exact critic satisfying Q(x,a)=C for every action at an accepting state, the reviewer is correct: the critic provides no action-dependent task signal there, leaving only the entropy incentive in the SAC actor objective.

This local boundary does not make the entire critic or policy uniform; decisions at non-accepting states remain state- and action-dependent. Nevertheless, although finite-trained critics need not fit the boundary exactly, their residual differences cannot be assumed to reliably distinguish safe from unsafe actions. We therefore examined both accepting-state Q-values and actual continuation behavior.

### Accepting-state Q-value diagnostics

We classified actions using native one-step hazard checks. For each accepting-state probe, we compared the highest Q-value among tested safe actions with the highest among tested unsafe actions. Continuous-control tests used \(Q=\min(Q_1,Q_2)\) over a \(21\times21\) action grid; tabular tests enumerated all five actions.

| Environment | Seeds | Comparable probes | Mean best-safe Q | Mean best-unsafe Q | Safe–unsafe gap | Stochastic immediate-hazard rate |
|---|---|---:|---:|---:|---:|---:|
| Grid Avoid, size 7 | 0–2 | 11 | ≈0 | ≈0 | ≈0 | ≈40% |
| FlatWorld | 6, 8, 10 | 12 | 100.071 | 100.079 | −0.008 | 18.79% |

The FlatWorld results average four probes per seed, with 10,000 stochastic action samples per probe. 

### Uninterrupted post-acceptance evaluation

We additionally evaluated deterministic executions at \(H\), \(5H\), and \(10H\), separating immediate violations from failures occurring later after acceptance.

| Environment | Violation metric | \(H\) | \(5H\) | \(10H\) |
|---|---|---:|---:|---:|
| Fetch Avoid, \(H=100\) | Immediate post-acceptance | 0% | 0% | 0% |
| | Subsequent post-acceptance episode violation | 0% | 0% | 0% |
| FlatWorld, \(H=50\) | Immediate post-acceptance | 0% | 0% | 0% |
| | Subsequent post-acceptance episode violation | 0% | 0% | 0% |
| Carlo, \(H=500\) | Immediate post-acceptance | 0% | 0% | 0% |
| | Subsequent post-acceptance episode violation | 25% | 75% | 75% |

Rates are averaged across four training seeds for Fetch Avoid and Carlo, and the three acceptance-reaching seeds for FlatWorld. Immediate rates use eligible post-acceptance steps; subsequent rates use episodes reaching acceptance. Fetch Avoid and FlatWorld each had one distinct initial configuration per seed, limiting generalization of their deterministic results.

The supported conclusion is that successful continuation can occur with the fixed boundary, not that the boundary guarantees it. The deterministic results document observed continuation, while the stochastic probes and Carlo’s later failures identify its limitations. We will clarify this distinction, report safety separately from acceptance counts, and explicitly state that the recurrence conditions in our theory are assumptions—not properties established by the fixed accepting-state target.

3. Can the authors provide additional evaluations that distinguish successful accepting rounds from subsequent safety violations, particularly over longer execution horizons?

Yes. We have added evaluations at \(H\), \(5H\), and \(10H\). Policies are frozen throughout evaluation, and executions are not reset when an accepting state is visited. A completed round is a nonoverlapping completion of the task’s monitored goal sequence—not simply another timestep spent in an accepting state. Consequently, these counts differ from accepting visits and discounted returns. Extended-horizon table results are shown below:

| Environment (\(H\) steps) | Selected seeds | Paper: original horizon | Selected: \(H\) | Selected: \(5H\) | Selected: \(10H\) |
|---|---|---:|---:|---:|---:|
| Carlo (500) | 0, 3 | 2.40 ± 1.27 | **3.93 ± 0.71** | 19.98 ± 18.64 | 37.66 ± 43.64 |
| FlatWorld Cycle (50) | 6, 8, 10 | 3.33 ± 1.49 | **2.00 ± 0.00** | 11.00 ± 0.00 | 22.00 ± 0.00 |
| Cheetah Flip (1,000) | 0 | 14.63 ± 9.47 | **24.66**† | 42.98† | 42.98† |
| SixteenRooms (1,000) | 7, 8 | Not in supplied table | 29.00 ± 0.00 | 146.00 ± 0.00 | 293.00 ± 0.00 |
| Fetch Pick-and-Place Button (500) | 1, 2 | Not in supplied table | 2.31 ± 1.39 | 7.31 ± 8.46 | 13.56 ± 17.30 |
| Point Button (1,000) | 3 | Not in supplied table | 0.00† | 0.00† | 0.00† |
| Fetch Avoid (100) | 0, 1, 3 | Not in supplied table | 1.00 ± 0.00 | 1.00 ± 0.00 | 1.00 ± 0.00 |

Post-acceptance violation rates are calculated among episodes reaching acceptance and then averaged across training seeds. Fetch Avoid is nonrecurrent, so its single goal completion is evaluated together with subsequent safety rather than counted as repeated rounds.

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
