# Project Management Principles

## Team Structure
- Project Owner: Eu Hock, HenryHien
- Members: Tony, Binh, Minh, Tinh, Phuc, Thanh, ...

## Task Management Rules
- Work is managed in a **Task-based process** (not Scrum).
- **Project Owner** responsibilities:
  - Define overall Project Goals.
  - Create and clearly communicate Product Backlog Items (PBI).
  - Prioritize PBIs and set order of execution.
- **Members (Tony, Binh, Minh, Tinh, Phuc, Thanh, …)** responsibilities:
  - Split Product Backlog Items into executable **Tasks**.  
  - Ensure each Task is achievable within a reasonable timeline (≤ 1 week).  
  - Take ownership of assigned Tasks and complete them.  
  - Report progress daily and actively request reviews.  
  - Support other Members in completing tasks with proper quality and within deadlines.  
  - Other Members provide guidance, reviews, and coordination.  
  - Other Members facilitate task flow and resolve blockers.  
  - A Task is **not Done** until reviewed and approved.  

## Task Definition
- Size guideline:
  - Tiny: 4h
  - Small: 8h
  - Medium: 16h
  - Large: 24h

## Events
- **Task Planning [30min, Every (will defined)xx xx:xx AM]**
  - Project Owner 
    - Decide which PBIs will be converted into executable Tasks.
    - Estimate effort required.
    - Assign Tasks to members.
  - Members: 
    - Each members must confirm they can complete their Tasks within the agreed timeline.

- **Daily Check-in [10min, Every (will defined) xx:xx AM]**
  - Yesterday’s work progress.
  - Today’s plan.
  - Issues or blockers.
  - Review status updates.

- **Task Review [30min, Every (will defined)xx xx:xx AM]**
  - Confirm completed Tasks.
  - Identify uncompleted or delayed Tasks.
  - Record new PBIs or follow-up Tasks if needed.

## GitHub Workflow to Manage Backlog

### Create Product Backlog Item
- Create an Issue in the repository using the template **“Product Backlog Item”**.
  - **Title**: Deliverable name (e.g., “Basic Design Document”).
    - NOT a task name (e.g., “Build appliances”).
  - **Type**: “PBI” (auto assigned by template).
    - Status: “Proposal”.
    - Priority: [P1, P2, P3, P4].
    - Size: -  
    - Task: -
  - **Description**:
    - User Story
    - Deliverables
    - Notes

### Create Task [During Task Planning]
- Create an Issue in the repository using the template **Sprint Backlog Item**.
  - **Title**: Task name.
  - **Assignees**: One responsible person.
  - **Type**: “SBI” (auto assigned by template).
    - Status: “Proposal”.
    - Size: [Tiny, Small, Medium, Large].
    - Priority: Defined by Project Owner.
  - **Description**:
    - Steps / Goals.
    - Related PBI: “#0”.

### Task Workflow
1. **Proposal/Pending** → Create and estimate size. Define clear Definition of Done.  
2. **Todo** → Assigned to one responsible member.  
3. **In Progress** → Work started. Actively request review when ready.  
   - Update documents/code with PR and link PR in the Task issue.  
   - Do not close Task automatically with PR merge.  
   - If blocked, mark as **Pending**.  
4. **Review** → Reviewer assigned, status updated.  
5. **Done** → Review completed, final approval given.  
   - Report to Project Owner: completion, effort spent, and new PBIs if required.  
   - Task marked as completed.  
