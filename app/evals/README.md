# Voice Travel Agent - Evaluation System

This module provides comprehensive evaluation checks for the Voice Travel Agent's itineraries. It implements three critical evaluation types as required:

## Overview

The evaluation system consists of three main components:

1. **Feasibility Eval** - Validates itinerary practicality
2. **Edit Correctness Eval** - Ensures voice edits are accurate
3. **Grounding & Hallucination Eval** - Verifies information authenticity

## Installation

The evaluation modules are part of the main application. No additional installation is required beyond the main project dependencies.

## Module Structure

```
app/evals/
├── __init__.py              # Module exports
├── feasibility.py           # Feasibility evaluation
├── edit_correctness.py      # Edit correctness evaluation
├── grounding.py             # Grounding & hallucination evaluation
├── runner.py                # Evaluation orchestrator
├── example_usage.py         # Usage examples
└── README.md                # This file
```

---

## 1. Feasibility Evaluation

**Purpose**: Ensure the itinerary is realistic and achievable.

### Checks Performed

#### 1.1 Daily Duration ≤ Available Time
- Verifies activities fit within time windows (morning: 3h, afternoon: 3h, evening: 4h)
- Detects over-scheduling (too many activities in one time slot)
- Flags days with excessive activities (>10 activities/day)

#### 1.2 Reasonable Travel Times
- Validates travel time between consecutive activities
- Maximum recommended: 60 minutes between activities
- Requires location data or travel time estimates

#### 1.3 Pace Consistency
- Checks for balanced activity distribution
- Ideal: 6 or fewer activities per day
- Warns if all activities are in one time period
- Flags under-planned days (<2 activities)

### Usage Example

```python
from app.evals import EvaluationRunner

runner = EvaluationRunner()

# Run feasibility check
results = runner.run_feasibility_eval(
    itinerary_text=itinerary,
    travel_times={
        1: [25.0, 15.0],  # Day 1 travel times in minutes
        2: [20.0, 18.0],  # Day 2 travel times
    }
)

print(f"Passed: {results['passed']}")
print(f"Summary: {results['summary']}")
```

### Expected Output Format

```json
{
  "eval_type": "feasibility",
  "passed": true,
  "summary": {
    "total_days": 2,
    "total_checks": 6,
    "passed_checks": 6,
    "failed_checks": 0
  },
  "results": [
    {
      "day": 1,
      "check": "daily_duration",
      "passed": true,
      "total_activities": 3,
      "issues": []
    }
  ]
}
```

---

## 2. Edit Correctness Evaluation

**Purpose**: Ensure voice edits only modify intended sections without unintended changes.

### Checks Performed

#### 2.1 Voice Edits Only Modify Intended Sections
- Parses itinerary into sections (Day 1, Day 2, Travel Tips, etc.)
- Compares original and edited versions
- Identifies which sections changed

#### 2.2 No Unintended Changes Elsewhere
- Detects modifications outside the intended scope
- Uses similarity scoring (0.0 to 1.0)
- Flags unintended changes with context

### Usage Example

```python
from app.evals import EvaluationRunner

runner = EvaluationRunner()

results = runner.run_edit_correctness_eval(
    original_itinerary=original_text,
    edited_itinerary=edited_text,
    edit_instruction="Change Day 2 afternoon activity to museum visit"
)

print(f"Passed: {results['passed']}")
print(f"Unintended changes: {results['summary']['unintended_changes']}")
```

### Expected Output Format

```json
{
  "eval_type": "edit_correctness",
  "passed": true,
  "summary": {
    "total_changes": 1,
    "intended_changes": 1,
    "unintended_changes": 0
  },
  "differences": [
    {
      "section": "day_2",
      "similarity": 0.85,
      "change_type": "modification"
    }
  ],
  "issues": []
}
```

---

## 3. Grounding & Hallucination Evaluation

**Purpose**: Verify information is grounded in actual data sources and not hallucinated.

### Checks Performed

#### 3.1 POIs Map to Dataset Records
- Extracts POI names from itinerary
- Compares against search results (Foursquare, OpenStreetMap)
- Calculates grounding percentage
- Pass threshold: ≥70% of POIs grounded

#### 3.2 Tips Cite RAG Sources
- Checks for source citations in travel tips
- Expected format: `[Source: Wikivoyage - City - Section]`
- Pass threshold: ≥50% of tips have citations

#### 3.3 Uncertainty Explicitly Stated
- Detects uncertainty markers (may, might, could, uncertain, etc.)
- Ensures uncertainty is stated for:
  - Weather predictions
  - Search failures
  - Missing data

### Usage Example

```python
from app.evals import EvaluationRunner

runner = EvaluationRunner()

search_results = {
    "museum": [
        {"name": "Louvre Museum", "rating": 4.7},
        {"name": "Musée d'Orsay", "rating": 4.8}
    ],
    "restaurant": [
        {"name": "Le Jules Verne", "rating": 4.5}
    ]
}

results = runner.run_grounding_eval(
    itinerary_text=itinerary,
    search_results=search_results
)

print(f"Passed: {results['passed']}")
print(f"POI Grounding Rate: {results['results'][0]['grounding_percentage']:.1f}%")
```

### Expected Output Format

```json
{
  "eval_type": "grounding",
  "passed": true,
  "summary": {
    "total_checks": 3,
    "passed_checks": 3,
    "failed_checks": 0
  },
  "results": [
    {
      "check": "poi_grounding",
      "passed": true,
      "grounding_percentage": 85.0,
      "grounded_pois": ["Louvre Museum", "Eiffel Tower"],
      "ungrounded_pois": ["Some Random Place"]
    },
    {
      "check": "tip_citations",
      "passed": true,
      "total_tips": 3,
      "tips_with_citations": 2,
      "citation_percentage": 66.7
    },
    {
      "check": "uncertainty_markers",
      "passed": true,
      "total_uncertainty_markers": 1
    }
  ]
}
```

---

## Running All Evaluations

Use the `EvaluationRunner` to run all evaluations together:

```python
from app.evals import EvaluationRunner

runner = EvaluationRunner()

results = runner.run_all_evals(
    itinerary_text=itinerary,
    context={
        "search_results": search_results,
        "travel_times": travel_times,
        "original_itinerary": original_itinerary,  # Optional, for edit checks
        "edit_instruction": "Change Day 1 lunch"   # Optional
    }
)

# Generate human-readable report
report = runner.generate_report(results)
print(report)

# Save results to JSON
runner.save_results(results, "eval_results.json")
```

---

## Integration with Voice Agent

### During Itinerary Creation

```python
from app.evals import EvaluationRunner

# After agent creates itinerary
itinerary_text = agent_response["itinerary"]
search_results = agent_response["search_results"]

# Run evaluations
runner = EvaluationRunner()
eval_results = runner.run_all_evals(
    itinerary_text=itinerary_text,
    context={"search_results": search_results}
)

# Check if itinerary passes all checks
if not eval_results["overall"]["all_passed"]:
    logger.warning(f"Itinerary failed evaluation: {eval_results['overall']['all_issues']}")
    # Optionally regenerate or warn user
```

### During Voice Edits

```python
# Before edit
original_itinerary = current_itinerary

# After edit
edited_itinerary = agent_response["updated_itinerary"]

# Run edit correctness check
runner = EvaluationRunner()
edit_results = runner.run_edit_correctness_eval(
    original_itinerary=original_itinerary,
    edited_itinerary=edited_itinerary,
    edit_instruction=user_edit_request
)

if not edit_results["passed"]:
    logger.warning("Edit made unintended changes!")
```

---

## Example Output

### Console Output

```
============================================================
Starting Comprehensive Itinerary Evaluation
============================================================

[1/3] Running Feasibility Evaluation...
  Feasibility: ✓ PASSED
    - total_days: 2
    - total_checks: 6
    - passed_checks: 6

[2/3] Running Grounding & Hallucination Evaluation...
  Grounding: ✓ PASSED
    - total_checks: 3
    - passed_checks: 3

[3/3] Running Edit Correctness Evaluation...
  Edit Correctness: ✓ PASSED
    - total_changes: 1
    - unintended_changes: 0

============================================================
Evaluation Complete
Overall Status: ✓ PASSED
Total Evaluations: 3
Passed: 3
Failed: 0
Time Elapsed: 0.15s
============================================================
```

### Report Output

```
======================================================================
ITINERARY EVALUATION REPORT
======================================================================
Timestamp: 2024-02-15T10:30:45.123456

OVERALL RESULTS
----------------------------------------------------------------------
Status: ✓ PASSED
Pass Rate: 100.0%
Total Issues: 0

1. FEASIBILITY EVALUATION
----------------------------------------------------------------------
Status: ✓ PASSED
Total Days: 2
Checks Passed: 6/6

2. GROUNDING & HALLUCINATION EVALUATION
----------------------------------------------------------------------
Status: ✓ PASSED

Poi Grounding:
  - Grounded POIs: 5
  - Ungrounded POIs: 1
  - Grounding Rate: 83.3%

Tip Citations:
  - Total Tips: 3
  - Tips with Citations: 2
  - Citation Rate: 66.7%

Uncertainty Markers:
  - Uncertainty Markers Found: 1

3. EDIT CORRECTNESS EVALUATION
----------------------------------------------------------------------
Status: ✓ PASSED
Total Changes: 1
Intended Changes: 1
Unintended Changes: 0

======================================================================
END OF REPORT
======================================================================
```

---

## Testing

Run the example usage script:

```bash
cd app/evals
python example_usage.py
```

This will run all four examples:
1. Full evaluation
2. Feasibility only
3. Grounding only
4. Edit correctness

---

## Thresholds & Configuration

### Current Thresholds

| Check | Threshold | Configurable |
|-------|-----------|--------------|
| Max activities per day | 10 | Yes (FeasibilityEval.MAX_ACTIVITIES_PER_DAY) |
| Ideal activities per day | 6 | Yes (FeasibilityEval.IDEAL_ACTIVITIES_PER_DAY) |
| Max travel time | 60 min | Yes (FeasibilityEval.MAX_TRAVEL_TIME_MINUTES) |
| POI grounding rate | 70% | Yes (in check_poi_grounding) |
| Tip citation rate | 50% | Yes (in check_tip_citations) |

### Customizing Thresholds

```python
from app.evals import FeasibilityEval

# Customize thresholds
feasibility = FeasibilityEval()
feasibility.MAX_TRAVEL_TIME_MINUTES = 45  # Stricter travel time
feasibility.IDEAL_ACTIVITIES_PER_DAY = 5  # More relaxed pace

# Use in evaluation
results = feasibility.evaluate(itinerary_text)
```

---

## Logging

All evaluations log detailed information:

```python
import logging

# Enable detailed evaluation logging
logging.basicConfig(level=logging.INFO)

# Or configure specific logger
eval_logger = logging.getLogger("evals")
eval_logger.setLevel(logging.DEBUG)
```

---

## Future Enhancements

Potential improvements for the evaluation system:

1. **Real-time Travel Time Validation**
   - Integrate with OSRM API to calculate actual travel times
   - Validate against realistic driving/walking times

2. **Budget Feasibility**
   - Check if activities fit within user's budget
   - Validate cost estimates

3. **Weather Integration**
   - Check activity appropriateness for weather conditions
   - Suggest indoor alternatives for rain

4. **Accessibility Checks**
   - Validate wheelchair accessibility
   - Check for mobility-friendly routes

5. **Batch Evaluation**
   - Evaluate multiple itineraries
   - Compare and rank itineraries

6. **Automated Testing**
   - Unit tests for each evaluation component
   - Integration tests with sample itineraries

---

## Contributing

When adding new evaluation checks:

1. Create a new method in the appropriate eval class
2. Update the `evaluate()` method to include the new check
3. Add corresponding tests
4. Update this README with usage examples

---

## License

This evaluation system is part of the Voice Travel Agent project.
