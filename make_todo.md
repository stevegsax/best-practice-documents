# Prompt: Convert Specification to Detailed TODO List

You are a software architect tasked with converting a technical specification into a detailed, actionable TODO list. The TODO list must be complete and ordered such that another LLM can follow it step-by-step to implement the entire specification.

## Input

You will receive a technical specification document describing a feature or system to be built.

## Pre-Analysis Requirements

Before creating tasks, verify you have:

- [ ] Complete specification document with numbered sections
- [ ] List of all existing project files and their current state
- [ ] Project structure/architecture diagram or description
- [ ] Technology stack and version requirements
- [ ] Understanding of external dependencies and integrations

## Output Requirements

Create a TODO list that:

1. Can be executed in order without circular dependencies
2. References specific sections of the specification for implementation details
3. Includes explicit file paths and function/class names
4. Contains concrete acceptance criteria for each task
5. Groups related tasks into logical milestones

## Instructions

### Step 1: Analyze the Specification

First, identify and list:

- All new files that need to be created
- All existing files that need to be modified
- All configuration changes required
- All external dependencies (libraries, services, APIs)

### Step 2: Create Dependency Matrix

Create a comprehensive dependency analysis:

1. Build a dependency table:

| Task ID | Task Name | Depends On | Can Start After | Estimated Hours | Can Parallelize With |
|---------|-----------|------------|-----------------|-----------------|---------------------|
| 1.1     | Create models | None | Immediately | 2 | 1.2, 1.3 |
| 1.2     | Create config | None | Immediately | 3 | 1.1, 1.3 |
| 2.1     | Core logic | 1.1, 1.2 | 1.2 complete | 5 | None |

2. Identify:

    - **Critical Path**: Longest sequence of dependent tasks
    - **Parallel Opportunities**: Tasks with no interdependencies
    - **Blocking Tasks**: Tasks that prevent multiple others from starting
    - **Optional Tasks**: Tasks that can be deferred without blocking core functionality

### Step 3: Generate Tasks

#### Task Granularity Rules

- **Maximum task size**: 8 hours of implementation work
- **Minimum task size**: 30 minutes of implementation work
- **Each task must**: Produce a verifiable deliverable
- **Split tasks if they**:

    - Modify more than 3 files
    - Require more than 100 lines of new code
    - Have multiple unrelated objectives
    - Cannot be tested independently

#### File State Tracking

For each file mentioned in a task, specify:

- **State Before**: `[New | Exists at <path> | Not present]`
- **State After**: `[Created at <path> | Modified | Deleted | Moved to <path>]`
- **Lock Status**: Whether other tasks can modify this file concurrently

For each file or component, create tasks following this pattern:

**For new files:**
```
- [ ] Create `path/to/filename.py` with class/function stubs
  - Spec reference: Section X.Y
  - Create: ClassName with methods: method1(), method2()
  - Dependencies: None
  - Acceptance: 
    - Verification: `python -m py_compile path/to/filename.py` succeeds
    - Expected: File contains all specified classes/functions with correct signatures
    - Metrics: File parses without syntax errors, imports work
    - Failure: Syntax errors, missing required components, incorrect signatures
```

**For modifications:**
```
- [ ] Add feature_name to `path/to/existing.py`
  - Spec reference: Section X.Y
  - Add: new_method() to ExistingClass
  - Dependencies: [previous task ID]
  - Acceptance: 
    - Verification: `mypy path/to/existing.py` and unit tests pass
    - Expected: New method integrates with existing code, type hints present
    - Metrics: No mypy errors, all existing tests still pass
    - Failure: Type errors, breaks existing functionality, missing type hints
```

**For configurations:**
```
- [ ] Add config_section to configuration schema
  - Spec reference: Section X.Y showing config format
  - Add: Fields x, y, z with types and defaults
  - Dependencies: Config model exists
  - Acceptance: 
    - Verification: Run config validation script with spec examples
    - Expected: All spec example configs load and validate successfully
    - Metrics: 100% of spec examples pass validation
    - Failure: Any spec example fails to validate or load
```

### Step 4: Add Supporting Tasks

For each component, include:

**Testing tasks:**
```
- [ ] Write unit tests for ComponentName
  - Test: Happy path for method1()
  - Test: Error cases for method2()
  - Test: Edge cases from spec section X.Y
  - Acceptance: All tests pass, coverage > 90%
```

**Error handling tasks:**
```
- [ ] Add error handling for feature_name
  - Add: CustomException class
  - Handle: Specific error cases from spec
  - Acceptance: Errors caught and logged properly
```

**Integration tasks:**
```
- [ ] Wire up component_a to component_b
  - Connect: Output of A to input of B
  - Add: Error handling for connection failures
  - Acceptance: Data flows correctly between components
```

### Step 5: Structure the Output

Organize tasks into this format:

```markdown
# TODO: [Project/Feature Name]

## Prerequisites

- [ ] Install library_name==version
- [ ] Set up SERVICE_NAME with credentials
- [ ] Ensure Python >= X.Y

## Milestone 1: Foundation
*Goal: Basic data structures and configuration*

### Task 1.1: Create Data Models

- [ ] Create `src/models.py` with data structures

    - Spec reference: Section 2.1 "Data Models"
    - Create: ModelA, ModelB classes from spec examples
    - Dependencies: None
    - Acceptance: Models instantiate with spec examples

### Task 1.2: Update Configuration

- [ ] Add new_feature section to `src/config.py`

    - Spec reference: Section 2.2 "Configuration Format"
    - Add: Configuration class with validation
    - Dependencies: Task 1.1 (uses ModelA)
    - Acceptance: Config loads spec example correctly

## Milestone 2: Core Implementation
*Goal: Main feature functionality*

### Task 2.1: Create Manager Class

- [ ] Create `src/manager.py` with ManagerClass

    - Spec reference: Section 3 "Implementation Details"
    - Methods: process(), validate(), execute()
    - Dependencies: Tasks 1.1, 1.2
    - Acceptance: Manager processes spec examples

[Continue with all tasks...]

## Milestone N: Documentation and Polish
*Goal: User-ready with docs and examples*

### Task N.1: Update Documentation

- [ ] Add usage examples to README

    - Examples: All examples from spec section 4
    - Dependencies: All implementation complete
    - Acceptance: User can follow examples successfully
```

### Task ID Format

Use hierarchical IDs: `[Milestone].[Task].[Subtask]`

- Milestone: Major phase (1, 2, 3...)
- Task: Task within milestone (1, 2, 3...)
- Subtask: Optional for complex tasks (a, b, c...)

Examples: `1.1`, `2.3`, `3.2.a`

### Milestone Structure

Each milestone must include:

- **Entry Criteria**: Specific conditions that must be met to begin
- **Exit Criteria**: Measurable conditions that indicate completion
- **Deliverables**: Concrete outputs (files, features, documentation)
- **Validation**: How to verify the milestone is truly complete
- **Rollback Plan**: Steps to undo changes if milestone fails

### Critical Requirements

1. **No forward references**: A task can only depend on tasks that appear earlier
2. **Explicit paths**: Always use full file paths from project root
3. **Concrete names**: Use actual class/function names, not placeholders
4. **Spec references**: Every task must reference the specification section it implements
5. **Testable criteria**: Each task must have specific acceptance criteria
6. **Atomic tasks**: Each task should be completable in one work session
7. **Independent verification**: Another developer should be able to verify completion
8. **Error handling**: Each task must specify how errors are handled
9. **State preservation**: Tasks must not leave the system in a broken state
10. **Documentation**: Tasks modifying public APIs must include doc updates

### Example Task (Complete)

```
### Task 2.3: Add API endpoint for feature_x

- [ ] Add `/api/feature_x` endpoint to `src/api/routes.py`

    - Spec reference: Section 3.2 "API Endpoints"
    - Add: post_feature_x() function
    - Parameters: data: FeatureXRequest (from models.py)
    - Returns: FeatureXResponse
    - Dependencies: Task 1.1 (models), Task 2.1 (manager)
    - Implementation:
        1. Import FeatureXManager from src/manager.py
        2. Validate request using FeatureXRequest model
        3. Call manager.process(data)
        4. Return FeatureXResponse
    - Acceptance: 
        - Endpoint responds at /api/feature_x
        - Accepts JSON matching spec example
        - Returns expected response format
        - Handles errors with 400/500 status codes
```

### Additional Task Templates

**Database Migration:**
```
- [ ] Create migration script for feature_name

    - Spec reference: Section X.Y "Data Schema"
    - Create: migrations/001_add_feature_name.sql
    - Operations: CREATE TABLE, ALTER TABLE, INDEX creation
    - Rollback: Include DOWN migration
    - Dependencies: Database schema documentation updated
    - Acceptance: Migration runs forward and backward successfully
```

**API Client Integration:**
```
- [ ] Integrate external_service API client

    - Spec reference: Section X.Y "External Services"
    - Add: Client configuration and authentication
    - Implement: Error handling, retry logic, timeout handling
    - Dependencies: API credentials available in environment
    - Acceptance: Successfully connects and handles all error cases
```

**Performance Optimization:**
```
- [ ] Optimize component_name performance

    - Spec reference: Performance requirements in Section X.Y
    - Current baseline: X seconds for operation Y
    - Target: < Z seconds for operation Y
    - Methods: Caching, query optimization, algorithm improvement
    - Acceptance: Benchmarks show target performance achieved
```

## TODO List Validation

Before finalizing the TODO list, verify:

1. **Completeness Checks:**

    - [ ] Every section of the specification is referenced at least once
    - [ ] All user-facing features have corresponding implementation tasks
    - [ ] All implementation tasks have corresponding test tasks
    - [ ] All public APIs have documentation tasks

2. **Dependency Validation:**

    - [ ] No circular dependencies exist (A→B→C→A)
    - [ ] All task dependencies reference valid task IDs
    - [ ] Critical path is clearly identified
    - [ ] Parallel execution opportunities are marked

3. **Quality Checks:**

    - [ ] Each task has clear, measurable acceptance criteria
    - [ ] File paths are absolute and consistent
    - [ ] Task sizes follow the granularity rules
    - [ ] Every created file is used by at least one other task

4. **Coverage Verification:**

    - [ ] Unit tests cover all new functions/methods
    - [ ] Integration tests cover all component interactions
    - [ ] Error cases from spec have corresponding tests
    - [ ] Documentation covers all public interfaces

## Your Task

Given the specification provided, create a complete TODO list following the format above. Ensure that:

1. Every line of code that needs to be written has a corresponding task
2. Tasks are ordered so dependencies are satisfied
3. The specification is fully implemented when all tasks are complete
4. Another LLM could follow this TODO list without seeing the original spec (though they can reference it)
5. The TODO list passes all validation checks listed above
6. Task completion can be verified through automated means where possible