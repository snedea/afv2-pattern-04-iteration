# AFv2 Pattern #4: Iteration - Test Cases

## Overview

**Pattern:** Quality-driven iterative refinement loop with convergence detection
**Flow:** Start → Planner → Gate → Research → [loop back to Gate] → Report → Direct Reply
**Repository:** https://github.com/snedea/afv2-pattern-04-iteration

---

## Prerequisites

### 1. Import Pattern into Flowise

1. Open Flowise UI (http://localhost:3000)
2. Navigate to **Agentflows** section
3. Click **"Add New"** → **"Import Agentflow"**
4. Upload `04-iteration.json`
5. Pattern should load with nodes:
   - Start
   - Agent.Planner
   - Quality Gate (ConditionAgent)
   - Agent.Research
   - Agent.Report
   - Direct Reply
   - Sticky Notes (documentation)

### 2. Configure API Keys

**All agents require Anthropic API key configuration:**

1. Configure model for all agents:
   - **Credential:** Select your "Anthropic API Key" credential
   - **Model Name:** `claude-sonnet-4-5-20250929`
   - **Temperature:** `0.2` (for deterministic quality scoring)
   - **Streaming:** `true`

2. Save the workflow

### 3. Verify Node Configuration

**Expected Configuration:**

| Node | Type | State Updates | Key Feature |
|------|------|---------------|-------------|
| Agent.Planner | Agent | `iteration.plan`, `iteration.current=0` | Initial plan creation |
| Quality Gate | ConditionAgent | Reads `iteration.quality_score` | Scoring system (0.0-1.0) |
| Agent.Research | Agent | `iteration.content`, `iteration.quality_score`, `iteration.current++` | Iterative refinement |
| Agent.Report | Agent | `output` | Final report generation |
| Direct Reply | DirectReply | N/A | Terminal node |

**Quality Gate Configuration:**
- **Target Score:** 0.85 (85% quality threshold)
- **Max Iterations:** 3
- **Paths:** CONVERGED (≥0.85) | CONTINUE (<0.85, iterations<3) | MAX_ITERATIONS (iterations≥3)

---

## Test Cases

### TC-4.1: Early Convergence (Pass on First Iteration)

**Objective:** Verify loop exits early when quality threshold met immediately

**Input:**
```
Write a comprehensive analysis of renewable energy adoption in Europe, including market trends, policy drivers, and economic impacts.
```

**Expected Execution Flow:**

1. **Agent.Planner Executes:**
   - Creates initial research plan
   - Sets iteration counter to 0
   - Updates state:
     ```json
     {
       "iteration.plan": "[research plan]",
       "iteration.current": 0
     }
     ```

2. **Quality Gate (First Check):**
   - Iteration 0, no quality score yet
   - Decision: CONTINUE → proceed to Research

3. **Agent.Research Executes (Iteration 1):**
   - Performs comprehensive research
   - Produces high-quality content immediately
   - Scores content quality: 0.90 (exceeds 0.85 threshold)
   - Updates state:
     ```json
     {
       "iteration.content": "[comprehensive research results]",
       "iteration.quality_score": 0.90,
       "iteration.current": 1
     }
     ```

4. **Quality Gate (Second Check):**
   - Reads `iteration.quality_score = 0.90`
   - Condition: 0.90 ≥ 0.85 → **CONVERGED**
   - Decision: Exit loop → Route to Report

5. **Agent.Report Executes:**
   - Generates final report
   - Includes iteration count (1) and quality score (0.90)
   - Updates state: `output` with final report

6. **Direct Reply Returns:**
   - Displays completion message

**Validation Checklist:**

- [ ] **Planner Execution:**
  - State contains `iteration.plan`
  - State contains `iteration.current=0`

- [ ] **First Quality Gate:**
  - Routed to CONTINUE (no score yet)
  - Research agent executed

- [ ] **Research Execution (Iteration 1):**
  - Produced comprehensive content
  - Quality score ≥ 0.85 (e.g., 0.90)
  - State contains `iteration.current=1`

- [ ] **Second Quality Gate:**
  - Read `iteration.quality_score=0.90`
  - Detected convergence (0.90 ≥ 0.85)
  - Routed to CONVERGED path (NOT CONTINUE)

- [ ] **Loop Exit:**
  - Research agent did NOT execute again
  - **CRITICAL:** Only 1 iteration performed (not 2 or 3)
  - Loop exited early on convergence

- [ ] **Report Generation:**
  - Report includes iteration count (1)
  - Report includes quality score (0.90)
  - Report indicates early convergence

**Success Criteria:**
- Quality threshold (0.85) met on first iteration
- Loop exited immediately (no unnecessary iterations)
- Final iteration count = 1
- Report indicates convergence achieved

**Expected Output:**
```
✅ CORRECT:
Iteration count: 1
Quality score: 0.90
Status: Converged (threshold 0.85 met)

❌ INCORRECT:
Iteration count: 3
Quality score: 0.90
[Loop continued despite meeting threshold]
```

---

### TC-4.2: Progressive Convergence (3 Iterations to Target)

**Objective:** Verify iterative refinement improves quality over multiple iterations

**Input:**
```
Analyze the impact of artificial intelligence on healthcare diagnostics, focusing on accuracy improvements and adoption barriers.
```

**Expected Execution Flow:**

1. **Agent.Planner → Quality Gate → Agent.Research (Iteration 1):**
   - Research produces initial content
   - Quality score: 0.60 (below threshold)
   - Updates state: `iteration.content`, `iteration.quality_score=0.60`, `iteration.current=1`

2. **Quality Gate (After Iteration 1):**
   - Reads `iteration.quality_score=0.60`
   - Condition: 0.60 < 0.85 AND iterations < 3 → **CONTINUE**
   - Decision: Loop back to Research for refinement

3. **Agent.Research (Iteration 2):**
   - Reads previous content from `iteration.content`
   - Identifies gaps and refines
   - Improved quality score: 0.75 (improved but still below threshold)
   - Updates state: `iteration.quality_score=0.75`, `iteration.current=2`

4. **Quality Gate (After Iteration 2):**
   - Reads `iteration.quality_score=0.75`
   - Condition: 0.75 < 0.85 AND iterations < 3 → **CONTINUE**
   - Decision: Loop back again

5. **Agent.Research (Iteration 3):**
   - Further refinement based on previous iterations
   - Final quality score: 0.88 (exceeds threshold)
   - Updates state: `iteration.quality_score=0.88`, `iteration.current=3`

6. **Quality Gate (After Iteration 3):**
   - Reads `iteration.quality_score=0.88`
   - Condition: 0.88 ≥ 0.85 → **CONVERGED**
   - Decision: Exit loop → Route to Report

7. **Agent.Report → Direct Reply**

**Validation Checklist:**

- [ ] **Iteration 1:**
  - Quality score = 0.60 (below threshold)
  - Gate decision = CONTINUE
  - Research executed again

- [ ] **Iteration 2:**
  - Content refined (different from Iteration 1)
  - Quality score = 0.75 (improved, but still below threshold)
  - Gate decision = CONTINUE
  - Research executed again

- [ ] **Iteration 3:**
  - Content further refined
  - Quality score = 0.88 (exceeds threshold)
  - Gate decision = CONVERGED
  - Research did NOT execute again

- [ ] **Quality Progression:**
  - Score improved each iteration: 0.60 → 0.75 → 0.88
  - Content became more comprehensive each iteration
  - Refinements addressed gaps from previous iterations

- [ ] **Loop Behavior:**
  - Gate correctly routed to CONTINUE for iterations 1-2
  - Gate correctly routed to CONVERGED for iteration 3
  - Loop exited after convergence (not at max iterations)

- [ ] **Report:**
  - Iteration count = 3
  - Quality score = 0.88
  - Status = Converged

**Success Criteria:**
- Quality improved progressively (0.60 → 0.75 → 0.88)
- Loop executed exactly 3 times
- Loop exited on convergence (not max iterations)
- Each iteration refined previous content

**Quality Progression Validation:**
```javascript
const iterations = [
  { iteration: 1, score: 0.60, action: "CONTINUE" },
  { iteration: 2, score: 0.75, action: "CONTINUE" },
  { iteration: 3, score: 0.88, action: "CONVERGED" }
];

// Validation rules
iterations.forEach((iter, idx) => {
  if (idx > 0) {
    assert(iter.score > iterations[idx-1].score); // ✅ Quality improves
  }
  if (iter.score >= 0.85) {
    assert(iter.action === "CONVERGED"); // ✅ Exit on convergence
  }
});
```

---

### TC-4.3: Max Iterations Without Convergence

**Objective:** Verify loop terminates at max iterations (3) even if quality threshold not met

**Input:**
```
Provide a brief overview of quantum computing applications.
```

**Expected Execution Flow:**

1. **Agent.Research (Iteration 1):**
   - Produces content
   - Quality score: 0.55 (below threshold)
   - Updates state: `iteration.current=1`, `iteration.quality_score=0.55`

2. **Quality Gate:** CONTINUE (0.55 < 0.85, iterations < 3)

3. **Agent.Research (Iteration 2):**
   - Attempts refinement
   - Quality score: 0.65 (improved slightly, still below threshold)
   - Updates state: `iteration.current=2`, `iteration.quality_score=0.65`

4. **Quality Gate:** CONTINUE (0.65 < 0.85, iterations < 3)

5. **Agent.Research (Iteration 3):**
   - Final refinement attempt
   - Quality score: 0.72 (improved, but still below 0.85 threshold)
   - Updates state: `iteration.current=3`, `iteration.quality_score=0.72`

6. **Quality Gate (After Iteration 3):**
   - Reads `iteration.quality_score=0.72` (below threshold)
   - Reads `iteration.current=3` (max iterations reached)
   - Condition: iterations ≥ 3 → **MAX_ITERATIONS**
   - Decision: Exit loop → Route to Report (despite not converging)

7. **Agent.Report:**
   - Generates report with best available content
   - Includes warning: "Max iterations reached without full convergence"
   - Report shows: iteration count=3, quality score=0.72, status=Max iterations

8. **Direct Reply Returns**

**Validation Checklist:**

- [ ] **Iteration Execution:**
  - Exactly 3 iterations executed
  - Research did NOT execute a 4th time
  - **CRITICAL:** Loop terminated at 3 iterations (not infinite)

- [ ] **Quality Scores:**
  - Iteration 1: 0.55 (below threshold)
  - Iteration 2: 0.65 (below threshold)
  - Iteration 3: 0.72 (below threshold)
  - None reached 0.85 threshold

- [ ] **Gate Decisions:**
  - After iteration 1: CONTINUE
  - After iteration 2: CONTINUE
  - After iteration 3: MAX_ITERATIONS (NOT CONTINUE)

- [ ] **Max Iterations Exit:**
  - Gate detected `iteration.current=3`
  - Routed to MAX_ITERATIONS path
  - Report executed despite low quality score

- [ ] **Report Content:**
  - Iteration count = 3
  - Quality score = 0.72 (final score, below threshold)
  - Status = "Max iterations reached"
  - Warning/note indicates convergence not achieved

- [ ] **No Infinite Loop:**
  - Loop terminated after exactly 3 iterations
  - No 4th iteration attempted
  - Workflow completed successfully

**Success Criteria:**
- Loop executed exactly 3 times (max limit enforced)
- Loop exited at max iterations despite not converging
- Report includes status "Max iterations reached"
- No infinite loop (CRITICAL safety check)

**Max Iterations Safety Validation:**
```
Expected Behavior:

Iteration 1: score=0.55 < 0.85, iterations=1 < 3 → CONTINUE ✅
Iteration 2: score=0.65 < 0.85, iterations=2 < 3 → CONTINUE ✅
Iteration 3: score=0.72 < 0.85, iterations=3 ≥ 3 → MAX_ITERATIONS ✅
[STOP - No iteration 4]

✅ PASS: Loop terminated at 3 iterations
❌ FAIL: Iteration 4 executed (infinite loop bug!)
```

---

## Common Issues & Debugging

### Issue 1: Infinite Loop (Research Continues Beyond 3 Iterations)

**Symptoms:**
- Research agent executes 4, 5, or more times
- Loop never exits
- Workflow appears stuck

**Debugging Steps:**
1. Verify Quality Gate has MAX_ITERATIONS condition:
   - Should check `iteration.current ≥ 3`
   - Should have separate output anchor for MAX_ITERATIONS
   - MAX_ITERATIONS should route to Report (not Research)

2. Check iteration counter increment:
   - Agent.Research should increment `iteration.current` by 1
   - Counter should persist across iterations
   - Verify state update: `iteration.current = {{ iteration.current + 1 }}`

3. Verify edge connections:
   ```json
   // Gate should have 3 outputs:
   { "source": "quality_gate", "sourceHandle": "output-0", "target": "agent_report" },      // CONVERGED
   { "source": "quality_gate", "sourceHandle": "output-1", "target": "agent_research" },    // CONTINUE
   { "source": "quality_gate", "sourceHandle": "output-2", "target": "agent_report" }       // MAX_ITERATIONS
   ```

**Solution:**
- Add MAX_ITERATIONS condition to Quality Gate
- Ensure Research increments `iteration.current` each time
- Verify MAX_ITERATIONS routes to Report (not back to Research)

---

### Issue 2: Loop Exits Immediately (No Iterations)

**Symptoms:**
- Research executes once, then immediately exits
- Quality score is low but loop doesn't continue
- Only 1 iteration performed

**Debugging Steps:**
1. Check Quality Gate CONTINUE condition:
   - Should route to CONTINUE when `quality_score < 0.85 AND iterations < 3`
   - Should NOT require quality score on first iteration

2. Verify edge from Quality Gate to Research:
   - CONTINUE output (output-1) should connect to Agent.Research
   - Check loop-back edge exists

3. Verify Quality Gate reads state correctly:
   - Should read `{{ iteration.quality_score }}` variable
   - Should read `{{ iteration.current }}` variable

**Solution:**
- Configure CONTINUE condition correctly
- Ensure loop-back edge connects Gate → Research
- Verify Quality Gate reads state variables

---

### Issue 3: Quality Score Not Improving Across Iterations

**Symptoms:**
- Quality score stays constant (e.g., 0.60 → 0.60 → 0.60)
- No refinement between iterations
- Content appears identical across iterations

**Debugging Steps:**
1. Verify Agent.Research reads previous content:
   - Should access `{{ iteration.content }}` from state
   - Should use previous content as context for refinement

2. Check Research prompt includes refinement instructions:
   - Prompt should instruct agent to improve previous content
   - Should identify gaps and address them

3. Verify memory enabled:
   ```json
   {
     "agentEnableMemory": true  // ✅ Required
   }
   ```

**Solution:**
- Enable memory for Agent.Research
- Update prompt with explicit refinement instructions
- Ensure Research reads previous iteration content from state

---

### Issue 4: Quality Gate Always Routes to CONVERGED

**Symptoms:**
- Loop exits after 1 iteration regardless of quality score
- Low-quality content (0.50) incorrectly marked as converged

**Debugging Steps:**
1. Check Quality Gate threshold:
   - Should be 0.85 (not 0.50 or 0.00)
   - CONVERGED condition: `quality_score ≥ 0.85`

2. Verify Quality Gate reads correct state variable:
   - Should read `{{ iteration.quality_score }}`
   - Variable name must match Research state update

3. Check scenario configuration:
   - Scenario 0: CONVERGED (score ≥ 0.85)
   - Scenario 1: CONTINUE (score < 0.85, iterations < 3)
   - Scenario 2: MAX_ITERATIONS (iterations ≥ 3)

**Solution:**
- Set convergence threshold to 0.85
- Verify state variable names match
- Check scenario conditions in Quality Gate config

---

### Issue 5: Report Missing Iteration Metrics

**Symptoms:**
- Final report doesn't show iteration count
- No quality score displayed
- Missing convergence status

**Debugging Steps:**
1. Verify Agent.Report reads state:
   - Should access `{{ iteration.current }}`
   - Should access `{{ iteration.quality_score }}`
   - Should access `{{ iteration.content }}`

2. Check Report prompt includes metrics:
   - Prompt should instruct to include iteration count
   - Should include quality score
   - Should indicate convergence status

3. Verify memory enabled for Report agent

**Solution:**
- Update Report prompt to include iteration metrics
- Ensure Report reads state variables
- Enable memory for Report agent

---

## Test Execution Report

**Test Date:** `[YYYY-MM-DD]`
**Flowise Version:** `[version]`
**Pattern Version:** `1.0`

### Test Results Summary

| Test Case | Status | Iterations | Final Score | Notes |
|-----------|--------|-----------|-------------|-------|
| TC-4.1: Early Convergence | ⬜ Pass / ⬜ Fail | `[1]` | `[0.90]` | |
| TC-4.2: Progressive Convergence | ⬜ Pass / ⬜ Fail | `[3]` | `[0.88]` | |
| TC-4.3: Max Iterations | ⬜ Pass / ⬜ Fail | `[3]` | `[0.72]` | |

### Configuration Details

- **Anthropic Model:** `claude-sonnet-4-5-20250929`
- **Temperature:** `0.2`
- **Quality Threshold:** `0.85`
- **Max Iterations:** `3`
- **Streaming:** `Enabled`
- **Memory Enabled:** `Yes`

### Iteration Performance Metrics

| Test Case | Iter 1 Score | Iter 2 Score | Iter 3 Score | Exit Reason |
|-----------|-------------|-------------|-------------|-------------|
| TC-4.1 | 0.90 | N/A | N/A | Converged (early) |
| TC-4.2 | 0.60 | 0.75 | 0.88 | Converged |
| TC-4.3 | 0.55 | 0.65 | 0.72 | Max iterations |

### Issues Encountered

`[List any issues encountered during testing]`

### Additional Notes

`[Any additional observations or recommendations]`

---

## Pattern Validation

This pattern has been validated against `validate_workflow.py` with the following checks:

✅ **Structural Validation:**
- Proper JSON structure
- All required node and edge fields present

✅ **Node Type Validation:**
- Start node present
- 1 Planner agent
- 1 Quality Gate (ConditionAgent) with 3 output anchors
- 1 Research agent (loop body)
- 1 Report agent
- 1 DirectReply terminal node
- Sticky Notes for documentation

✅ **Loop Logic:**
- Loop-back edge exists (Research → Quality Gate)
- Max iterations enforced (≤ 3)
- Convergence detection (score ≥ 0.85)
- MAX_ITERATIONS safety exit

✅ **State Management:**
- All agents have memory enabled
- Iteration counter (`iteration.current`) increments correctly
- Quality score tracked across iterations
- Content preserved for refinement

✅ **Quality Gate Configuration:**
- 3 scenarios: CONVERGED | CONTINUE | MAX_ITERATIONS
- Threshold: 0.85 (85% quality target)
- Max iterations: 3
- Loop-back edge connects to Research

✅ **Safety Validation:**
- No infinite loop possible (max 3 iterations enforced)
- Graceful exit when max reached
- Report generated even without convergence

---

## Additional Resources

- **Pattern Library:** [Context Foundry AFv2 Patterns](https://github.com/snedea/afv2-patterns-index)
- **Pattern #4 Repository:** https://github.com/snedea/afv2-pattern-04-iteration
- **Flowise Documentation:** https://docs.flowiseai.com/
- **AgentFlow v2 Specification:** [AFv2 Docs](https://docs.flowiseai.com/agentflows)

---

🤖 Built with Context Foundry
