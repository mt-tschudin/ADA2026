# ADA — Lecture 6: Causal Analysis of Observational Data

**Course:** Applied Data Analysis (CS-401), EPFL  
**Lecturer:** Robert West  
**Lecture:** 6 — Causal Analysis of Observational Data  
**Date:** 14 October 2026

> Main idea: causal conclusions are easiest in randomized experiments. With observational data, treatment assignment is not random, so we must deal explicitly with **confounding**, create comparable treated/control groups where possible, and test how sensitive conclusions are to hidden variables.

---

## 1. Association is not automatically causation

Suppose treated smokers quit more often than untreated smokers.

That does **not** immediately prove that the treatment caused the higher quit rate.

A variable such as **motivation to quit** could influence both:
- whether someone takes the treatment;
- whether someone successfully quits.

Such a variable is a **confounder**.

```text
          Motivation
          /        \
         v          v
    Treatment ---> Outcome
```

The observed treatment–outcome association may therefore combine:
1. a real treatment effect;
2. differences caused by confounders.

---

## 2. Randomized controlled experiments

A randomized controlled experiment has:
- a **treatment** group;
- a **control** group;
- random assignment to treatment.

Because everyone has the same treatment probability, treatment assignment is disconnected from pre-existing subject characteristics.

This makes treatment and control groups comparable in expectation.

```text
Randomization ---> Treatment ---> Outcome

Confounders ---------------------> Outcome
```

The confounders may still affect the outcome, but they no longer determine treatment assignment.

> Randomization is the “holy grail” because it breaks systematic links between confounders and treatment.

---

## 3. Why observational studies are needed

Randomized experiments may be:
- unethical;
- too expensive;
- impossible;
- unrealistic.

Much real-world data is therefore **observational / found data**.

In observational studies:
- the researcher does not control treatment assignment;
- subjects may self-select;
- treatment and outcome may have common causes.

That makes causal interpretation much harder.

---

## 4. Seat-belt example

Suppose we compare death rates between:
- seat-belt wearers;
- non-seat-belt wearers.

A direct difference may be confounded.

For example, **driver cautiousness** may affect both:
- seat-belt use;
- crash severity.

Crash severity then affects survival.

```text
Driver cautiousness ---> Seat belt
        |
        v
Crash severity -------> Survival
```

So the observed difference in survival is not automatically equal to the causal effect of wearing a seat belt.

---

## 5. Matching

A central strategy in observational studies is **matching**.

Ideal goal:

> For each treated subject, find an otherwise identical untreated subject.

Then compare their outcomes.

The lecture’s seat-belt example matches people within the same accident:
- driver + passenger;
- exactly one died;
- exactly one wore a seat belt.

This controls many potential confounders automatically:
- same car;
- same crash;
- same speed;
- same crash severity.

This is a **matched observational study**.

### Core idea

Matching tries to imitate the comparability produced by randomization.

---

## 6. Natural experiments

Sometimes treatment assignment is not randomized by the researcher, but external circumstances make it unusually haphazard.

This is a **natural experiment**.

Examples mentioned in the lecture:
- twin studies;
- Vietnam draft;
- cholera in London.

Natural experiments are valuable because treatment assignment may be close to random even though the researcher did not create it.

---

## 7. Why perfect matching is difficult

There are two major problems.

### 7.1 Unobserved covariates

Two people may look identical in the dataset but differ in something unmeasured.

Example:
- an unobserved genetic factor affects desire to smoke;
- the same factor also affects lung-cancer risk.

Matching cannot control for variables that were never observed.

### 7.2 Combinatorial explosion

Even with observed covariates, finding exact matches becomes nearly impossible when many variables are involved.

Matching simultaneously on age, income, education, health, location, etc. may leave almost no identical pairs.

---

## 8. The naive observational model

A simplifying assumption is:

> **Only observed covariates determine treatment assignment.**

Equivalently:

> People who look comparable in all observed variables are comparable for treatment assignment.

If this assumption were true, then within matched subjects treatment assignment would be close to random.

This is a strong assumption and generally cannot be proven from the observed data.

---

## 9. Propensity scores

Instead of matching on every covariate separately, compress them into one number:

\[
e(x)=P(T=1\mid X=x)
\]

where:
- \(T=1\): treated;
- \(X\): observed covariates;
- \(e(x)\): **propensity score**.

The propensity score is the probability of receiving treatment given the observed covariates.

It can be estimated from data, for example with a model whose:
- inputs = observed covariates;
- output = treatment indicator.

---

## 10. Why propensity scores help

The key property is **balance**.

For subjects with the same propensity score \(p\),

\[
P(X=x\mid T=1,e(X)=p)
=
P(X=x\mid T=0,e(X)=p).
\]

Intuitively:

> Treated and control subjects with the same treatment probability should have similar distributions of observed covariates.

So exact equality on every covariate is not required.

---

## 11. Propensity-score matching

Typical workflow:

1. estimate a propensity score for each subject;
2. pair treated and control subjects with similar scores;
3. compare outcomes in the matched sample.

Exact equality of propensity scores is unlikely, so approximate matching is used.

The lecture frames this as a matching problem:
- treated subjects on one side;
- controls on the other;
- edge cost = propensity-score difference;
- find a low-cost matching.

The exact algorithm is less important than the resulting **balance**.

---

## 12. Always check covariate balance

A matching algorithm returning pairs does not mean the job is done.

You must verify that treated and control groups are actually balanced on observed covariates.

Useful checks include:
- means;
- distributions;
- box plots;
- standardized differences.

The lecture’s key message is:

> **What matters is balance, not the matching method itself.**

Other matching approaches include:
- exact matching;
- Mahalanobis-distance matching;
- coarsened exact matching;
- combinations of methods.

---

## 13. Hidden confounding remains

Even perfect balance on observed variables does not guarantee a valid causal effect.

What we estimate is:

\[
P(T=1\mid \text{observed covariates}),
\]

but the true assignment mechanism may depend on all relevant variables:

\[
P(T=1\mid \text{all covariates}).
\]

These may differ:

\[
P(T=1\mid X_{\text{obs}})
\neq
P(T=1\mid X_{\text{all}}).
\]

So matching controls **observed imbalance**, but cannot automatically remove bias from unobserved confounders.

---

## 14. Sensitivity analysis

Sensitivity analysis asks:

> How strong would an unobserved confounder need to be to overturn the conclusion?

Instead of assuming matched subjects have identical treatment probabilities, allow their treatment odds to differ by a factor

\[
\Gamma \ge 1.
\]

For matched subjects \(k\) and \(\ell\),

\[
\frac{1}{\Gamma}
\le
\frac{\pi_k/(1-\pi_k)}
{\pi_\ell/(1-\pi_\ell)}
\le
\Gamma,
\]

where \(\pi_k\) and \(\pi_\ell\) are their true treatment probabilities.

### Interpretation

- \(\Gamma=1\): no hidden bias; naive model holds.
- \(\Gamma=2\): one matched subject may have up to twice the treatment odds of the other.
- Large \(\Gamma\): increasingly strong hidden confounding is allowed.

As \(\Gamma\to\infty\), the assumption becomes uninformative.

---

## 15. Robust vs. fragile conclusions

Suppose under the naive model you reject

\[
H_0:\text{no treatment effect}.
\]

Now increase \(\Gamma\).

If the result becomes non-significant at

\[
\Gamma=1.1,
\]

then only a tiny amount of hidden bias is needed to overturn the conclusion.

The result is fragile.

If it takes something like

\[
\Gamma>6,
\]

then a very strong hidden difference in treatment odds would be required.

That makes the result much more robust, provided such a strong hidden factor is implausible according to domain knowledge.

> Sensitivity analysis does not prove the absence of hidden confounding. It quantifies how strong hidden confounding would have to be.

---

## 16. Mechanical vs. scientific parts

The lecture separates observational causal analysis into two parts.

### Mechanical part
Construct comparable groups using observed variables.

Examples:
- exact matching;
- propensity-score matching.

### Scientific part
Assess whether hidden variables could still explain the result.

Tools include:
- sensitivity analysis;
- natural experiments;
- domain knowledge;
- substantive arguments.

This scientific part cannot be replaced by matching algorithms alone.

---

## 17. Practical workflow

```text
Causal question
      ↓
Define treatment and outcome
      ↓
Identify likely confounders
      ↓
Can you randomize?
   ↙            ↘
 yes            no
 ↓              ↓
RCT      observational study
                ↓
        measure relevant covariates
                ↓
          match / balance groups
                ↓
          verify covariate balance
                ↓
         compare treated vs control
                ↓
      assess hidden confounding
          (sensitivity analysis)
                ↓
        interpret cautiously
```

---

## 18. Key distinctions

| Concept | Meaning |
|---|---|
| **Treatment** | Exposure/intervention whose effect is studied |
| **Outcome** | Variable potentially affected by treatment |
| **Confounder** | Variable influencing both treatment assignment and outcome |
| **Randomization** | Breaks systematic links between treatment and covariates |
| **Matching** | Creates treated/control groups similar on observed covariates |
| **Propensity score** | \(P(T=1\mid X)\) based on observed covariates |
| **Balance** | Similar covariate distributions in treated and control groups |
| **Natural experiment** | Treatment assignment is unusually haphazard due to external circumstances |
| **Sensitivity analysis** | Tests robustness to possible unobserved confounding |

---

# Key takeaways

1. **Association does not automatically imply causation.**
2. **Confounding occurs when treatment and outcome share common causes.**
3. **Randomized experiments are ideal because treatment assignment is random.**
4. **Observational studies are often necessary, but causal conclusions require stronger assumptions.**
5. **Matching aims to make treated and control subjects comparable on observed covariates.**
6. **Exact matching becomes difficult in high-dimensional data.**
7. **Propensity scores compress many observed covariates into treatment probability.**
8. **The goal of propensity-score matching is covariate balance, not prediction accuracy by itself.**
9. **Always verify balance after matching.**
10. **Matching cannot automatically remove hidden confounding.**
11. **Sensitivity analysis measures how strong hidden bias would need to be to overturn a result.**
12. **Natural experiments can provide especially credible causal evidence from observational settings.**
13. **Good causal analysis combines statistical machinery with scientific reasoning about the data-generating process.**

---

# Compact mental model

```text
OBSERVED ASSOCIATION
        ↓
Could treatment assignment be confounded?
        ↓
Randomization possible?
   yes ↙       ↘ no
     RCT       Observational data
                  ↓
           Match / balance on
          observed covariates
                  ↓
        Estimate treatment effect
                  ↓
      Could hidden covariates explain it?
                  ↓
          Sensitivity analysis
```

The central lesson of the lecture is:

> **With observational data, causal inference is fundamentally a design problem: create treated and control groups that are as comparable as possible, verify observed balance, and then ask how robust the conclusion is to differences you could not observe.**
