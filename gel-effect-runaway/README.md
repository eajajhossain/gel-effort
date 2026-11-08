# gel-effect-runaway

Infer how propagation and termination become diffusion controlled in bulk MMA free-radical
polymerization (the gel and glass effects; their laws are stated, their seven constants are
not) from self-heating ampoule conversion and polystyrene-calibrated GPC data. Then predict
the mid-plane exotherms, conversion, molar masses and molar-mass distributions of thick PMMA
sheet castings through cure and post-cure, and design a fast cure program for a 20 mm sheet
that stays clear of thermal runaway and bubble formation.

| | |
|---|---|
| Author | Eajaj Hossain (Brainware University, Kolkata), eajajhossain890@gmail.com |
| Profile | https://github.com/eajajhossain |
| Domain | engineering-sciences / chemical-engineering / polymer-reaction-engineering |
| Agent budget | 18000 s; 2 CPUs, 2 GB RAM, open internet |
| Verifier | separate container, no network, binary reward |

See [instruction.md](instruction.md) for the task as the agent receives it and
[task.toml](task.toml) for the configuration. The verifier's plant model, `tests/plant.py`,
is the data-generating model itself; its slab equations are built by `slab_system()`, which
`simulate_slab()` and the verifier's continuous-time cure run share. The script that sampled
the data and the comparator implementations are not included in this package.

## Revision notes

### Kinetics laws stated, seven constants withheld (this revision)

Platform pass@1 on the previous revision (one attempt per model, terminus-2): both models
reward 0. GPT-6-sol (max) built a free-volume fit with GPC constraints and missed on casting
and distribution errors and on D1 (X_avg 0.9350 against 0.975); Claude Opus 5.5 (low)
introduced an Mw-dependent gel model, acknowledged a rough ampoule fit, and missed C1 `t80_min`
(408.75 against 372.96 min, +9.6 %) and D1 (X_avg 0.9391). The platform's auto review of that
revision passed every check (test failure attribution, error-message actionability, test
accuracy, false-positive risk, requirement coverage, environment adequacy, trajectory
integrity) except "task clarity", which failed on identifiability alone. The finding:
instruction.md said that the dependence of `kp` and `kt` on the state of the mixture was not
given and that the ampoule data "do not determine that dependence outside the conditions they
cover", yet the verifier scored C3's 120 C post-cure and D1 against one unpublished gel/glass
law with one hidden constant set; "a smooth, positive additional diffusion suppression confined
to high-temperature, high-conversion states would leave every ampoule observation unchanged
while changing those scored outcomes", so the stated inputs could not identify the scored
extrapolations and the verifier did not accept the resulting equivalence class. The previous
two revisions had tried to meet this with wording alone (stating the extrapolation as fact,
then warning about it); the review held that such a warning "does not supply the missing
identifying assumption". This revision supplies it.

- **instruction.md, Kinetics.** The gel-effect law for `kt` and the glass-effect law for `kp`
  are stated in closed form, exactly as `tests/plant.py` implements them:
  `kp = kp0/(1 + exp(B_p*(1/V_f - 1/V_fp)))`,
  `kt = kt0/(1 + exp(z_t)) + C_rd*kp*[M]` with
  `z_t = A_t*(1/V_f - 1/V_ft) + m*ln(Mw/1e5) + E_t*(1/T - 1/343.15)`, `Mw` the local cumulative
  weight-average molar mass of the polymer formed so far (ln term zero before any has formed),
  with the units and signs of the seven constants. The constants themselves (`B_p`, `V_fp`,
  `A_t`, `V_ft`, `m`, `E_t`, `C_rd`; `HIDDEN` in `plant.py`) stay withheld and are to be
  inferred from the ampoule data. The sentence "no conversion is given below which `kp` or
  `kt` equals its chemically controlled value" is replaced by the law. The extrapolation note
  (C1's initiator loading, C3's 120 C post-cure, the interior of a thick sheet; the ampoule
  span) stays as a statement of fact; the clause that the ampoule measurements "are consistent
  with one such dependence" is gone, as is every "no law is stated" posture of the previous
  two revisions.
- **instruction.md, Acceptance.** The claim that the tolerances "accommodate any smooth rate
  dependence that jointly reproduces" the ampoule data (which the review rejected, and which
  a single truth cannot honour) is replaced: the reference values are the physical system's,
  which follows the stated kinetics at its values of the seven constants, and the tolerances
  allow for fitting error and numerical error (the wording of the platform copy of the previous
  revision, which passed the instruction-structure check). Every band is unchanged.
- **Identifiability under the stated laws.** With the functional form fixed, the only freedom
  is the seven constants, and the ampoules determine them: every converged seven-constant fit
  from neutral starts reaches the same values (Reference solution), the reference solution's
  worst casting quantity sits at 0.73 of its band, and dropping any stated term worsens the
  ampoule fit 2.6 to 112 times and fails the castings (Calibration of the casting bands).
  `B_p`, the steepness of the glass effect, is the loosely determined constant (fit 4.49
  against 1.0). Profile (the other six constants refitted to the ampoule data at each fixed
  `B_p` with `solve.py`'s residuals, castings graded by the verifier's `score()`; script not
  included): at `B_p` = 15, 8, 2, 1, 0.5 the cost is 43.77, 43.68, 43.78, 44.24, 45.31 and
  every casting passes (C3 `Mn` at 1.072, 1.073, 1.049, 0.993, 0.912 of the truth); at
  0.45, 0.4 and 0.25 (cost 45.55, 45.84, 47.45) C3 `Mn` falls to 0.898, 0.883 and 0.822 of
  the truth and fails its band (at 0.25 the C3 distribution too, L1 0.0504); every other
  quantity stays inside its band at every point. The cost at `B_p` = 0.45 is 1.91 above the
  best fit, at the edge of the 95 % single-parameter profile interval (1.92): at that edge a
  fit statistically consistent with the data misses C3's post-cure `Mn` by about 1 %. The
  glass-effect steepness is constrained only weakly by the 50-70 C ampoules and sets the
  molar mass of the chains C3 starts at 120 C. Publishing `B_p` as separately measured, as
  the cage effect is, would remove that edge; it was considered and not adopted in this
  revision.
- **What stays with the agent.** Fitting seven constants through a self-heating ampoule
  model, the slice-by-slice polystyrene-equivalent GPC convention, the conduction-limited
  slab with shrinkage, the molar-mass distribution accumulated from instantaneous chain-length
  distributions, and the D1 design within about 1 K of the runaway boundary with four program
  points. The Hardening revision had withdrawn these laws because the models passed with
  them under the limits of that time (720 min, 12 points, molar masses within 1.15,
  distributions within 0.06): Claude Opus 5.5 in both pass@1 runs of that revision, GPT-6-sol
  in one (in the other its fit came out a few kelvin too reactive at 60 C and failed the
  castings). The current limits are tighter (670 min, 4 points, 1.10, 0.05). No pass@1 has
  been run on this revision.
- **Unchanged.** The data, `castings.json`, `design.json`, the casting truth
  `tests/truth/expected.json`, the plant model's equations and constants, the verifier's code
  and every acceptance band and limit. `tests/plant.py` changed only in its docstring (the
  laws are now stated in instruction.md; the `max(Mw, 1e3)` guard never binds), so
  `tests/truth/SHA256SUMS.txt` was regenerated for `plant.py` and checked with the
  `sha256sum --check --strict` command of `tests/test.sh`; `expected.json` and `design.json`
  keep their hashes. `solution/solve.py` changed only in its docstring and comments (it
  implements the stated laws, which it used to describe as its own choice); its code is
  AST-identical to the previous revision's. `tests/test_state.py` changed only in the comment
  block above the bands (the "envelope" framing is withdrawn; its initiator loadings now read
  0.001, 0.002, 0.003 as in `ampoules.csv`). task.toml is unchanged.

### Four-point D1 programs, tighter molar-mass and distribution bands, four physics tests (previous revision)

Platform pass@1 on the previous revision (670 min, tabulated cage effect; one attempt per
model, terminus-2): both models reached reward 0, GPT-6-sol (max) with 2 of 4 tests failed
and Claude Opus 5.5 (low) with 1 of 4. Their castings failed the same way, on the physics
the task is built around: C1 (w_AIBN 0.0005, below the ampoule range) `t80_min` +9.6 % (band
6 %), `t_dT_mid_max_min` +14 to +17 % (band 12 %), `X_avg_t1` -0.017 and -0.027 (band 0.010);
C3 `Mn_avg_end` x1.18 and x0.82 (band 1.15); the distributions L1 0.061, 0.061, 0.067, 0.069
(Opus; band 0.06) and 0.22-0.25 (GPT). Opus's fit of its own gel law found no chain-length
dependence (its `n` = 0), GPT's law had none; both therefore misplace the gel onset of C1,
whose low initiator loading gives a different Mw at the onset than any ampoule. GPT's D1
program, a 66 C hot start cooled at 86 min, ran away in the plant at 84 min (162.7 C,
X_avg 0.78). Opus's D1 program passed the plant:
`[[0, 55], [150, 55], [200, 45], [420, 50], [520, 70], [580, 120], [670, 120]]`, hottest
under-converted layer 74.2 C, X_avg 0.9826. Three things followed from those transcripts.

- **D1: `max_points` 12 -> 4** (`environment/data/design.json` and the byte-identical
  `tests/truth/design.json`); the cycle limit (670 min), bath range (40-130 C), ramp limit,
  bubble limit and target are unchanged. Opus's program is a warm start that chills the sheet
  through the gel effect and then post-cures; it needs seven points, and it is not the
  conservative design its 21 K reading suggests. In the plant the family is a knife-edge: of
  nine neighbours (55 C for 120/150/180 min, 45 C until 330/380/420 min, then 70 C and 120 C),
  six run away to 130-165 C; its own program compressed to 600 min runs away at 149 C, and
  ending its chill 40 min earlier gives 137 C. The large reading comes from the bubble
  criterion, which counts only layers below 0.9 conversion: a chill makes the exotherm peak
  after the layers cross 0.9. The other levers were evaluated and rejected: a bath floor at
  47.5 or 48 C still admits the strategy (75 C), and a floor above the 48.33 C hold-runaway
  boundary (52 C) kills every hold-and-ramp design with it (Opus's program at a 52 C floor:
  159 C); cutting the cycle limit below about 645 min fails the reference solution (651 min)
  before it fails the chill family, whose perfect-knowledge minimum is about 524 min; raising
  the target does not bite because the 108.5 C post-cure approaches its equilibrium ceiling
  (`X_eq` 0.9886 at 108.5 C, 0.9805 at 120 C, 0.9756 at 125 C) asymptotically: the reference
  program reaches 0.980 at 652 min, 0.982 at 655 min and 0.985 only at 712 min, while a 120 C
  finish tops out at 0.9839. With four points, every program is a hold, a ramp and a hold in
  some order. An author-side scan of 358 four-point (and fewer) programs outside the
  hold-and-ramp family, graded exactly as the verifier grades them (80 volumes, continuous
  bubble check): warm hold + cool ramp + single reheat (168), single ramps (16), ramp then hold
  (36), hold + two-slope reheat (54), immediate V-dip + reheat + finish hold (84). Single ramps,
  ramp-then-hold and two-slope reheats never pass (hottest under-converted layer 109-167 C or
  X_avg below 0.85). Seven warm-start programs pass, all of the form 65-70 C for 45-150 min,
  cool to 40-45 C by 150 min, one slow reheat (about 0.13 K/min) to 105-112 C, e.g.
  `[[0, 65], [45, 65], [150, 45], [670, 112]]` (76.5 C, X_avg 0.9757) and
  `[[0, 70], [150, 40], [610, 110], [670, 110]]` (89.9 C, 0.9773): they use the whole 670 min
  and end 0.0005-0.0023 above the target, against a late-conversion error of about 0.002 in the
  reference model itself, and each of the seven runs away (111-160 C) when the whole program
  is shifted by 0.4 K on one side (four of them colder, three hotter). The 5-point chill
  family, for comparison, passes 8 of 36
  programs with margins up to 22 K, so `max_points` = 5 would not have closed the loophole.
  What is left is the hold-and-ramp window: in the plant a single hold followed by the fastest
  ramp needs 718 min at 46.5 C, 682 at 47.0, 664 at 47.25, 645 at 47.5, 624 at 47.75 and
  597 at 48.0 C (zero margin), and holds above 48.33 C run away, so the hold must lie within
  about 1.1 K below the runaway boundary; the reference program (hold 47.59 C, ramp at 587 min)
  shifted as a whole by -0.4 K reheats under-converted material to 103 C and by +1.0 K runs away
  in the hold (133 C); it passes at +0.4 and +0.7 K. A model that misjudges the gel
  timing by the 10 % both transcripts show (36-54 min on C1) misplaces that boundary by about
  0.5-0.8 K. Every correct design on record fits: the reference solution (4 points, 92.6 C,
  651.2 min), `marten_ham_pubf` (4 points, 93.0 C, 626.3 min), `marten_ham` (3 points, 93.5 C,
  621.5 min) and `fv_chain` re-expressed with its two-slope ramp collapsed to one
  (`[[0, 47.65], [588.22, 47.65], [651.2, 110.63], [651.67, 110.63]]`, 91.6 C, 651.7 min;
  its recorded 5-point program is no longer admissible as written).
- **Casting bands: molar masses within a factor 1.10 (was 1.15), distribution L1 at most 0.05
  (was 0.06).** `t80_min` (6 %), `t_dT_mid_max_min` (12 %), the temperature excess and the
  conversions are unchanged: the independent model `marten_ham_pubf` sits at 4.1 % and 7.8 %
  on the first two, and both agents already fail C1 by +9.6 % and +14 % or more. Under the new
  bands the correct models stay inside: reference solution worst 0.73 of a band (C3 `Mn`,
  factor 1.072) and 0.34 (distribution 0.017), `fv_chain` 1.046 and 0.019, `marten_ham_pubf`
  1.051 (0.52) and 0.042 (0.84), `marten_ham` 1.068 and 0.035, `coarse_grid` 1.040 and 0.018;
  every wrong-physics row fails as before. Opus's 0.061 distributions now miss by 0.011, and
  the worse `marten_ham_pubf` minimum (1.348, 0.075) still fails.
- **Four physics tests.** `test_predictions_structure` and `test_cure_program_structure`
  passed for both agents on well-formed files alone. The suite is now
  `test_castings_conversion_and_exotherm` (t80, temperature excess and its time, X_avg at t1
  and at the end), `test_castings_molar_masses`, `test_castings_mwd_fractions` and
  `test_cure_program_in_plant`; each first requires the Outputs contract (exact schema, finite
  numbers, normalized distribution; program limits) as a prerequisite, so a malformed file
  fails every test and an invalid program is still never run in the plant. `score()` remains
  the union of the three casting scorers for the author-side graders; `tests/decide.py` lists
  the four names. On the values in the two transcripts (stand-ins for the unavailable
  `predictions.json` files, with the agents' failing quantities substituted into the reference
  predictions and their programs as submitted) each agent fails all four tests.
- **Reference solution.** Its D1 family is now hold, ramp at 1 K/min, post-cure hold (three
  parameters, at most four points; its previous optimum already had that shape), with the same
  margins, searched coarse to fine (Nelder-Mead on 10 volumes from two safe starts in parallel,
  then the earliest safe ramp start on 30 volumes by bisection) with design runs that stop at
  the margined target and abort on a runaway; the D1 search takes about 4 min instead of
  15-30, and the whole script about 8.5 min on 2 CPUs (Reference solution). Its program is
  `[[0, 47.607], [584.72, 47.607], [648.33, 111.22]]`, 92.86 C and X_avg 0.9791 in the plant.
  `tests/truth/SHA256SUMS.txt` was regenerated for `design.json`; `expected.json` and
  `plant.py` are unchanged.
- **instruction.md, extrapolation stated as fact.** The platform's post-evaluation review of
  the previous revision passed every check but "task clarity", which failed on
  identifiability ("distinct smooth rate laws can agree along the ampoule histories yet
  differ in the sheets") and on pragmatic communication (the instruction did not alert the
  solver that the castings extrapolate beyond the ampoules). A first wording that also
  described how the bands were calibrated and what the ampoules discriminate was rejected by
  the instruction-structure check as calibration commentary and a feature hint. The Kinetics
  section now states only facts of the inputs: where the castings and D1 leave the ampoule
  conditions (C1's initiator loading, C3's 120 C post-cure, the interior of a thick sheet),
  what the seven runs span, and that their measurements are consistent with one dependence
  to within the stated uncertainties. No law and no method is stated. The calibration
  evidence stays here: the reference family and `marten_ham_pubf`, whose fits sit at 0.8 and
  1.4 times the stated uncertainties in RMS over the 137 residuals (the latter with the
  earlier comparator engine), both pass every band; the `no_chain` and `no_Et` variants, at
  1.3 and 2.2 times, fail (Calibration of the casting bands), so the residual level alone is
  not the criterion. The `dT_mid_max_K` failure message now includes its computed threshold
  (the review's one actionability remark). The structure check then flagged padding in the
  Sheet D1 section (the interpolation rule repeated after D1 adopts the casting conventions);
  that sentence, the repeated "begins at the bath temperature at t = 0", the repeated point
  format and the repeated `w_AIBN` unit are gone, and the layer-conversion definition is
  reduced to the clause that adds to the general one.
- **Exact bounds.** The verifier's `_outside()` used to admit an excess of 1e-12 (relative
  or absolute) past every bound as a rounding allowance; the correctness review pointed out
  that a cycle time of 670.0000000000005 then passed a limit instruction.md states as "at
  most" with numbers "graded exactly as written". Every comparison is now exact in double
  precision, and instruction.md says so (Acceptance): a value on a bound passes, one past it
  by any amount fails. The reference solution therefore builds its 1 K/min ramp a fraction
  1e-9 longer than |dT|/RAMP (`RAMP_SLACK`, a few milliseconds over the ramp), so its slope
  never depends on the last bit of the arithmetic; before that its program met the exact
  inequality by 4e-14. The programs printed rounded in the cure-program tables below exceed
  1 K/min by up to 0.02 % as printed and would fail the exact check in that form; the
  full-precision programs were graded.

### Cycle limit 670 min and tabulated cage effect (previous revision)

Two changes on top of the hardening below. The data, the casting truth, the plant model's
equations, the verifier's code and every acceptance band are unchanged.

- **`design.json`, D1 cycle limit.** `max_cycle_time_min` 720 -> 670, in
  `environment/data/design.json` and the byte-identical `tests/truth/design.json`. Every correct
  design on record still passes: the reference solution (651.2 min), `fv_chain` with the 1 K bath
  margin (651.7 min), `marten_ham` (621.5 min) and `marten_ham_pubf` (626.3 min). The slack above
  the slowest of them shrinks from about 68 min to about 18 min, i.e. from about 0.9 K to about
  0.25 K of hold-temperature caution beyond the reference solution's 1 K bath margin (each kelvin
  costs about 75 min). In the plant, a single hold followed by the fastest 1 K/min ramp to
  108.5 C, with the ramp started at the earliest instant that keeps the bubble limit (no margin
  at all; `test_state._step_violations` on 80 volumes), needs 718 min at 46.5 C, 682 min at
  47.0 C, 664 min at 47.25 C, 645 min at 47.5 C, 624 min at 47.75 C and 597 min at 48.0 C (the
  ramp target hardly matters: 104 C or 113 C moves these by at most 1 min). The old limit
  excluded holds at or below about 46.5 C; the new one excludes holds at or below about 47.2 C,
  within 1.1 K of the 48.33 C runaway boundary, and a design that keeps the reference solution's
  margins has no room for an extra 0.3 K of caution. A model that places the runaway boundary
  too low (too reactive at 60 C, as the GPT transcript of the previous revision was) now fails
  on time instead of passing with a cold hold.
- **`properties.json` and instruction.md, cage effect.** `initiator_efficiency_cage_effect` no
  longer states the logistic law `f = f0/(1 + exp(B_f*(1/V_f - 1/V_ff)))` with `B_f` and `V_ff`:
  that is the same switch in `1/V_f` that the plant uses for `kp` and `kt`, so stating it handed
  over the structural model. It now gives `f/f0` as a table, `f_factors` at 100 evenly spaced
  `V_f_nodes` from 0.002 to 0.2, tabulated from the plant's law and rounded to 6 decimals
  (`f/f0 < 5e-7`, below `V_f` = 0.018, tabulates as 0). instruction.md names `f0`
  (`efficiency_f_low_conversion`) and says the end values apply outside the tabulated range
  (pure monomer exceeds `V_f` = 0.2 above 69 C, where `f/f0` differs from 1 by less than 1e-5);
  it does not prescribe the interpolation rule. The plant keeps the closed form, so an agent's
  interpolant is a small, known model error: against the exact law, linear interpolation is off
  by at most 0.0095 in `f/f0` (at `V_f` = 0.031, where `f/f0` = 0.137; 7 % relative at
  `f/f0` = 0.12, 1 % at 0.7), PCHIP by 0.0014, a natural cubic spline by 0.00025, and a
  two-parameter logistic fit of logit(`f/f0`) against `1/V_f` over the interior nodes recovers
  `B_f` and `V_ff` to better than 1e-4. The window `V_f` = 0.025-0.045 is the late stage of the
  50-70 C cures and C3's 120 C post-cure (pure polymer `V_f` = 0.028 there). Run in the plant
  itself with linear interpolation of the table in place of the law, every casting still passes:
  the largest shift is C3's Mn, -1.4 % (0.10 of its band); the other molar masses move by at
  most 0.4 %, the conversions by at most 0.0004 and the distributions by at most 0.002 in L1,
  and the reference D1 program's hottest under-converted layer moves by 0.03 K. A numerical
  side-effect worth knowing: a piecewise-linear `f` has a derivative kink at every node, and
  LSODA needs about 3.6 times the steps per slab run with it (61171 against 16703 right-hand-side
  evaluations on one D1 run with 10 volumes; PCHIP 28642); a cubic spline or a fitted smooth law
  has no such cost.
- **Reference solution and verifier files.** `solution/solve.py` keeps its closed-form cage
  effect and its computation is unchanged: `B_f` = 0.5 and `V_ff` = 0.035 are now set in the
  script (the constants the published table follows, which a fit of the table recovers) instead
  of being read from `properties.json`, so its outputs are byte-identical to the previous
  revision's, and its D1 program (651.2 min) meets the 670 min limit (verified end to end with
  `tests/test.sh`: four tests pass, reward 1). It also now stops with an error if its design
  exceeds `max_cycle_time_min` instead of writing it (its margin loop could not have recovered:
  a wider margin only lengthens the cycle). `tests/plant.py` changed only in its docstring;
  `tests/truth/SHA256SUMS.txt` was regenerated for `design.json` and `plant.py`. `expected.json` is
  unchanged and is reproduced from the edited `plant.py` with zero deviation. So that `tests/`
  holds only code and data files, the checksum file is now `SHA256SUMS.txt` (was `SHA256SUMS`;
  `tests/test.sh` names it accordingly) and `tests/.dockerignore` was removed (the package
  carries no bytecode). Checked end to end after the change: the oracle's outputs give reward 1,
  no outputs give reward 0, and a tampered `plant.py` leaves the reward at 0 with exit 2.

### Hardening (previous revision)

Platform pass@1 on the previous revision (one attempt per model, as reported by the platform):
Claude Opus 5.5 (low) reached reward 1. GPT-6-sol (max) reached reward 0, but failed only
`test_predictions_match_truth`; its D1 program passed. In an earlier pass@1 run both models
reached reward 1.

Why they passed: instruction.md stated the diffusion-control laws for `f`, `kp` and `kt` in
full and named the seven unknown constants, so the scientific core (which physics controls
the gel and glass effects) was given away. What was left was fitting seven constants and
careful numerics. Claude's transcript fits exactly those seven constants (it recovered
`E_t` = -6138 K against -6000 K). GPT's transcript does the same; its fit came out a few kelvin
too reactive at 60 C, so it predicted a runaway in C1 and C4 (mid-plane excess 92.7 K and
92.9 K against 16.0 K and 19.6 K). The truth model shows how close these castings sit to
runaway: with the bath 1 K hotter, C1 and C4 already leave their bands, and with it 2-3 K
hotter they run away.

Changes:

- **instruction.md, kinetics.** The gel-effect law for `kt` (free-volume onset, chain-length
  and temperature terms, reaction-diffusion floor), the glass-effect law for `kp` and their
  seven constants are no longer stated. The instruction now says that `f`, `kp` and `kt` all
  become diffusion controlled, that the published `kp0`, `kt0` and `f0` are the values without
  diffusion limitation (no conversion is given below which `kp` or `kt` equals them), and that
  the dependence of `kp` and `kt` on the conditions at each point (temperature, composition and
  the polymer formed there so far), at any conversion, has to be inferred from the ampoule
  data, with no form prescribed. The Arrhenius form of the `A`/`E_over_R_K` correlations and
  the gas constant are now stated, and the reward sentence names the Outputs contract. The free volume `V_f` and the cage effect on `f` (`B_f`, `V_ff` in
  `properties.json`) stay published as separately measured. Without the cage-effect constants,
  the reference family's nine-constant fit has minima of similar cost that disagree on C3's Mn
  (see Reference solution). The acceptance
  section now says that the reference values are the physical system's, and the input files
  are named with their full `/root/data/` paths.
- **instruction.md, ampoules.** The sentence "They are not isothermal." is gone. The
  `UA_W_per_K` heat-flow law, which defines the ampoule energy balance, is unchanged.
- **Bands.** Three casting bands are tightened, calibrated against the comparators: t80 8 % ->
  6 %, `X_avg_t1` 0.015 -> 0.010, molar masses a factor 1.20 -> 1.15. The distribution band
  stays at L1 0.06: the independent model that follows that revision's instruction reaches 0.042,
  which a 0.05 band would leave with too little margin. The two models that follow the current
  instruction pass: the reference solution (worst quantity 0.53 of its band) and the independent
  `marten_ham_pubf` (0.71). So do the earlier-setup comparators that fitted their own cage
  effect (`fv_chain`, `coarse_grid`, `marten_ham`; at most 0.71). Every wrong-physics model
  fails (Calibration of the casting bands).
- **Unchanged.** The data (only the free-text `description` of `properties.json` was
  corrected: the published values are those without diffusion limitation, and the density
  forms take degrees Celsius), the casting truth, the D1 limits (720 min), the plant model's
  equations, the verifier's code apart from the three band constants, and the reference
  solution's computations (only its comments changed). `tests/plant.py` changed only in its
  docstring, so `tests/truth/SHA256SUMS.txt` was regenerated.
- **Evaluated and rejected: a shorter D1 cycle limit.** A limit near 400 min would have
  required a hot-start program. The fitted models misjudge hot starts in a way the ampoule
  data cannot resolve, so the limit would have failed correct solutions. See Calibration of
  the cure program and of the 720 min limit.

### Earlier revision

- **instruction.md, units.** `T` is now defined once as a temperature in kelvin, with the
  Celsius conversion stated; both heat-flow laws are written with `T_bath` in kelvin
  (`UA_W_per_K*(T - T_bath)`, `h*(T_s - T_bath)`).
- **instruction.md, author-chosen items.** The diffusion-control laws were stated in full
  (logistic free-volume switches for `f`, `kp` and `kt`, the `m*ln(Mw/1e5)` and
  `E_t*(1/T - 1/343.15)` terms, the `C_rd*kp*[M]` reaction-diffusion floor), and all nine
  of their constants were named; the Hardening revision withdrew the `kp` and `kt` laws again
  (see above), and the current revision states them once more, with the seven constants
  withheld (Revision notes). Also stated since then: the reaction scheme, the initiation rate,
  concentrations per litre of the monomer-polymer mixture, the heat capacity basis, the
  ampoule start state, and the slice-by-slice GPC convention, which
  `tests/plant.py:ps_equivalent_averages` used to make the data. `solution/solve.py`
  forward-models that convention instead of converting averages.
- **tests, coverage.** The bubble limit is enforced in continuous time on the integrator's
  dense output, not at 10 s samples, and on 80 volumes instead of 24, within 0.03 K of a
  160-volume run (see Verification). A program that passes a 10 s sampled check but does break
  the limit between samples now fails.
- **Identifiability.** The oracle's fit used to start at the data-generating values of six
  constants. From neutral starts, the nine-constant fit has a lower-cost minimum that fails
  casting C3 (see Reference solution). The two cage-effect constants `B_f` and `V_ff` are
  therefore published in `properties.json` (`initiator_efficiency_cage_effect`); the other
  seven are withheld. That revision left the casting truth, the D1 limits and every acceptance
  band unchanged.

## Difficulty

PMMA sheet is made by casting bulk methyl methacrylate between cell walls in a water bath.
Past roughly 20-35 % conversion termination slows by orders of magnitude (the
Trommsdorff-Norrish gel effect). Near the glass transition, propagation and initiator
efficiency fall too, and above about 100 C the propagation equilibrium caps conversion. The
published rate constants are chemically controlled values; in the data-generating model kt is
already 5-20 % below its published value at a few percent conversion, and far below it in the
gel regime.

The instruction states the free volume of the mixture, the separately measured cage effect
on the initiator efficiency, and the gel-effect law for `kt` and the glass-effect law for
`kp` in closed form. It does not give the seven constants of those two laws. The agent has to
infer them from self-heating ampoule kinetics and polystyrene-calibrated GPC. The ampoules
determine them: every converged seven-constant fit from neutral starts reaches the same
values (Reference solution), and within the stated family, removing the chain-length term of
the gel onset, its temperature term or the reaction-diffusion floor makes the fit 2.6 to 112
times worse (in cost) and fails the castings. Fit cost (`scipy.optimize.least_squares` cost,
half the sum of squared scaled residuals) compares laws fitted with the same residuals; it is
not the pass criterion. The bands are not tuned to the reference implementation: a law from
another family, `marten_ham_pubf`, whose gel onset depends on chain length and temperature and
which has a reaction-diffusion term, passes at cost 127.4 (Calibration of the casting bands;
it was fitted in the revisions that withheld the laws). The agent then carries the fitted
model into a thick casting with a population balance, shrinkage and transient conduction.
Several realistic features break the usual shortcuts:

- The ampoules self-heat by 1.3-9.6 K.
- The GPC averages are polystyrene-equivalent, 10-15 % below the absolute values.
- The data are sparse where the gel effect is fastest.
- The castings are conduction-limited: the mid-plane runs 7.5-19.6 K above the bath,
  against 1.4-3.3 K for a lumped model.
- The castings extrapolate beyond the calibration. One is post-cured at 120 C, where
  depropagation sets the residual monomer. Two (C1 and C4, 60 C) sit just below runaway:
  in the data-generating model a bath 1 K hotter already moves their mid-plane excess out of
  its band (C1 16.0 -> 20.6 K, C4 19.6 -> 33.7 K), and 2-3 K hotter they run away to about
  70-90 K. A model that is slightly too reactive at 60 C predicts a runaway that does not
  happen.

Each casting's molar-mass distribution is graded bin by bin and must be accumulated from
instantaneous chain-length distributions; a Schulz-Zimm or log-normal shape with the exact
Mn and Mw fails every casting.

The agent must also design a bath program for a 20 mm sheet (D1) that finishes within 670 min
(720 min three revisions ago) with at most four program points (12 before the previous revision),
keeps every layer below 90 % conversion at or under 95 C, and ends at a mass-average conversion
of at least 0.975. Held at a constant bath temperature, the sheet runs away above about 48.4 C:
the mid-plane peaks at 72 C at 48.0 C, but at 148 C at 49 C. Among hold-then-ramp programs the
fastest safe one holds just below that boundary, and each kelvin of caution costs about 75 min;
starting hotter and cooling before the gel effect is faster still, but with four points such a
program has either no chill hold or no post-cure hold, and the only ones that pass in the plant
end within 0.0023 of the target and run away when shifted 0.4 K (Revision notes). The program
is run in the
data-generating model, so a design pushed to the edge of the agent's own model fails if that
model misplaces the boundary; two fitted comparators (`fv_chain`, `marten_ham`) put it about
0.3 K and 0.4 K too high (their no-margin holds at 48.6 C and 48.73 C against 48.33 C).

The data are synthetic. They come from a free-volume diffusion-control model
(Marten-Hamielec style chain-length-dependent gel onset, reaction-diffusion floor, glass and
cage effects, depropagation), with nine constants chosen to reproduce AIBN-initiated bulk
MMA, plus gravimetric and GPC-scale noise. instruction.md states its free volume, its cage
effect and its laws for `kp` and `kt`, not the seven constants of those laws. Polymer reaction
engineers do this when designing cure cycles for cast acrylic sheet.

## Reference solution

`solution/solve.py` keeps every published constant and the free volume as instruction.md states
it, and uses the cage effect in the closed form that the published table follows (its two
constants are set in the script; a fit of the table recovers them, see Revision notes). For
`kp` and `kt` it implements the laws instruction.md states (the family the data were
generated with): a logistic switch in kt whose onset depends on free volume, cumulative Mw
and temperature, with a reaction-diffusion floor proportional to kp[M]; a logistic
critical-free-volume switch for kp (glass effect); and net propagation kp([M] - [M]_eq) with
the published ceiling equilibrium. The seven constants of these two laws are fitted by
weighted least squares to all conversion points (scale 0.01) and to log Mn and log Mw (scale
0.05), each ampoule being a well-mixed vessel that loses heat through its stated UA. The
independent Marten-Hamielec comparator (Calibration of the casting bands) shows that the
bands are not tuned to the reference implementation.

What instruction.md leaves to the agent, and why it is not an unstated convention: every
convention that fixes the meaning of an answer is stated (units, start states, heat-flow
signs, mixture and heat-capacity rules, the GPC slice convention, regions, bins,
interpolation, the bubble criterion), and so are the laws for `kp` and `kt`, with their
reference points (343.15 K and 1e5 g/mol in `z_t`, `TREF` in the script) and the units and
signs of their constants. The one thing not given is the value of those seven constants.
They are properties of the physical system to be inferred from the ampoule data, and
instruction.md says so. It also says that the published values are those without diffusion
limitation and that the reference values are the physical system's, which follows the stated
kinetics at its constants, not the reference solution's. Equivalent methods pass. The
independent Marten-Hamielec model `marten_ham_pubf` was fitted in the revisions that withheld
the laws: it follows the published cage effect but uses its own laws for `kp` and `kt`, a
piecewise onset event with power-law and exponential post-onset termination and an
exponential glass effect, with none of the stated reference points. It passes every casting
band (Calibration of the casting bands) and designs a D1 program that passes in the plant
(Calibration of the cure program and of the 720 min limit), which shows that the bands admit
a structurally different fit of the same data, not only the reference implementation.

The ampoules constrain every term of the stated laws. The same fit with one term removed:

| `kp`/`kt` law (cage effect as published) | Fit cost (137 residuals) | Castings |
|---|---|---|
| reference: gel onset with chain length and temperature, reaction diffusion, glass effect | 43.6 | pass, worst quantity 0.53 of its band |
| no chain-length term in the gel onset | 113 | fail (3 checks) |
| no temperature term in the gel onset | 327 | fail (3 checks) |
| neither (plain free-volume switch, like the published cage effect) | 327 | fail (4 checks) |
| no reaction-diffusion floor | 4868 | fail (35 checks) |
| plain switch and no reaction diffusion | 6554 | fail (C1 never reaches 80 % conversion) |

The polystyrene-equivalent GPC averages are modelled the way the instruction says the GPC
computed them, slice by slice. With `M_PS = C M^b`, `b = (1 + a_PMMA)/(1 + a_PS)`, the slice
sums `sum(W M_PS)` and `sum(W / M_PS)` are fractional moments of order `+b` and `-b`. For the
gamma-shaped instantaneous distributions these are closed-form, so they are integrated over
time alongside the dead-polymer production. At the hidden constants this agrees with the
fine-binned slice calculation of `tests/plant.py` to within 0.1 % at every GPC point.

The fit starts from a 2 x 2 x 2 design over the gel-onset constants (A_t in {2, 5}, V_ft in
{0.155, 0.170}, E_t in {-2000, -8000} K), combined with each of two generic settings of the
other four (m = 0.3 or 1.0, C_rd = 300 or 50 L/mol, B_p = 0.7 or 1.5, V_fp = 0.025 or 0.020).
No start value is a data-generating value, and no pair is centred on one. The best start of
each setting is refined by bounded least squares (bounds `LOWER`/`UPPER` in `solve.py`: switch
steepness at least 0.05, V_ft at least 0.05, V_fp at least 0.005, 0 <= m <= 3, C_rd at least
1 L/mol, finite upper bounds; none binds at the fit), and the lower-cost fit is kept. Both settings reach the same constants:
A_t = 3.04, V_ft = 0.1638, m = 0.524, E_t = -6140 K, C_rd = 106 L/mol, B_p = 4.49 and
V_fp = 0.0265, at cost 43.64 (45.29 at the data-generating constants). Separate probes from
two more generic settings: one (m = 0.6, C_rd = 1000 L/mol, B_p = 3.0, V_fp = 0.035) reaches
the same constants. The other (m = 0.1, C_rd = 10 L/mol, B_p = 0.3, V_fp = 0.015) stalls at
cost 2004 without converging, far from any fit. Every start that converged reached the same
constants. B_p is only loosely determined (the data-generating
value is 1.0), but every casting comes out well inside its bands (the `oracle` row under
Calibration of the casting bands).

Why the cage effect is published (as the table `f_factors` against `V_f_nodes` since this
revision, as the constants `B_f` and `V_ff` before): with all nine constants free, the ampoules cannot
separate the glass switch on kp from the cage switch on f, because both act near the limiting
conversion at 50-70 C. The nine-constant fit has several minima of similar cost, and they
disagree about C3. Its 120 C post-cure starts new chains from the AIBN left after 960 min at
45 C, so C3's Mn depends directly on f:

| Nine-constant fit | Cost | C3 Mn / truth | C3 distribution L1 | C3 verdict |
|---|---|---|---|---|
| lowest found | 41.01 | 0.54 | 0.090 | fail |
| another minimum | 42.52 | 1.03 | 0.021 | pass |
| unbounded fit with B_p = -30 and V_fp < 0, so kp = kp0 throughout (glass effect off) | 45.27 | 0.60 | 0.026 | fail |
| data-generating constants | 45.29 | 1.00 | 0.000 | pass |

The best fit of the data therefore failed the task. The previous oracle passed only because
its fit started at the data-generating values of six constants. With the cage-effect
constants published (in practice they are measured separately), every converged
seven-constant fit from the starts above reaches the same constants, and they pass every
casting.

Each casting is a symmetric slab of 30 finite volumes in a mass coordinate (each volume keeps
its monomer mass and shrinks with its density), with composition-dependent conductivity and
convective walls to the programmed bath. The molar-mass distribution is accumulated from
instantaneous chain-length distributions: geometric live radicals with depropagation in the
radical balance, weight ~ n q^n from disproportionation and transfer, ~ n^2 q^n from combination.

For D1 the solution minimizes the cycle time over a two-stage family (hold, ramp at 1 K/min to
a post-cure hold, ended when the model reaches the target plus 0.005; at most the four points
`design.json` allows, and the shape its previous three-stage optimum already had) in its own
model, coarse to fine: Nelder-Mead on 10 volumes from two safe starts in parallel (45
evaluations each; both reach the same design), then on 30 volumes the earliest ramp start that
keeps the margined bubble limit, by bisection to 1 min. The fine stage replaced the previous
"re-check on a finer grid and widen the margins if it fails" loop: on the 10-volume grid the
three-parameter optimum (cycle 647 min) fails the 30-volume check by a few tenths of a kelvin,
and a wider margin would have lengthened the cycle instead of fixing the timing. Every design
run stops as soon as the model reaches the margined target and aborts as a runaway when a
layer below the bubble conversion is 10 K above the limit, with a 20 s wall-clock budget per
run, so no candidate drags the stiff solver through a runaway; the D1 search takes about
4 min on 2 CPUs (it took 15-30 min before). The program must also keep the bubble limit with
the bath 1 K hotter, which covers an error of that size in the model's runaway boundary and
about 10 % in its gel-effect timing. The 0.005 target margin covers the fitted model's error
in late conversion. With a 0.002 margin the plant ended only 0.0004 above the target, because
the post-cure is still converting about 0.001 per minute when the program ends. No random
seed is used.

| Oracle runs (previous revision, in containers) | Result |
|---|---|
| Setting | agent image built from `environment/` (python 3.11, numpy 1.26.4, scipy 1.13.0, pandas 2.2.2), `--network none`, 2 CPUs, 2 GB; then `tests/test.sh` in the verifier image built from `tests/`, `--network none`, 1 CPU, 1 GB |
| Runs | 3 sequential runs (the last on the final package), byte-identical outputs, all accepted by the verifier (all four tests pass, reward 1, verifier 4.1-4.7 s) |
| Wall time | 23.5, 22.9 and 22.0 min (2 CPUs, with another 2-process job on the same 4-core host) |
| Peak memory | 308-332 MB |
| Fitted constants | A_t 3.04, V_ft 0.1638, m 0.524, E_t -6140 K, C_rd 106 L/mol, B_p 4.49, V_fp 0.0265 (cost 43.64) |
| Castings, worst over C1-C4 | t80 1.5 % (band 6 %), peak time 1.2 % (12 %), dT 0.38 of its band, X_avg_t1 0.0012 (0.010), X_avg_end 0.0032 (0.006), molar mass factor 1.072 (1.15), distribution L1 0.017 (0.06) |
| D1 program | hold 47.59 C until 587.4 min, ramp at 1 K/min to 108.50 C at 648.3 min, hold to 651.2 min |
| D1 in the plant | hottest under-converted layer 92.57 C (continuous time, 80 volumes), final mass-average conversion 0.9791 |

| Oracle run (previous revision; the pinned numpy 1.26.4 / scipy 1.13.0 / pandas 2.2.2 in a Python 3.11 venv on a contended 2-CPU host, not in the containers) | Result |
|---|---|
| Fit | the same constants as before (A_t 3.04, V_ft 0.1638, m 0.524, E_t -6140 K, C_rd 106 L/mol, B_p 4.49, V_fp 0.0265; cost 43.64). The two generic settings reach costs within 1e-3 of each other, and multi-threaded BLAS moved their trailing digits between runs, which flipped `min()` between them (B_p 4.49 or 4.48; predictions then differ by at most 0.4 % in a distribution bin, the D1 program by less than 1e-4); the script now pins BLAS to one thread and keeps the first setting whose cost is within 0.05 of the best |
| Castings, worst over C1-C4 | t80 1.5 % (band 6 %), peak time 1.2 % (12 %), dT 0.38 of its band, X_avg_t1 0.0012 (0.010), X_avg_end 0.0032 (0.006), molar mass factor 1.072 (band 1.10, 0.73 of it), distribution L1 0.017 (band 0.05, 0.34 of it) |
| D1 program | `[[0, 47.607], [584.72, 47.607], [648.33, 111.22]]`: hold 47.61 C until 584.7 min, ramp at 1 K/min (built a fraction 1e-9 slower, `RAMP_SLACK`, so the exact ramp check holds by 6e-8 min rather than by rounding luck), end on the ramp at 648.3 min (3 points) |
| D1 in the plant | hottest under-converted layer 92.86 C (continuous time, 80 volumes), final mass-average conversion 0.9791, 21.7 min under the cycle limit |
| Wall time | 12.5 and 12.8 min for the two final runs on a loaded host (7.9-8.5 min when quiet; fit about 4.5-7, castings 1, D1 search 3.4-5.6); the two runs' outputs are byte-identical; `tests/test.sh` on them: four tests pass, reward 1 |

| Oracle run (this revision; the pinned numpy 1.26.4 / scipy 1.13.0 / pandas 2.2.2 and pytest 8.4.1 in a Python 3.11 venv on a 2-CPU cloud host, `/root/data`, `/root/results`, `/tests` and `/logs/verifier` laid out as in the containers) | Result |
|---|---|
| Fit | the same constants (A_t 3.04, V_ft 0.1638, m 0.524, E_t -6140 K, C_rd 106 L/mol, B_p 4.49, V_fp 0.0265; cost 43.64, 137 residuals); both generic settings refine to cost 43.638 |
| Castings, worst over C1-C4 (graded by the verifier's `score()`) | t80 1.5 % (band 6 %), peak time 1.2 % (12 %), dT 0.38 of its band, X_avg_t1 0.0012 (0.010), X_avg_end 0.0032 (0.006), molar mass factor 1.072 (band 1.10), distribution L1 0.017 (band 0.05): no failure |
| D1 program | `[[0, 47.607], [584.72, 47.607], [648.33, 111.22]]`, the previous revision's program to every printed digit |
| D1 in the plant | hottest under-converted layer 92.86 C (continuous time, 80 volumes), final mass-average conversion 0.9791 |
| `tests/test.sh` | hash check of `expected.json`, `design.json` and the edited `plant.py` passed, four tests passed, reward 1, exit 0 (verifier 6 s); with the previous revision's `plant.py` put in place of the edited one, reward 0 and exit 2 ("truth or plant model does not match its recorded hash") |
| Wall time | about 9 min (fit about 4, castings 1, D1 search 4.3) |

The casting predictions are those of the previous revisions' oracle up to the fit's tie-break
(the other setting's B_p 4.48 moves the predictions by at most 0.4 % in a distribution bin and
1e-5 in the scalars); the D1 program changed with the family and the search.

## Verification

`tests/test.sh` runs with `set -euo pipefail` and is the only writer of
`/logs/verifier/reward.txt`. It writes `0` before anything else, so a crash at any later point
still leaves a valid binary result. It then checks that the verifier files exist and that
`truth/expected.json`, `truth/design.json` and `plant.py` match `truth/SHA256SUMS.txt`, runs pytest
with a CTRF report, and has `tests/decide.py` confirm that the report lists exactly the four
tests below, none skipped, consistent with the exit status. Only then does it write `1` (every
test passed). An infrastructure error (missing file, hash mismatch, pytest exit other than 0 or
1, rejected report) leaves the reward at 0 and exits 2. pytest, numpy and scipy are baked into
`tests/Dockerfile`, at the agent image's numpy and scipy versions.

| Test | What it checks |
|---|---|
| `test_castings_conversion_and_exotherm` | exact schema of `/root/results/predictions.json` (prerequisite), then `t80_min`, `dT_mid_max_K`, `t_dT_mid_max_min`, `X_avg_t1`, `X_avg_end` of each casting against `tests/truth/expected.json` |
| `test_castings_molar_masses` | the schema prerequisite, then the four cumulative molar masses of each casting |
| `test_castings_mwd_fractions` | the schema prerequisite, then the binned distribution of each casting (L1) |
| `test_cure_program_in_plant` | exact schema of `/root/results/cure_program.json` and the program limits in `design.json` (prerequisite; an invalid program is never run), then the program run through `tests/plant.py`: bubble limit in continuous time, target conversion |

A well-formed file passes no test by itself: the two schema checks are prerequisites of the
physics tests, so a malformed `predictions.json` fails three tests and a malformed program one.

| Casting quantity | Acceptance band |
|---|---|
| `t80_min` | within 6 % relative error of the reference |
| `t_dT_mid_max_min` | within 12 % relative error of the reference |
| `dT_mid_max_K` | absolute error at most max(1.5 K, 25 % of the reference) |
| `X_avg_t1` | absolute error at most 0.010 |
| `X_avg_end` | absolute error at most 0.006 |
| `Mn_avg_end_g_per_mol`, `Mw_avg_end_g_per_mol`, `Mw_inner_end_g_per_mol`, `Mw_outer_end_g_per_mol` | within a multiplicative factor of 1.10 (1.15 before the previous revision) |
| `mwd_weight_fractions` | sum over the 16 bins of the absolute difference at most 0.05 (0.06 before the previous revision) |

Both schemas are exactly those stated in `instruction.md`. Extra or missing keys, unknown or
duplicate ids, non-finite or non-numeric values, a malformed distribution and a missing or
unreadable file all fail; an invalid cure program is never run in the plant.

The plant run uses `tests/plant.py`, the data-generating model, with 80 finite volumes per
half-thickness. `run_plant()` in `tests/test_state.py` integrates it step by step with LSODA
at rtol 1e-7 and exactly the settings of `plant.simulate_slab` (steps of at most 10 s). It
enforces the bubble limit in continuous time on the integrator's solution, not at output
samples. For each layer, `g(t) = min(T - 95 C, 100 K * (0.90 - X))` is positive exactly when
the layer is below the bubble conversion and hotter than the bubble temperature. Each
integration step's dense output is sampled at 17 equally spaced times, step ends included.
Around every sampled local maximum of `g` within 2 K of zero, a bounded Brent search (time
resolution 1e-4 s) locates the maximum of `g` on the dense output. Any `g > 0` anywhere in
`[0, cycle time]` fails the program, resolved to about 1e-5 K. Sampling 257 instead of 17
points per step moves the located maximum by less than 0.001 K. An independent check that
samples every step at 1025 points agrees with the verdict on 9 of 9 programs, one of them only
0.0001 K below the limit. A 720 min program runs in 5-9 s.

The continuous check matters because the hottest under-converted temperature usually falls
at the instant a layer passes 0.90 conversion while still heating, which lies between fixed
samples. On the D1 programs tested, the continuous maximum reads up to about 0.25 K above the
maximum over 10 s samples. The program
`[[0, 48.332656], [495.1, 48.332656], [554.5, 107.732656]]` shows the gap. Sampled every 10 s
on 80 volumes, its hottest under-converted layer is at 94.974 C, a pass. In continuous time,
layer 28 reaches 95.002 C at conversion 0.89993 at 500.04 min, and the verifier fails it.

Spatial resolution is converged. Hottest under-converted layer on 24, 40, 80 and 160 volumes,
from `run_plant()` with the programs at full precision (shown here to 4 decimals):

| Program | 24 | 40 | 80 (verifier) | 160 |
|---|---|---|---|---|
| hold-and-ramp optimum: [0, 48.3114], [495.0778, 48.3114], [554.5, 107.7336] | 94.353 C | 94.119 C | 94.019 C | 93.994 C |
| current reference solution: [0, 47.5914], [587.4201, 47.5914], [648.3334, 108.5047], [651.1667, 108.5047] | 92.553 C | 92.564 C | 92.568 C | 92.571 C |
| `isothermal_amp` design (outer layer during the ramp): [0, 47.0454], [615.4326, 47.0454], [673.9741, 105.587], [682.3333, 105.587] | 95.518 C | 95.595 C | 95.604 C | 95.605 C |
| `[[0, 47.822], [537.8162, 47.822], [596.4519, 106.4577]]` (outer layer) | 99.987 C | 100.063 C | 100.107 C | 100.112 C |

80 volumes are within 0.03 K of 160 volumes at the mid-plane and at the walls. Coarser grids
are not one-sided: they overstate the mid-plane and understate the wall layers. The truth is
reproducible from the package with the pinned numpy and scipy, to within 1e-14 relative on
every value (checked for the previous revision; for this revision the recipe below, run on the
edited `plant.py` with the pinned stack, reproduces every value of `expected.json` with zero
deviation). For each casting in `castings.json`, run
`plant.simulate_slab` with 40 volumes on the grid `t = linspace(0, 60 t_end_min, round(6
t_end_min) + 1)` s. Then `t80_min` is the linear interpolation of `X_avg` between the two
samples that bracket 0.80. `dT_mid_max_K` and `t_dT_mid_max_min` come from the first maximum of
`T_mid` minus the bath on that grid. `X_avg_t1` is interpolated at `t1_min`, and the end values
are the last sample. The inner and outer Mw come from `simulate_slab`'s `outer_frac` = 0.10,
and the distribution is `plant.cumulative_mwd` over the run, divided by its sum.

### Calibration of the casting bands

Each comparator reads only the agent-visible inputs, fits its own kinetics, predicts every
casting including the distribution, and is graded by the verifier's `score()`. Worst errors over
the four castings:

The molar-mass and distribution columns below report the measured factor and L1 distance;
their bands are now 1.10 and 0.05 (1.15 and 0.06 when the table was made). Every verdict is
the same under the new bands: the correct models' worst factors are 1.040-1.072 and their
distributions 0.017-0.042, and every failing row fails on at least one other quantity as well.

| Comparator | What differs | Fit cost (137 residuals) | t80 (band 6 %) | Peak time (12 %) | dT (fraction of band) | X_avg_t1 (0.010) | X_avg_end (0.006) | Molar mass factor (band 1.10) | Distribution L1 (band 0.05) | Verdict |
|---|---|---|---|---|---|---|---|---|---|---|
| `oracle` | `solution/solve.py`: the laws for `kp` and `kt` as instruction.md now states them (the reference family), seven constants fitted, cage effect as published | 43.64 | 1.5 % | 1.2 % | 0.38 | 0.0012 | 0.0032 | 1.072 | 0.017 | pass |
| `fv_chain` | reference family | 45 | 1.4 % | 1.2 % | 0.41 | 0.0019 | 0.0021 | 1.046 | 0.019 | pass |
| `marten_ham_pubf` | independent, fitted under the previous revisions' instruction (laws withheld): Marten-Hamielec onset as a per-layer event (chain length and temperature), power-law and exponential post-onset termination, reaction-diffusion term, exponential glass effect, literature exponents fixed; cage effect as published; five constants fitted (best minimum) | 127.4 | 4.1 % | 7.8 % | 0.71 | 0.0029 | 0.0010 | 1.051 | 0.042 | pass |
| `marten_ham_pubf`, worse minimum | the same model at the lowest minimum a cold 54-start search reached | 187.1 | 8.0 % | 10.2 % | 1.47 | 0.0061 | 0.0122 | 1.348 | 0.075 | fail |
| `marten_ham` | independent, as `marten_ham_pubf` but with its own exponential cage effect fitted instead of the published one (six constants) | 127.0 | 3.9 % | 7.8 % | 0.71 | 0.0028 | 0.0016 | 1.068 | 0.035 | pass |
| `coarse_grid` | `fv_chain` on 5 volumes | - | 0.9 % | 1.2 % | 0.34 | 0.0015 | 0.0021 | 1.040 | 0.018 | pass |
| `isothermal_amp` | ampoules fitted as isothermal | 210 | 9.4 % | 6.2 % | 1.27 | 0.0041 | 0.0023 | 5.079 | 0.131 | fail |
| `ps_as_absolute` | GPC values taken as absolute | 93 | 2.3 % | 1.4 % | 0.26 | 0.0070 | 0.0033 | 6.582 | 0.134 | fail |
| `no_deprop` | no depropagation | 45 | 1.3 % | 1.4 % | 0.39 | 0.0019 | 0.0146 | 1.220 | 0.032 | fail |
| `lumped_casting` | each casting one well-mixed lump | - | 19.1 % | 61.2 % | 3.45 | 0.0592 | 0.0102 | 1.612 | 0.263 | fail |
| `classical` | no diffusion control | 52426 | never 80 % | 92.1 % | 12.36 | 0.6099 | 0.5899 | 15.050 | 1.130 | fail |
| `no_chain` | `oracle` without the chain-length term in the gel onset | 113 | 8.5 % | 11.4 % | 0.36 | 0.0160 | 0.0028 | 1.053 | 0.031 | fail |
| `no_Et` | `oracle` without the temperature term in the gel onset | 327 | 10.2 % | 14.5 % | 0.998 | 0.0188 | 0.0028 | 1.131 | 0.043 | fail |
| `plain` | `oracle` with a plain free-volume switch for kt (neither term) | 327 | 10.2 % | 14.5 % | 1.004 | 0.0189 | 0.0035 | 1.130 | 0.043 | fail |
| `no_rd` | `oracle` without the reaction-diffusion floor | 4868 | 10.5 % | 29.2 % | 44.24 | 0.1116 | 0.0931 | 62.8 | 0.684 | fail |

The independent model's tightest margins are the distribution (0.042 against 0.05, 0.84), the
C4 temperature excess (0.71 of its band), t80 (4.1 % against 6 %, 0.68) and the C3 peak time
(7.8 % against 12 %, 0.65; C3's 45 C bath lies below the calibrated range, which is why this
band is wider than the 6 % of t80). These margins limit how far the bands could be tightened:
t80 went to 6 %, and the dT and peak-time bands were kept; the distribution band went to 0.05
in the previous revision (0.045 would leave the independent model at 0.93 of it, 0.04 would fail it).
The `X_avg_t1` (0.010) and molar-mass (1.10) bands leave every correct model at or below 0.29
and 0.73 of the band (the 0.73 is the reference solution's C3 `Mn`, factor 1.072; the
independent model is at 0.52). The reference solution, which implements the stated laws,
passes with its worst quantity at 0.73 of its band; `marten_ham_pubf` at 0.84.
`marten_ham_pubf` uses the published cage effect with its own laws for `kp` and `kt`: a
piecewise onset rather than logistic switches, and none of the stated reference points. It
shows that the bands are not tuned to the reference implementation: a structurally different
diffusion-control model, fitted to the same data, also passes. It has to be fitted with care. A cold search from 54 generic starts (the 6 best
refined) stopped in minima at cost 187-497, and the best of those fails. The minimum at cost
127.4 was reached by continuation: fit the model first with a free cage-effect term (the
`marten_ham` row, an easier landscape), then fix the published cage effect and refine. Least
squares does not move from there, and the published cage effect and the fitted one are nearly
interchangeable on these data (127.4 against 127.0). `marten_ham_pubf` is fitted with the
comparator engine of the earlier setup (GPC averages converted by universal calibration), so
its cost is comparable with the `marten_ham` row but not with the `oracle` row. Converting
averages instead of slices is a small approximation here: with b = 0.985, for the
most-probable and combination distributions it moves the polystyrene-equivalent Mw by 0.2-0.3 %
and Mn by 0.4-0.9 %, against the 5 % GPC uncertainty. The `oracle`,
`no_chain`, `no_Et`, `plain`, `no_rd` and `marten_ham_pubf` rows were fitted in the previous revision.
All other rows predate it and were re-graded with the tightened bands; every verdict is
unchanged. `fv_chain` and `coarse_grid` are nine-constant
fits from an earlier setup. Their scripts are not in this package and their starting values
are not recorded, so they are no evidence about the nine-constant fit's identifiability (see
Reference solution). The casting truth and `score()` are unchanged; three band constants were
tightened (Revision notes).

| Distribution check | L1 against the truth |
|---|---|
| Ignoring depropagation in the chain-length distribution | at most 0.016 |
| 60 s instead of 10 s sampling, or 20 instead of 40 volumes | 0.0035 |
| Schulz-Zimm shape with the exact Mn and Mw | 0.067, 0.081, 0.094 and 0.121 (fails) |
| Log-normal shape with the exact Mn and Mw | 0.149-0.298 (fails) |

### Calibration of the cure program and of the 720 min limit

This section was written under the 720 min limit with up to 12 program points and is kept as
recorded; the current limits are 670 min and 4 points (Revision notes, which give the plant's
minimum hold-and-ramp cycle by hold temperature under the current verifier, the scan of
four-point program shapes, and the sensitivity of every passing shape to a 0.4 K shift). Every
design in the tables below that passed at 720 min also meets 670 min, and all of them have at
most four points except `fv_chain`'s, whose two-slope ramp re-expressed as one 1 K/min ramp
(`[[0, 47.65], [588.22, 47.65], [651.2, 110.63], [651.67, 110.63]]`) passes the current
verifier at 91.6 C and X_avg 0.9792; the slack above the slowest of them is about 18 min.

An author-side search (script not included in this package) of the hold-and-ramp family
under the data-generating model without margins finds that the best program holds 48.31 C until 495.1 min, ramps at 1 K/min and
ends at 554.5 min. Programs that start hot and cool before the gel effect reach 289.7 min
(author-side hot-start search, not included). Cycle for a single hold followed by the fastest allowed
ramp, in the data-generating model:

| Hold temperature | Cycle, no margin | Cycle, bubble limit 89 C |
|---|---|---|
| 42 C | 1067 min | 1099 min |
| 44 C | 902 min | 927 min |
| 46 C | 754 min | 779 min |
| 47 C | 684 min | 711 min |
| 48 C | 597 min | 641 min |
| 48.3 C | 555 min | infeasible (the hold alone reaches 90 C) |
| 49 C | runaway to 148 C during the hold | runaway |

Each comparator designs a program in its own fitted model with the same optimiser, graded by
the verifier's `program_errors()` and `plant_errors()`:

| Model (margins) | Program, [min, C] | Cycle | Hottest under-converted layer in the plant | X_avg at the end in the plant | Verdict |
|---|---|---|---|---|---|
| `fv_chain` (target +0.002, bath +1 K) | [0, 47.6], [588, 47.6], [636, 95.3], [636, 95.3], [652, 110.6] | 651.7 min | 91.5 C | 0.9786 | pass |
| `marten_ham` (target +0.002, bath +1 K) | [0, 47.9], [559, 47.9], [622, 110.0] | 621.5 min | 93.4 C | 0.9773 | pass |
| `fv_chain` (target +0.002, no bath margin) | [0, 48.6], [481, 48.6], [542, 110.0] | 542.5 min | 133.9 C (runaway) | 0.9692 | fail |
| `marten_ham` (target +0.002, no bath margin) | [0, 48.7], [465, 48.7], [525, 108.7] | 524.8 min | 150.0 C (runaway) | 0.9301 | fail |
| `isothermal_amp` (target +0.002, bath +1 K) | [0, 47.0], [615, 47.0], [674, 105.6], [682, 105.6] | 682.3 min | 95.5 C (outer layer during the ramp) | 0.9789 | fail |
| `lumped_casting` (target +0.002, bath +1 K) | [0, 68.4], [138, 68.4], [179, 109.4] | 178.7 min | 158.5 C (runaway) | 0.7357 | fail |

These rows were graded by the earlier verifier (24 volumes, 10 s samples); the table rounds
the programs. In the previous revision every design was re-graded at full precision by the current
verifier: `program_errors()` (limits, including the 1 K/min ramp) and then `plant_errors()`
(80 volumes, continuous time). Every verdict holds (programs shown rounded; the rounded
ramps can exceed 1 K/min by up to 0.02 %, the full-precision ones do not):

| Model (margins) | Program, rounded, [min, C] | Hottest under-converted layer, current verifier | X_avg at the end | Verdict |
|---|---|---|---|---|
| `fv_chain` (target +0.002, bath +1 K) | [0, 47.65], [588.22, 47.65], [635.91, 95.34], [636.37, 95.34], [651.67, 110.63] | 91.66 C | 0.9786 | pass |
| `marten_ham` (target +0.002, bath +1 K) | [0, 47.87], [559.4, 47.87], [621.5, 109.97] | 93.57 C | 0.9773 | pass |
| `fv_chain` (no bath margin) | [0, 48.6], [481.15, 48.6], [542.5, 109.95] | 134.11 C | 0.9695 | fail |
| `marten_ham` (no bath margin) | [0, 48.73], [464.89, 48.73], [524.83, 108.67] | 150.04 C | 0.9300 | fail |
| `isothermal_amp` | [0, 47.05], [615.43, 47.05], [673.97, 105.59], [682.33, 105.59] | 95.60 C | 0.9789 | fail |
| `lumped_casting` | [0, 68.37], [137.66, 68.37], [178.67, 109.37] | 158.86 C | 0.7358 | fail |

The hold-temperature table above also comes from the earlier verifier. For the hold-and-ramp
optimum's family (hold until 495.1 min, then 1 K/min to 554.5 min), the hottest hold that
keeps the bubble limit moves by about 0.005 K under the revised verifier, from about 48.328 C
to 48.3326 C.

In the previous revision the independent model `marten_ham_pubf` (published cage effect, its
own laws for `kp` and `kt`; best-minimum fit) designed a program the same way
with the reference solution's margins (target +0.005, bath +1 K), graded by the current
verifier at full precision (shown rounded):

| Model (margins) | Program, [min, C] | Cycle | Hottest under-converted layer in the plant | X_avg at the end in the plant | Verdict |
|---|---|---|---|---|---|
| `marten_ham_pubf` (target +0.005, bath +1 K) | [0, 47.88], [561.4, 47.88], [620.6, 107.09], [626.3, 107.09] | 626.3 min | 92.97 C | 0.9798 | pass |

The correct models pass with the 1 K bath margin; the two tested without it (`fv_chain`, `marten_ham`) fail. The 720 min limit leaves
about 10 % slack above the slower correct design (651.7 min) and the reference solution
(651.2 min), and excludes holds at or below about 46.5 C, including the 45 C cure of casting C3
(about 830 min). It does not demand near-optimality; it demands a design that is fast enough
and safe on the true system.

#### A shorter limit was evaluated and rejected

A limit near 400 min would make every hold-and-ramp program too slow (the fastest, with no
margin, takes 554.5 min) and so require a hot start. Designs in a seven-parameter family (hot
hold, cool to a gel hold, intermediate hold, post-cure ramp) and a five-parameter family
(without the intermediate hold) were optimised with every margin of the reference solution
plus a bubble-limit margin (bath +1 K, target +0.005, bubble limit 94 C up to 0.905
conversion), then graded by the verifier:

| Designing model | Family | Program, [min, C] | Cycle | Plant verdict |
|---|---|---|---|---|
| data-generating model | 7-parameter | [0, 67.1], [39.4, 67.1], [55.3, 51.2], [252.0, 51.2], [297.1, 96.3], [297.8, 96.3], [314.2, 112.6] | 314.2 min | pass (93.36 C, X_avg 0.9801) |
| data-generating model | 5-parameter | [0, 65.8], [56.5, 65.8], [71.8, 50.4], [261.0, 50.4], [323.2, 112.7] | 323.2 min | pass (94.04 C, X_avg 0.9801) |
| reference family (`fv_chain`) | 5-parameter | [0, 68.3], [32.5, 68.3], [48.8, 52.0], [257.4, 52.0], [319.2, 113.7] | 319.2 min | fail: the mid-plane passes 95 C at 166 min at conversion 0.878; hottest under-converted layer 111.5 C |
| independent Marten-Hamielec (`marten_ham`) | 5-parameter | [0, 69.2], [27.1, 69.2], [48.5, 47.7], [270.5, 47.7], [335.2, 112.4] | 335.2 min | fail: the post-cure ramp starts before the gel effect is over; a layer passes 95 C at 294.8 min at conversion 0.8997; hottest under-converted layer 127.3 C |

Both fitted models' designs fail, from opposite sides, although the reference family's
constants are close to the data-generating ones and both models pass every casting band.
Hot-start programs are safe only in a narrow, two-sided window. For the data-generating model's
own 5-parameter design (run in that model on 24 volumes), the hottest layer below 0.90
conversion stays under 95 C with the whole program shifted by 0 to +1.0 K, and exceeds it at
-0.5 K (98.6 C) and at +1.5 K (156 C). Hotter, the gel hold runs away. Cooler, the gel effect
is not over when the post-cure ramp starts, and the ramp runs away in the remaining monomer
(108-153 C at -1 to -2 K). The independent Marten-Hamielec model, which passes every casting
band, predicts a runaway (139-159 C) for that same program at every offset from -2 K to +2 K.
Its gel onset depends on the temperature history, and the constant-temperature ampoules cannot
tell the two onset laws apart. Under a hot-start limit, D1 would turn on a modelling choice the
data cannot settle, and the reference solution's margins would not protect it. The 720 min
limit is kept.

### Nulls, self-test and container checks

| Trivial null | Result |
|---|---|
| `ampoule_lookup`, `ampoule_heated`: nearest ampoule run, dT of 0 or 10 K, Schulz-Zimm distribution | fail 6-10 of the 10 checks on every casting |
| D1: C3 schedule cut at 720 min | X_avg 0.666 |
| D1: C3 schedule compressed to end at 720 min | runaway to 150 C, X_avg 0.933 |
| D1: 60 C hold | runaway to 164 C |
| D1: 130 C from the start | runaway to 164 C, X_avg 0.115 |
| D1: 40 C hold | X_avg 0.209 |

Before the continuous-time verifier, an author-side self-test of 78 cases (exact truth, values just inside and outside every band,
unit mistakes, malformed files and schemas, malformed distributions, invalid programs, plant
runs of the reference optimum and of hot and cold holds) gave the intended verdict in every
case. After the earlier revision that introduced the continuous-time verifier, a self-test
of 88 cases gave the intended verdict in every case. It covers the same ground plus plant
runs of a runaway hold, a 130 C start, a 40 C hold, the reference optimum cut 1 min short, and the between-sample
excursion above. With the bands tightened in the previous revision, a self-test of 80 cases gave the
intended verdict in every case. Its band cases sit at 0.95 and 1.05 of each current band (t80,
peak time, dT, X_avg_t1, X_avg_end, each molar mass, the distribution). It also covers the
schema and unit mistakes, malformed files, invalid programs, and plant runs of the hold-and-ramp
optimum (pass), a 130 C hold and a 40 C hold (both fail). `solution/solve.py` does not read
`tests/` and embeds no truth beyond the two constants of the cage-effect law that the published
table follows (`B_f` = 0.5, `V_ff` = 0.035, recoverable from the table to better than 1e-4).

| Containers built from the previous revision (verifier: networking disabled, 1 CPU, 1 GB) | Result |
|---|---|
| Oracle output | all four tests pass, reward 1, `test.sh` exit 0 |
| nop (no output files) | all four tests fail, reward 0, `test.sh` exit 0 |
| `plant.py` altered after the build | reward 0, `test.sh` exit 2 ("truth or plant model does not match its recorded hash") |
| `truth/expected.json` altered after the build | reward 0, `test.sh` exit 2 |

| `tests/test.sh` on the previous revision (pinned stack in a venv, `/tests`, `/root/results` and `/logs/verifier` laid out as in the containers) | Result |
|---|---|
| Oracle output | all four tests pass, reward 1, exit 0 |
| no output files | all four tests fail, reward 0, exit 0 |
| `plant.py` altered | reward 0, exit 2 ("truth or plant model does not match its recorded hash") |
| `predictions.json` missing one key, valid program | the three casting tests fail on the contract prerequisite, the plant test passes; reward 0 |
| valid predictions, 5-point program | the plant test fails on the program limits (not run in the plant), the three casting tests pass; reward 0 |
| GPT-6-sol's and Claude Opus 5.5's transcript values (stand-in files, see Revision notes) | all four tests fail for each: C1 `t80_min`, C3 `Mn`, the distributions, and the 8- and 7-point programs rejected by the limits |

| `tests/test.sh` on this revision (same layout, pinned stack in a Python 3.11 venv) | Result |
|---|---|
| Oracle output of this revision's `solve.py` | hash check passed, all four tests pass, reward 1, exit 0 |
| previous revision's `plant.py` in place of the edited one (stale hash) | reward 0, exit 2 ("truth or plant model does not match its recorded hash") |
| `expected.json` recomputed from the edited `plant.py` (recipe under Verification) | zero deviation on every value of every casting |

Model-agent results: the platform's pass@1 results on the previous revisions are summarised
under Revision notes. None has been run on this revision yet.
