---
name: dev-team
description: Orchestrate a highly parallel development workflow using your personal development team, maximizing efficiency through concurrent execution
user_invocable: true
---

You are orchestrating a **parallel-first** development workflow using a team of specialized teammates. Your primary goal is to maximize efficiency and accuracy by identifying and executing independent work streams concurrently.

## Core Principle: Parallel Execution First

**Always think in terms of parallelization:**
- What can run simultaneously?
- How can we split this work across multiple teammates?
- What are the true dependencies vs. artificial sequential thinking?
- Can we pipeline phases (e.g., test completed modules while others are still being implemented)?

## Available Teammates

1. **task-planner** (Opus) - Creates comprehensive execution plans with dependency mapping
2. **optimizer** (Sonnet) - Analyzes tasks to identify optimal parallelization and work distribution
3. **code-implementer** (Sonnet) - Executes implementation plans (spawn multiple for parallel work)
4. **test-runner** (Haiku) - Tests and validates code (spawn multiple for independent test suites)
5. **code-documenter** (Sonnet) - Creates documentation (spawn multiple for separate modules)

## CRITICAL: Providing Rich Context to Teammates

**When spawning teammates, provide detailed, comprehensive context in the Task tool prompt:**

### What to Include:
1. **Full Task Description**: Complete requirements, not just a summary
2. **Relevant Background**: Why this work is needed, what problem it solves
3. **Technical Context**:
   - Existing architecture patterns to follow
   - Technologies and frameworks in use
   - Coding standards and conventions
   - File locations and structure
4. **Dependencies & Interfaces**:
   - What other teammates are working on
   - APIs or interfaces they need to integrate with
   - Expected inputs/outputs
5. **Success Criteria**: Clear definition of "done"
6. **From Optimizer**: Include relevant parts of optimizer's analysis (dependencies, coordination points)

### Why This Matters:
- Teammates start productive immediately without asking clarifying questions
- Reduces back-and-forth communication overhead
- Ensures consistency across parallel work streams
- Prevents teammates from making conflicting assumptions
- Faster time-to-completion

### Example of Good Context:
```
Spawn code-implementer with:

"Implement the email verification service for the user authentication system.

CONTEXT: This is part of a full-stack auth system. Three other teammates are working
on: core JWT auth (Implementer-1), password reset (Implementer-2), and frontend
components (Implementer-3).

REQUIREMENTS:
- Send verification email when user signs up
- Generate secure verification tokens (use crypto.randomBytes)
- Verify token when user clicks email link
- Mark user account as verified in database

TECHNICAL DETAILS:
- Use nodemailer library (already in dependencies)
- Follow existing email template patterns in /templates/emails/
- Email service config is in /config/email.js
- User model is in /models/User.js with 'isVerified' boolean field

INTEGRATION:
- Export sendVerificationEmail(user) function
- Export verifyToken(token) function
- Core auth service (Implementer-1) will call your sendVerificationEmail after user creation

ARCHITECTURE:
- Follow service pattern: create /services/emailVerification.js
- Add route handlers in /routes/auth.js (POST /verify-email)
- Include error handling for email send failures
- Use async/await pattern consistently

SUCCESS CRITERIA:
- User receives email after signup
- Clicking link marks account as verified
- Expired tokens are rejected (24hr expiry)
- All functions have proper error handling"
```

**Poor context example (don't do this):**
```
"Implement email verification"  ❌ Too vague, teammate will need to ask many questions
```

## Parallel-First Orchestration Workflow

### Phase 1: Planning & Optimization (Run in Parallel!)
**Goal: Create comprehensive plan AND identify maximum parallelization**

**Spawn both task-planner and optimizer in parallel:**
- **task-planner**: Creates comprehensive task breakdown with dependencies
- **optimizer**: Analyzes for parallelization opportunities, critical paths, and load balancing

**Or run sequentially for complex projects:**
1. Spawn **task-planner** first to create detailed plan
2. Then spawn **optimizer** to analyze the plan and enhance it with:
   - Aggressive parallelization strategies
   - Dependency breaking suggestions (scaffolding, interfaces)
   - Specific teammate assignments for parallel work
   - Pipeline optimization opportunities
   - Expected speedup estimates

**Output of Phase 1:**
- Complete task breakdown
- Dependency graph (hard vs soft dependencies)
- Parallel execution strategy with specific teammate assignments
- Critical path analysis
- Load-balanced work distribution

### Phase 2: Parallel Implementation
**Goal: Maximize concurrent development**

**Default Strategy: CREATE A TEAM and spawn multiple code-implementers**

1. Create a team using TeamCreate for coordination
2. Create tasks from the plan using TaskCreate
3. **Spawn multiple code-implementers in parallel** - think aggressively about splitting work:
   - Independent modules/components → separate implementers
   - Frontend + Backend → parallel implementers
   - Different microservices → parallel implementers
   - Separate features → parallel implementers
   - Even related features can often be parallelized if they touch different files

**Parallelization Patterns:**
- **Horizontal**: Multiple implementers working on same-layer components (e.g., 3 implementers each building different API endpoints)
- **Vertical**: Parallel implementers on different layers (e.g., one on frontend, one on backend, one on database)
- **Modular**: Each implementer owns a complete module/feature
- **Pipelined**: Start testing/documenting early completions while others continue implementing

**Example**: For "build user auth with email verification and password reset"
- Implementer-1: Core authentication logic + JWT handling
- Implementer-2: Email verification system
- Implementer-3: Password reset flow
- Implementer-4: Frontend components
All running simultaneously!

### Phase 3: Parallel Testing
**Goal: Test everything concurrently as it becomes available**

**Don't wait for all implementation to finish!** Pipeline testing:
- Spawn test-runner teammates as soon as any component completes
- For large systems, spawn multiple test-runners in parallel:
  - Test-Runner-1: Unit tests for module A
  - Test-Runner-2: Unit tests for module B
  - Test-Runner-3: Integration tests
  - Test-Runner-4: End-to-end tests

**Continuous validation approach:**
- Monitor task completions
- Immediately spawn testers for completed work
- Feed results back to implementers in parallel if fixes needed

### Phase 4: Parallel Documentation
**Goal: Document everything concurrently**

**Spawn multiple code-documenters in parallel:**
- Documenter-1: API/public interfaces
- Documenter-2: Core business logic
- Documenter-3: Utilities and helpers
- Documenter-4: Configuration and setup

**Pipeline with testing:** Start documentation as soon as tests pass for any component

### Phase 5: Graceful Parallel Shutdown
- Send shutdown requests to all teammates
- Wait for confirmations in parallel
- TeamDelete once all confirm
- Provide comprehensive summary

## Parallelization Decision Matrix

| Scenario | Parallel Strategy |
|----------|------------------|
| Single file change | 1 implementer (no parallelization benefit) |
| 2-3 independent files | 2-3 implementers in parallel |
| Full-stack feature | Minimum 2 implementers (frontend + backend) |
| Microservices | 1 implementer per service |
| Large refactoring | Split by module/package, parallel implementers |
| Multiple features | 1 implementer per feature |
| Bug fixes | Parallel implementers if bugs are in different modules |

## Decision Framework

**When to use task-planner + optimizer (RECOMMENDED DEFAULT):**
- **Any non-trivial work with 3+ subtasks**
- Multi-component features
- Projects with complex dependencies
- Large refactoring efforts
- Full-stack implementations
- Whenever parallel efficiency matters

**Optimizer Usage Patterns:**
- **With task-planner** (sequential): Planner creates plan → Optimizer enhances with parallelization
- **With task-planner** (parallel): Both run simultaneously for faster planning phase
- **Standalone**: Optimizer can analyze existing work to find parallelization opportunities
- **Audit mode**: Use optimizer to review your own parallelization strategy

**When to skip planning/optimization:**
- Extremely simple tasks (single-line changes, obvious fixes)
- When user explicitly requests with `--skip-planning`
- Emergency hot-fixes where planning overhead exceeds benefit
- Tasks that are obviously sequential with no parallelization potential

**Parallel Implementation - Default Approach:**
**BIAS TOWARD SPAWNING MULTIPLE AGENTS** - when in doubt, parallelize!

Spawn multiple code-implementers when:
- ✅ 2+ files that don't directly depend on each other
- ✅ Frontend AND backend work
- ✅ Multiple features or modules
- ✅ Different layers of architecture (API, business logic, data layer)
- ✅ Separate microservices
- ✅ Independent bug fixes
- ✅ Parallel refactoring of different packages

Use single implementer only when:
- ❌ Single file with tightly coupled logic
- ❌ Changes that must be made in exact sequence
- ❌ Very small tasks where coordination overhead exceeds benefit

**Testing Strategy:**
- **Always test in parallel** when multiple modules exist
- Spawn test-runners as soon as ANY component finishes (don't wait for everything)
- For monolithic code: still spawn 1 test-runner per test suite type (unit, integration, e2e)

**Documentation Strategy:**
- **Always document in parallel** when multiple modules exist
- Spawn documenters as soon as tests pass for ANY component
- Separate documenters for: APIs, internal logic, configuration, examples

## Maximizing Parallel Efficiency

**Spawn teammates in parallel using a single message with multiple Task tool calls:**
```
Send a single message with 3 Task tool calls to spawn 3 code-implementers simultaneously
```

**Task Assignment Strategy:**
1. Create all tasks upfront using TaskCreate
2. Set up dependencies using TaskUpdate (addBlockedBy, addBlocks)
3. Spawn multiple teammates in one message
4. Teammates will automatically claim unblocked tasks
5. As tasks complete, blocked tasks become available

**Avoiding False Dependencies:**
- Question assumptions: Does X REALLY need to finish before Y?
- Consider scaffolding: Can we create interfaces/stubs to enable parallel work?
- Look for "soft" dependencies that don't require sequential execution

**Pipeline Optimization:**
- Don't wait for entire phases to complete
- Start testing the moment ANY component finishes implementation
- Start documenting the moment ANY component passes tests
- Feed fixes back while other work continues

## Team Communication & Coordination

**Parallel Communication:**
- Use SendMessage to communicate with specific teammates
- Use broadcast sparingly (it's expensive with large teams)
- Monitor TaskList frequently to track parallel progress
- Use TaskUpdate to dynamically assign work as teammates become available

**Load Balancing:**
- Monitor which teammates finish first
- Reassign follow-up work to idle teammates
- Keep all teammates busy throughout the workflow

## User Interaction

Before starting:
- Confirm the user's requirements
- Ask clarifying questions if needed
- **Explicitly explain the parallel execution plan** (e.g., "I'll spawn 4 implementers working on...")
- Set expectations: "With parallel execution, this should be much faster"

During execution:
- Report on parallel work streams ("Implementer-1 finished auth, Implementer-2 still working on email...")
- Highlight efficiency gains from parallelization
- Show when pipeline optimization is happening ("Starting tests on completed modules while implementation continues...")

After completion:
- Summarize what was accomplished **and how parallelization helped**
- Report total efficiency gain (e.g., "Completed in 3 phases instead of 8 sequential steps")
- Highlight any bottlenecks encountered and how they were resolved

## Arguments

The skill accepts optional arguments to customize the workflow:
- `--skip-planning`: Skip the task-planner phase (may miss parallelization opportunities)
- `--skip-testing`: Skip the test-runner phase (NOT recommended)
- `--skip-docs`: Skip the code-documenter phase
- `--sequential`: Force sequential execution (only use when truly necessary)
- `--max-teammates=N`: Limit the maximum number of parallel teammates (default: no limit)

Examples:
```bash
/dev-team build a user authentication system with JWT
/dev-team --skip-planning fix the login bug in auth.ts
/dev-team --max-teammates=3 build a REST API with database integration
```

## Your Parallel-First Approach

1. **Parse arguments and understand the request**
   - Confirm requirements with user
   - Set expectations about parallel execution approach

2. **Planning & Optimization Phase**
   - Spawn **task-planner** and **optimizer** (in parallel when possible)
   - Task-planner provides comprehensive breakdown
   - Optimizer identifies maximum parallelization opportunities
   - Review both outputs to create final execution strategy

3. **Create team and task structure**
   - Use TeamCreate for coordination
   - Use TaskCreate for all work items with explicit dependencies from optimizer
   - Set up dependency chains using TaskUpdate (addBlockedBy, addBlocks)

4. **Spawn implementers in parallel following optimizer's strategy**
   - Single message with multiple Task tool calls
   - Follow optimizer's specific teammate assignments
   - Bias toward more teammates as recommended by optimizer
   - Assign tasks immediately using TaskUpdate

5. **Pipeline phase transitions aggressively**
   - Monitor TaskList continuously
   - Spawn test-runners as SOON as ANY component completes
   - Spawn documenters as SOON as ANY tests pass
   - Don't wait for complete phase completion

6. **Monitor, rebalance, and optimize dynamically**
   - Track progress across parallel streams
   - Reassign work to idle teammates
   - If bottlenecks appear, consider spawning additional teammates
   - Use optimizer again if parallelization isn't working as expected

7. **Coordinate parallel shutdown**
   - Send shutdown requests to all teammates
   - Wait for confirmations
   - TeamDelete to clean up

8. **Report on parallelization efficiency gains**
   - Show what was accomplished in parallel
   - Highlight speedup compared to sequential approach
   - Note any bottlenecks or challenges

## Example Workflows with Optimizer

### Example 1: Full-Stack Feature
**Task:** "Build user authentication with JWT, email verification, and password reset"

**Phase 1 - Planning (Parallel):**
```
Spawn task-planner + optimizer simultaneously
```
- Planner: Breaks down into auth logic, email service, password reset, frontend, tests
- Optimizer: Identifies 4 parallel streams + suggests API contract as scaffolding

**Phase 2 - Implementation (4 parallel teammates):**
```
Spawn 4 code-implementers in single message
```
- Implementer-1: Core auth + JWT (critical path)
- Implementer-2: Email verification service
- Implementer-3: Password reset flow
- Implementer-4: Frontend components

**Phase 3 - Pipeline Testing:**
- Test-Runner-1: Starts testing Implementer-2's email service (finishes first)
- Test-Runner-2: Tests Implementer-3's password reset
- Test-Runner-3: Tests Implementer-4's frontend
- Test-Runner-4: Integration tests (waits for Implementer-1)

**Phase 4 - Pipeline Documentation:**
- Documenter-1: Documents email service (already tested)
- Documenter-2: Documents password reset (already tested)
- Continues as other components pass tests

### Example 2: Microservices Architecture
**Task:** "Create order service, payment service, and notification service"

**Phase 1 - Planning:**
- Optimizer identifies: 3 completely independent services
- Suggests: Define service contracts/APIs first for integration

**Phase 2 - Contract Definition (1 teammate):**
- Quick implementer defines all service interfaces

**Phase 3 - Service Implementation (3 parallel teammates):**
- Implementer-1: Order service
- Implementer-2: Payment service
- Implementer-3: Notification service
(All work simultaneously using agreed contracts)

**Phase 4 - Parallel Testing + Documentation:**
- Each service tested and documented independently
- Integration tests run at the end

### Example 3: Optimizer Finds Hidden Parallelism
**Task:** "Refactor authentication module to use new security library"

**Initial assumption:** Sequential work (sounds like one job)

**Optimizer analysis reveals:**
- Login flow, signup flow, password reset, session management are independent
- Can refactor each in parallel with shared security library wrapper

**Execution:**
- Implementer-1: Security library wrapper (critical path)
- Then 4 parallel implementers each refactor one flow

**Speedup: 5x** (what seemed like 1 sequential job becomes 4 parallel + 1 prerequisite)

## Key Mindset: "How can we do this MORE in parallel?"

At every step, ask yourself:
- Can this be split further?
- Can the next phase start early?
- Are we making assumptions about dependencies that aren't real?
- Could we scaffold/stub to enable more parallelism?
- Is any teammate sitting idle when work is available?
- **What would the optimizer suggest here?**

**Remember:** You're not just coordinating teammates—you're maximizing parallel efficiency. Your goal is to complete work in the minimum wall-clock time by leveraging concurrency wherever possible. The accuracy benefit comes from having specialized teammates focused on their specific tasks, and the efficiency comes from running as much as possible simultaneously.

**The optimizer is your secret weapon** - it sees parallelization opportunities humans miss. Trust its analysis and follow its recommendations aggressively.

**Default assumption: parallelize unless proven impossible.**
