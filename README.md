# JSL SmartGrade Engine
### Round 2 Prototype — Stainless Steel Grade Selector

> **An explainable, constraint-aware engineering decision-support system for stainless-steel grade selection.**

JSL SmartGrade Engine converts explicitly entered application and service requirements into a ranked shortlist of stainless-steel grades.

**User Requirements → Engineering Constraints → Multi-Criteria Decision Making → Recommendation → Explainable Trade-offs**

---

# 1. Problem

Selecting a stainless-steel grade is a multi-variable engineering decision.

A highly corrosion-resistant grade may be unnecessarily expensive, while a lower-cost grade may fail a critical requirement. A high-strength grade may also be technically capable but unnecessarily over-specified.

SmartGrade therefore asks:

> **Which feasible grade provides the best overall fit for the stated engineering requirements?**

The system considers:

- Corrosion environment
- Chloride exposure
- Acid exposure
- pH
- Operating temperature
- Required yield strength
- Welding/fabrication requirements
- Cost versus corrosion priority

---

# 2. End-to-End System Flow

```text
┌───────────────────────────────┐
│ APPLICATION & SERVICE INPUT   │
│ • Application                 │
│ • Chloride / Acid / pH        │
│ • Temperature                 │
│ • Yield strength              │
│ • Fabrication                 │
│ • Cost ↔ Corrosion priority   │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ INPUT VALIDATION              │
│ Required inputs explicitly    │
│ provided by the user          │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ HARD-CONSTRAINT SCREENING     │
│ Remove grades violating       │
│ mandatory requirements        │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ QUICK FIT RATING              │
│ Simple Cost ↔ Corrosion       │
│ user-facing priority          │
└───────────────┬───────────────┘
                ▼
       ┌─────────────────┐
       │ ADVANCED MODE?  │
       └────────┬────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
   ┌───────────┐    Recommendation
   │    AHP    │
   │ Weighting │
   │ + CR      │
   └─────┬─────┘
         ▼
   ┌───────────────┐
   │    TOPSIS     │
   │    Ranking    │
   └───────┬───────┘
           ▼
┌───────────────────────────────┐
│ RECOMMENDATION                │
│ Top grade + alternatives      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ WHY THIS GRADE?               │
│ Explain technical reasoning   │
│ and trade-offs                │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ TRADE-OFF + WHAT-IF ANALYSIS  │
│ Show ranking sensitivity      │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│ ENGINEERING REVIEW FLAG       │
│ Escalate unusual/critical     │
│ cases for expert review       │
└───────────────────────────────┘
```

---

# 3. Critical Input Philosophy

## Application ≠ Engineering Requirement

The application type is a **label/template only**. It must not silently change engineering requirements.

```text
Chemical Process Tank
        │
        ├── Does NOT automatically set chloride
        ├── Does NOT automatically set acid exposure
        ├── Does NOT automatically set pH
        ├── Does NOT automatically set temperature
        ├── Does NOT automatically set strength
        └── Does NOT automatically set fabrication
```

When the website first opens:

- No recommendation is shown
- Required engineering parameters are unselected
- pH is blank
- Chloride must be explicitly selected
- Acid exposure must be explicitly selected
- Temperature must be explicitly selected
- Required strength must be explicitly selected
- Welding/fabrication must be explicitly selected
- Calculation is unavailable until required inputs are provided

Changing **Chemical Process Tank** to **Marine / Coastal Structure** does not by itself change chloride, acid, pH, temperature, strength, fabrication, or priority.

Application templates may suggest starting conditions, but the user must explicitly review/select the engineering parameters.

---

# 4. Application & Service Requirements

| Parameter | Purpose |
|---|---|
| Application | Identifies the use case |
| Chloride exposure | Corrosion screening |
| Acid exposure | Chemical environment screening |
| pH | Environmental severity |
| Operating temperature | Temperature compatibility |
| Minimum yield strength | Mechanical requirement |
| Welding/fabrication | Fabrication compatibility |
| Cost ↔ Corrosion priority | Decision preference |

The recommendation is driven by the engineering inputs rather than the application name.

---

# 5. Quick Fit Rating

The default experience uses a simple priority control:

```text
Cost ───────────────●──────────── Corrosion Resistance
                    ↑
              User Priority
```

This keeps the primary workflow easy for fabricators, customers, and non-specialist users.

Quick Fit provides:

- Recommended grade
- Fit score
- Top-pick indication
- Short reasoning
- Leading alternatives

**Important:** the Cost ↔ Corrosion control is a user-facing priority interface, not the full AHP calculation. Detailed AHP is exposed in Advanced Engineering Mode.

---

# 6. Hard-Constraint Engineering Screening

Mandatory engineering constraints are checked before multi-criteria ranking.

```text
All Grades
    │
    ▼
Corrosion requirement?
    │
    ▼
Temperature compatible?
    │
    ▼
Strength requirement?
    │
    ▼
Fabrication compatible?
    │
    ▼
Feasible Grades
```

Possible screening factors:

- Corrosion / PREN threshold
- Temperature compatibility
- Minimum yield strength
- Fabrication/weldability requirement
- Other configured engineering constraints

The interface shows which grades passed and the exact reason rejected grades failed.

A grade that fails a mandatory constraint should **not** be allowed to win through TOPSIS.

---

# 7. Advanced Engineering Mode

Advanced Engineering Mode provides technical transparency for judges and engineers.

The five criteria are:

1. Corrosion
2. Temperature
3. Yield Strength
4. Weldability
5. Lifecycle Cost

```text
                 ┌──────────────┐
                 │  CORROSION   │
                 └──────┬───────┘
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│TEMPERATURE  │  │YIELD        │  │WELDABILITY  │
│             │  │STRENGTH     │  │             │
└─────────────┘  └─────────────┘  └─────────────┘
                        │
                        ▼
                ┌─────────────┐
                │ LIFECYCLE   │
                │    COST     │
                └─────────────┘
```

Advanced mode exposes:

- AHP pairwise comparisons
- Criterion weights
- λmax
- Consistency Index
- Consistency Ratio (CR)

AHP is intentionally hidden from the normal flow so technical depth does not reduce ease of use.

---

# 8. AHP Priority Weighting

AHP converts decision priorities into criterion weights.

For five criteria:

```text
Pairwise comparisons
= n(n−1)/2
= 5(5−1)/2
= 10 comparisons
```

```text
User Priorities
       │
       ▼
Pairwise Comparison Matrix
       │
       ▼
AHP Weight Calculation
       │
       ├── Criterion Weights
       │
       └── Consistency Check
              │
              ▼
             CR
```

The consistency diagnostic helps identify whether the priority judgments are reasonably consistent.

---

# 9. TOPSIS Ranking

After hard constraints are applied, feasible grades are ranked using TOPSIS in Advanced Engineering Mode.

```text
Feasible Grade Dataset
        │
        ▼
Decision Matrix
        │
        ▼
Normalization
        │
        ▼
Apply AHP Weights
        │
        ▼
Weighted Decision Matrix
        │
        ├───────────────┐
        ▼               ▼
   Ideal Best       Ideal Worst
        │               │
        └───────┬───────┘
                ▼
       Distance Calculation
                │
                ▼
        Relative Closeness
                │
                ▼
          Final Ranking
```

Calculation sequence:

1. Decision matrix
2. Normalization
3. AHP-weight application
4. Ideal best solution
5. Ideal worst solution
6. Distance from ideal solutions
7. Relative closeness
8. Final ranking

Displayed TOPSIS scores should come from the actual calculation, not manually assigned demonstration values.

---

# 10. Recommendation & Explainability

The final recommendation provides:

- Recommended grade
- Top alternatives
- Fit/ranking information
- Key decision drivers
- Engineering constraints satisfied
- Main trade-offs

The **Why This Grade?** layer connects the mathematical ranking to engineering reasoning.

```text
                  RECOMMENDATION
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      CORROSION      MECHANICAL     FABRICATION
          │              │              │
          ▼              ▼              ▼
     PREN / ENV.      YIELD ETC.    WELDABILITY
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                   COST / LIFECYCLE
                         │
                         ▼
                 FINAL EXPLANATION
```

The system explains:

- Why the grade satisfies the stated duty
- Which criteria drove the ranking
- Corrosion suitability
- Mechanical suitability
- Weldability/fabrication suitability
- Cost/lifecycle trade-offs

Alternative grades can receive dynamic **Why Not?** explanations.

---

# 11. Over-Specification Detection

A grade can pass every mandatory constraint while still providing substantially more capability than required.

SmartGrade can flag:

> **Potential Over-Specification**

```text
Required strength
       │
       ▼
   250 MPa
       │
       ▼
Candidate grade
       │
       ▼
Large performance margin
       │
       ▼
Potential Over-Specification
```

This is a screening indication, not a final engineering or commercial decision.

---

# 12. Trade-Off Analysis

Leading candidates can be compared using:

- Corrosion / pitting resistance
- Temperature margin
- Yield strength
- Weldability
- Lifecycle cost

```text
                 Corrosion
                    ▲
                    │
              ┌─────┼─────┐
              │     │     │
     Cost ◄───┼─────┼─────┼───► Strength
              │     │     │
              └─────┼─────┘
                    │
                    ▼
                Weldability
```

The radar/trade-off view supports technical demonstration and explainability.

---

# 13. What-If / Sensitivity Analysis

Recommendations are not intended to behave like fixed lookup-table answers.

```text
Input Requirement
       │
       ▼
Engineering Screening
       │
       ▼
Feasible Grades
       │
       ▼
AHP Weights
       │
       ▼
TOPSIS Scores
       │
       ▼
Final Ranking
```

Example:

```text
COST PRIORITY
      ↓
More weight on lifecycle cost
      ↓
Ranking A

        VS.

CORROSION PRIORITY
      ↓
More weight on corrosion resistance
      ↓
Ranking B
```

Possible What-If questions:

- What happens when corrosion becomes more important than cost?
- What happens when required strength increases?
- What happens when temperature increases?
- What happens when chloride exposure becomes more severe?

---

# 14. Engineering Review Flag

SmartGrade is a **decision-support prototype**, not a replacement for engineering approval.

Engineering review can be recommended for:

- Extreme or unusual conditions
- Critical applications
- Novel/exotic chemistry
- Missing critical information
- Potential SCC concerns
- Insufficient property data
- Ambiguous recommendations
- Highly sensitive rankings

```text
                    Recommendation
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
          Normal Case            Review Trigger
                │                     │
                ▼                     ▼
          User Output          Engineering Review
                                      │
                                      ▼
                              Final Engineering
                                  Decision
```

---

# 15. Grade / Property Data

The prototype dataset includes:

- Cr
- Ni
- Mo
- N
- PREN
- Yield strength
- Tensile strength
- Elongation
- Temperature-related screening information
- Weldability
- Relative cost index

The technical basis should distinguish:

```text
┌─────────────────────────────┐
│ PUBLIC / REFERENCE DATA     │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ ILLUSTRATIVE PROTOTYPE DATA │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ JSL-VALIDATED DATA          │
└─────────────────────────────┘
```

JSL-specific validation is required before industrial deployment.

---

# 16. PREN Screening

The prototype uses:

```text
PREN = %Cr + 3.3(%Mo) + 16(%N)
```

PREN is a screening indicator, not a complete corrosion-life prediction.

Actual corrosion performance also depends on:

- Service environment
- Chemistry
- Temperature
- Fabrication
- Surface condition
- Other application-specific factors

---

# 17. Cost Model

The prototype uses a **relative lifecycle/cost index** rather than live commercial pricing.

```text
Relative Cost
      +
Maintenance
      +
Downtime
      +
Lifecycle Considerations
      ↓
Relative Lifecycle Cost Index
```

Actual deployment should use validated JSL/customer-specific commercial and lifecycle data.

---

# 18. Data Transparency

The **Grade Data / Technical Evidence** area is intended to make the technical basis visible.

Before industrial deployment, property data and engineering thresholds should be validated against:

- JSL specifications
- Applicable standards
- Technical documentation
- Validated material data
- Expert engineering judgment

The goal is to make important recommendations traceable to an identifiable technical basis.

---

# 19. Validation Strategy

```text
┌─────────────────────────┐
│ Engineering Benchmark  │
│ Case                    │
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│ Reference / Expert      │
│ Recommendation          │
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│ SmartGrade              │
│ Recommendation          │
└────────────┬────────────┘
             ▼
       ┌───────────┐
       │  Compare  │
       └─────┬─────┘
             ▼
┌─────────────────────────┐
│ Investigate Mismatches  │
└─────────────────────────┘
```

Useful validation metrics:

- Feasible-grade screening agreement
- Top-1 recommendation agreement
- Top-3 overlap
- Constraint-violation rate
- Expert acceptance rate
- Recommendation turnaround time

**Do not claim an accuracy percentage unless it has actually been measured on a defined validation set.**

---

# 20. Recommended Demo Flow

```text
1. Select Application
          ↓
2. Enter Service Requirements
          ↓
3. Calculate Recommendation
          ↓
4. Show Quick Fit Rating
          ↓
5. Open Advanced Engineering Mode
          ↓
6. Show AHP Weights + CR
          ↓
7. Show Engineering Screening
          ↓
8. Show TOPSIS Ranking
          ↓
9. Explain "Why This Grade?"
          ↓
10. Explain Alternatives / Over-Specification
          ↓
11. Change One Requirement / Priority
          ↓
12. Recalculate and Show Ranking Change
```

This demonstrates both **ease of use** and **technical depth**.

---

# 21. Deployment Architecture

```text
                    ┌──────────────────────┐
                    │ JSL Grade / Property │
                    │      Database        │
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │   Validation Layer   │
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │ Engineering Rule     │
                    │ Engine                │
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │     AHP Engine       │
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │    TOPSIS Engine     │
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │ Explainability /     │
                    │ Reporting Layer      │
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │    Web Application   │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
           Sales          Distributors        OEMs
                                               │
                                               ▼
                                           Engineers
```

The architecture separates:

- Grade data
- Engineering rules
- Decision methodology
- Explainability
- User interface

This allows new grades, applications, criteria, and markets to be added without rebuilding the entire system.

---

# 22. Scalability Concept

```text
Single Grade Selector
        ↓
Multi-Application Selector
        ↓
Multi-Market Selector
        ↓
JSL Digital Engineering Platform
```

Potential extensions include:

- Additional stainless-steel grades
- Additional corrosion environments
- More detailed chemical-service inputs
- More application templates
- Customer-specific lifecycle cost
- Additional decision criteria
- Technical datasheet generation
- Engineering report generation
- Integration with JSL digital/sales channels

---

# 23. Prototype Limitations

- Reference/public data may not represent the complete JSL internal dataset.
- Screening thresholds require engineering validation.
- PREN is a screening metric, not a complete corrosion model.
- Relative cost indices are illustrative rather than live market prices.
- Application categories do not replace detailed service-condition characterization.
- Special/exotic environments require expert review.
- Actual lifecycle cost requires JSL/customer-specific data.
- The recommendation is decision support and does not replace engineering approval.

These limitations define the boundary between the **Round 2 prototype** and a potential production system.

---

# 24. Local Run

The prototype is a self-contained HTML/React application.

### Option 1 — Open directly

Open:

```text
index.html
```

### Option 2 — Local server

```bash
python3 -m http.server 8080
```

Then open:

```text
http://localhost:8080
```

If React/ReactDOM are loaded from a CDN, internet access is required when the page first loads.

---

# 25. Editing the Prototype

Key editable areas:

```text
Grade / Property Dataset
        │
        ▼
Engineering Screening Rules
        │
        ▼
AHP Criteria / Weighting
        │
        ▼
TOPSIS Calculation
        │
        ▼
Recommendation Logic
        │
        ▼
Explanation Layer
        │
        ▼
UI / Application Templates
```

Before changing engineering thresholds or property values, verify and document the technical source.

---

# 26. Round 2 Positioning

## JSL SmartGrade Engine

> **An explainable, constraint-aware engineering decision-support system that translates application requirements into a ranked stainless-steel grade shortlist while making technical and commercial trade-offs transparent.**

The prototype is designed to:

- Reduce repetitive first-level screening
- Improve consistency
- Surface engineering trade-offs
- Prevent unsuitable grades from winning the ranking
- Identify potential over-specification
- Explain recommendation reasoning
- Support What-If / sensitivity analysis
- Provide a structured starting point for engineering decisions

The long-term concept is not simply a **grade lookup tool**.

It is a scalable:

```text
                    ┌──────────────────┐
                    │ USER REQUIREMENT │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
                    │ ENGINEERING      │
                    │ SCREENING        │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
                    │ DECISION         │
                    │ OPTIMIZATION     │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
                    │ EXPLAINABLE      │
                    │ RECOMMENDATION   │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
                    │ ENGINEERING      │
                    │ REVIEW           │
                    └──────────────────┘
```

**SmartGrade turns stainless-steel grade selection from a static lookup exercise into a transparent, constraint-aware engineering decision workflow.**
