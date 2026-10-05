# JEPA-WMs independent PHYWM external replication

Overall verdict: **JEPA_FORMAL_NULL**. Planning: **PLANNING_POSITIVE**.

## A. ID / OOD competence diagnosis

**Original600-step checkpoint:** the retrospective DEV-ID clips were trained on. This diagnostic cannot certify unseen trajectories.

| Population | Full MSE | Full / persistence | Moving MSE | Moving / persistence | Static / persistence |
|---|---:|---:|---:|---:|---:|
| train_id | 0.2681 | 0.6398 | 0.9151 | 0.5689 | 0.6805 |
| dev_id | 0.2860 | 0.6275 | 0.9818 | 0.5457 | 0.6822 |
| dev_ood | 0.5910 | 1.5899 | 1.6484 | 1.1979 | 1.9152 |

Retrospective classification: Case A. Moving-action degradation DEV-ID: shuffled +32.72%, zero +34.48%.

The data were split by fixed SHA256 clip-ID ordering within each of12 TRAIN assets:24 TRAIN-ID /8 DEV-ID per asset (288/96), plus unchanged128 DEV-OOD clips from4 held-out identities. Entire trajectories remain disjoint. A fresh TRAIN-ID-only implicit qualification was necessary to correct retrospective exposure. Original data/cameras/actions/native targets and model were preserved.

## B. Competence qualification and continuation

Qualification seed234; final budget 600 steps. Checkpoints: [0, 100, 300, 600]. Gate: pooled DEV-ID full/moving <=0.95×persistence, shuffled and zero each >=1% moving degradation. OOD competence not required.

| Step | TRAIN-ID full ratio | DEV-ID full ratio | DEV-ID moving ratio | DEV-OOD full ratio | DEV-OOD moving ratio |
|---:|---:|---:|---:|---:|---:|
| 0 | 7.0358 | 6.5563 | 2.0820 | 7.6462 | 2.4155 |
| 100 | 0.9807 | 0.9954 | 0.8363 | 1.5404 | 1.1702 |
| 300 | 0.7769 | 0.8424 | 0.7561 | 1.5761 | 1.2248 |
| 600 | 0.6014 | 0.7285 | 0.6601 | 1.6047 | 1.2641 |

Final competence: {'passed': True, 'dev_id_ratios': {'full': 0.7285113102118536, 'moving': 0.6600663282968812, 'static': 0.7650403097988123}, 'action_moving_degradation': {'shuffled': 0.20697748346529243, 'zero': 0.18224373700774343}, 'train_competent': True, 'ood_competent': False, 'case': 'A', 'verdict': 'PASS_ID_COMPETENCE', 'training_exposure': 'TRAIN-ID only; DEV-ID never trained; original600 checkpoint diagnostic was training-exposed'}. See complete per-clip/horizon curves; no checkpoint selection.

![Qualification curves](../experiments/jepawms_formal_v1/qualification_curves.png)

## C. Frozen formal R experiment

Reached: yes. Seeds [701, 702, 703]; 600 steps per arm; batch8. Both arms train on the same288 TRAIN-ID clips; all96 DEV-ID and128 DEV-OOD clips are evaluated.

Parameter names/shapes/counts, optimizer groups, initialization tensors and sampled batch indices match exactly within each seed. Trainable17,641,773; frozen22,056,576. Only R conditioning values differ: learned constant null vs per-clip canonical13D mobility. DINO targets remain frozen; source native shifted visual/proprio L2 plus sequential rollout loss is unchanged. Masks are evaluation sidecars only. No physical-state decoder or loss was introduced.

## D. Formal DEV-ID results

| Seed | Implicit full | Explicit full | Full gain | Implicit moving | Explicit moving | Moving gain |
|---:|---:|---:|---:|---:|---:|---:|
| 701 | 0.3310 | 0.3357 | -1.41% | 1.1566 | 1.1909 | -2.96% |
| 702 | 0.3369 | 0.3431 | -1.82% | 1.1544 | 1.1562 | -0.16% |
| 703 | 0.3433 | 0.3469 | -1.05% | 1.1778 | 1.1888 | -0.94% |

Persistence MSE: full 0.4558, moving 1.7993. Implicit/explicit ratios to persistence by seed: 701 full 0.7262/0.7364, moving 0.6428/0.6619; 702 full 0.7392/0.7526, moving 0.6416/0.6426; 703 full 0.7530/0.7609, moving 0.6546/0.6607.

Gain mean ± sample SD: full -1.43% ± 0.39%, moving -1.35% ± 1.45%, static -1.30% ± 1.23%. Descriptive n=3; no significance claim.

| Horizon | Mean full gain | Mean moving gain |
|---:|---:|---:|
| 1 | -1.40% | -1.82% |
| 2 | -1.35% | -1.20% |
| 3 | -1.51% | -1.16% |

## E. Formal DEV-OOD results

| Seed | Implicit full | Explicit full | Full gain | Implicit moving | Explicit moving | Moving gain |
|---:|---:|---:|---:|---:|---:|---:|
| 701 | 0.6063 | 0.6024 | +0.65% | 1.7322 | 1.7313 | +0.05% |
| 702 | 0.5939 | 0.5887 | +0.87% | 1.7040 | 1.6938 | +0.60% |
| 703 | 0.5994 | 0.6032 | -0.62% | 1.7122 | 1.7462 | -1.99% |

Persistence MSE: full 0.3717, moving 1.3760. Implicit/explicit ratios to persistence by seed: 701 full 1.6310/1.6205, moving 1.2588/1.2582; 702 full 1.5976/1.5837, moving 1.2383/1.2309; 703 full 1.6126/1.6226, moving 1.2443/1.2690.

Gain mean ± sample SD: full +0.30% ± 0.81%, moving -0.45% ± 1.36%, static +0.65% ± 0.56%. Descriptive n=3; no significance claim.

| Horizon | Mean full gain | Mean moving gain |
|---:|---:|---:|
| 1 | -0.85% | -1.11% |
| 2 | +0.63% | +0.01% |
| 3 | +0.64% | -0.46% |

All4 OOD assets retained; complete per-asset/kind errors, gains, motion and visibility are in `formal_per_asset.csv` and `formal_ood_metrics.json`. Stationary/subpixel cases remain included.


| OOD asset | Kind | Implicit moving mean | Explicit moving mean | Full gain mean | Moving gain mean | Median max motion px | Mean visible moving fraction |
|---|---|---:|---:|---:|---:|---:|---:|
| 45213 | revolute | 1.9477 | 1.9549 | +2.33% | -0.38% | 1.3516 | 0.0809 |
| 45235 | revolute | 2.3491 | 2.3722 | -1.98% | -0.99% | 17.6722 | 0.0735 |
| 45622 | prismatic | 1.6447 | 1.6588 | -0.42% | -0.86% | 10.6364 | 0.0840 |
| 46598 | prismatic | 0.9229 | 0.9092 | +2.32% | +1.48% | 0.0001 | 0.0393 |

Revolute mean seed gains: full -0.02%, moving -0.70%. Prismatic: full +0.76%, moving -0.02%. These are exploratory mechanism strata; no assets were excluded.

Moving gains by asset vary across seeds. Only46598 has positive moving gain in all3 seeds (+1.54%,+1.81%,+1.09%); it is also nearly stationary. The other3 assets have negative mean moving gains, with mixed per-seed signs. This does not establish consistent OOD dynamics utility.

## F. ID versus OOD contrast

OOD minus ID mean gains: full +1.73%; moving +0.91%.

The contrast concerns this16-identity replication. Four OOD identities and three training seeds do not establish a universal distribution-shift law. Report every seed, horizon and asset; moving-region and static outcomes remain distinct.

![Formal gains](../experiments/jepawms_formal_v1/formal_horizon_gains.png)

## G. Action sensitivity

Correct/shuffled/zero actions are evaluated at the same final checkpoint in both arms, without retraining. Per-seed/population/horizon full and moving values are preserved in `formal_action_sensitivity.json`.

| Seed / arm / population | Correct moving | Shuffled change | Zero change |
|---|---:|---:|---:|
| 701-implicit-dev_id | 1.1566 | +25.21% | +20.94% |
| 701-explicit-dev_id | 1.1909 | +22.56% | +18.63% |
| 701-implicit-dev_ood | 1.7322 | +6.60% | +1.00% |
| 701-explicit-dev_ood | 1.7313 | +6.84% | +1.48% |
| 702-implicit-dev_id | 1.1544 | +24.28% | +18.96% |
| 702-explicit-dev_id | 1.1562 | +21.95% | +17.15% |
| 702-implicit-dev_ood | 1.7040 | +6.33% | +0.74% |
| 702-explicit-dev_ood | 1.6938 | +7.14% | +2.50% |
| 703-implicit-dev_id | 1.1778 | +24.90% | +18.47% |
| 703-explicit-dev_id | 1.1888 | +21.40% | +15.76% |
| 703-implicit-dev_ood | 1.7122 | +7.18% | +1.30% |
| 703-explicit-dev_ood | 1.7462 | +6.17% | +1.37% |

## H. Factor corruption

No-retraining explicit final-model tests swap complete canonical13D factors using deterministic different-asset donors within each evaluation population. Same-type and cross-type swaps preserve valid factor encodings; cross-type is counterfactual functional-use evidence, not a different simulated target. Null uses the explicit model own unused zero-initialized null parameter; no implicit weights are transplanted. Full outcomes are in `factor_corruption.json`.

| Population / condition | Mean full MSE | Mean moving MSE |
|---|---:|---:|
| dev_id / true | 0.3419 | 1.1786 |
| dev_id / null | 0.3740 | 1.2357 |
| dev_id / same_type | 0.3626 | 1.2037 |
| dev_id / cross_type | 0.4146 | 1.2934 |
| dev_ood / true | 0.5981 | 1.7238 |
| dev_ood / null | 0.6081 | 1.7173 |
| dev_ood / same_type | 0.5975 | 1.7264 |
| dev_ood / cross_type | 0.6452 | 1.7473 |

DEV-ID corruption increases full/moving MSE: null +9.40%/+4.85%, same-type +6.08%/+2.12%, cross-type +21.30%/+9.74% (mean per-seed relative changes). R is functionally used, while the matched true-R arm is worse than implicit on ID: used with negative marginal utility in this setting. OOD null is +1.68% full but -0.38% moving; same-type -0.09%/+0.15%; cross-type +7.88%/+1.37%. Region-dependent corruption effects do not justify a blanket useful/ignored verdict.

The saved factor mapping audit shows 48/96 ID and64/128 OOD same-type swaps have factor-vector L2 <=1e-6; these swaps provide weak interventions for half the clips. Null and cross-type vectors differ throughout. Same-type insensitivity alone cannot establish that R is ignored.

## I. Native planning

Status: PLANNING_POSITIVE.

Official unmodified CEMPlanner and native final-frame squared latent-distance objective; 32 samples,3 iterations,8 elites,3 action intervals. Four prospectively selected DEV-ID goals (two revolute/two prismatic), paired planner seeds811/812, first frozen model seed701. Goal selection uses physical eligibility before outcomes. Each6D wrench is bounded [-1,1], appended with inactive zero, repeated five raw controls per0.4s interval. Both arms use identical history, targets, constraints and initial simulator state.

The callback uses cached frozen RGB-to-DINO encodings, past controls and candidate future controls; no future reference actions or joint states enter the predictor. Future latent placeholders repeat the last context frame. A single sequence is executed open-loop in SAPIEN, rather than receding-horizon MPC. Joint state scores final normalized error and first reached interval; no state-based early stopping. Native planner.device is set to CPU without changing its algorithm. A separate local dependency target resolves the module nevergrad import without changing the prior runtime.

| Arm | Successes / trials | Success rate | Mean final error / joint span | Mean latent goal cost |
|---|---:|---:|---:|---:|
| implicit | 4/8 | 50.0% | 0.0848 | 0.2629 |
| explicit | 5/8 | 62.5% | 0.0777 | 0.2696 |

Success requires final error <=10% joint span. Per-trial error, first reached interval, complete q trace, CEM costs and actions are saved in `planning_metrics.json` and `planning_actions.json`. Paired prefix replay checks verify the observed context state to1e-5. Eight trials per arm reuse four goals, two planner seeds and one model seed; this is diagnostic feasibility evidence, not an independent large planning sample.

The gain is one additional successful trial; both arms already solve the two revolute goals at joint limits. Explicit has higher predicted latent goal cost despite slightly lower physical final error. This pilot does not show that a latent-prediction improvement was accompanied by planning improvement: the primary prediction experiment did not establish such an improvement. First-reached interval is reported separately because later overshoot can turn an intermediate hit into final failure.

## J. Scientific conclusion

Valid native dynamics competence: True; formal verdict: JEPA_FORMAL_NULL.

**Interpretation:** ID native dynamics competence is established; all formal arms also beat ID persistence. All formal arms remain worse than OOD persistence. Explicit R worsens pooled ID full and moving errors in3/3 seeds; OOD full has two small positive seeds and one negative, while moving mean is negative. The frozen overall verdict is NULL because the endpoints are mixed and do not show consistent global utility. This is not proof of zero effect: ID endpoints show consistent degradation.

H1 (prediction benefit) is unsupported by the pooled primary results. H2 has a descriptive relative contrast: OOD-minus-ID is +1.73 percentage points full and +0.91 moving, driven largely by ID harm rather than an established OOD benefit. H3 is unsupported: moving gain is not stronger than static. H4 is unsupported as a general monotonic trend; OOD full improves from first to later horizons, but moving and ID effects do not show consistent increasing benefit. These remain small-sample descriptive findings.

**Paper placement:** report this as a completed independent external intervention test with native latent competence, paired architecture/order controls and a null overall result with ID degradation. It qualifies as an external replication test, but not positive validation of explicit-R benefit. Action destruction and factor corruption establish functional input use; the small planning pilot and joint-kind/per-asset strata remain diagnostic.

The scientific claim concerns native future-latent prediction under a matched null-versus-true mobility intervention. It does not establish exact3D-state accuracy. R is constant within an asset/joint and can also supply identity information; this replication does not isolate structure semantics from an identity cue. Four OOD identities and three seeds limit generalization. The OOD predictor is weak against persistence, which limits representation-boundary interpretation there. No universal distribution-shift law or cross-backbone numeric equivalence is established.

**Next exact experiment (proposal only):** freeze a larger disjoint asset panel and a factor-independent identity-code control before outcomes; compare null, true mobility and dimension-matched identity codes with identical native JEPA architecture, training order and budget. Predeclare ID competence, OOD competence and the paired ID/OOD interaction. This distinguishes factor semantics from identity information without rescuing this completed run.

## Files, provenance and resources

All required receipts are in `experiments/jepawms_formal_v1/`; previous Stage A roots/reports were preserved. Original manifest SHA256: `65de20c9c361cf5723dc9369eec58f62ee86a3b2dbca00242e45b03519d59b47`. Read-only verification passed all3584 original sidecar/frame files. The remote root is `/sciclone/scr10/mliu22/PHYWM-jepawms-formal-v1/`; large checkpoints remain there and are individually hashed.

Qualification execution code is the `8b9c15a` version, SHA256 `15b82f447a5880ab19c07a16ec9f98a32163ba05fd292c33ef68f2370025942c`. Later runner changes add formal-only corruption modes and hardware receipts, leaving native training/forecast definitions unchanged. Formal code hash is frozen in the protocol.

Slurm stdout/stderr, resource benchmark, exact hardware/runtime/torch/CUDA/peak-memory records, commit history and artifact hashes accompany the report. GPU availability was checked before the formal launch; all8 A40s had long existing reservations, so the available CPU nodes were used with identical FP32 architecture.

Exact formal runtime: seed-701-explicit 33.61 min, seed-701-implicit 36.97 min, seed-702-explicit 36.86 min, seed-702-implicit 34.50 min, seed-703-explicit 34.52 min, seed-703-implicit 36.86 min. Total six-arm compute runtime 3.555 hours. Python3.10.21, PyTorch2.7.0+cu126, CUDA build12.6, FP32,16 CPU threads; GPU peak memory is not applicable to CPU execution. Slurm batch MaxRSS is saved per job in `slurm_accounting.txt`.

Jobs:81810 qualification;81811 thread-cost benchmark;81812 match preflight;81813 array0-5 six formal runs;81816 array0-2 corruption;81824 failed planning initialization attempt;81825 successful complete planning rerun (3m14s).

Commits before final receipts:

- `3391dc0 Seal JEPA replication hashes and prior-artifact preservation`
- `0a927e0 Complete native CEM pilot and JEPA formal scientific report`
- `0fac88a Freeze bounded native CEM goals and matched planning pilot`
- `aeff805 Record all six matched JEPA formal runs and factor corruption`
- `f3cf1ff Freeze paired three-seed native JEPA null versus mobility experiment`
- `5e21d44 Establish unseen-trajectory JEPA competence and separate asset OOD weakness`
- `1165590 Complete conditional JEPA formal runner and reproducible reporting pipeline`
- `cb1c388 Record retrospective JEPA ID/OOD diagnosis and native matched experiment implementation`
- `8b9c15a Freeze JEPA trajectory ID/OOD split and uncontaminated qualification protocol`

The final report/receipt commit follows these boundaries; `git log -- experiments/jepawms_formal_v1 docs/jepawms_formal_v1.md` provides the complete history. Checkpoint byte identities are in `remote_checkpoint_hashes.json`; delivered artifact and prior-preservation checks are in `artifact_hashes.json`.
