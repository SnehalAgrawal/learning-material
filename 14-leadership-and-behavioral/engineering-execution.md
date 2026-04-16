# Engineering Execution & ADRs

### 1. Overview
As you move into Lead/Staff roles, you stop being just a "ticket taker" and become a "project driver." Engineering Execution is about how software gets built safely, predictably, and at scale. It focuses on writing technical strategy documents and making decisions that impact multiple teams for years to come.

### 2. Key Concepts
*   **Architecture Decision Records (ADRs)**: A short text file capturing an architectural decision, its context, and the consequences. It prevents the "Why did we do this?" question 2 years later.
*   **RFC (Request for Comments) / Design Docs**: A formal document written *before* coding starts, distributed to other seniors to gather critique on a proposed architecture.
*   **Tech Debt Management**: Categorizing debt (accidental vs. deliberate) and negotiating with Product to assign 10-20% of every sprint to paying it down.
*   **Buy vs. Build**: The framework for deciding whether to use a SaaS product (Auth0, Datadog) or build it in-house.

### 3. Real-World Usage
*   **Migrating a Database**: Writing a 10-page Design Doc (RFC) detailing how you will move from MongoDB to PostgreSQL with zero downtime (Dual writes -> Dark reads -> Cutover).
*   **Choosing a Framework**: Writing an ADR deciding to use React instead of Vue for a new internal dashboard, listing the tradeoff that "Vue is simpler, but our entire company is already trained in React."
*   **Project Slicing**: Taking a massive 6-month product requirement and slicing it into 1-month iterative deliverables so the business gets value sooner.

### 4. Tradeoffs
*   **Analysis Paralysis vs. Rushing**: Spending 4 weeks writing an RFC for a tiny feature is a waste of time. Writing zero code-level documentation for a massive feature ensures it will fail.
*   **Building vs. Buying**: Building it yourself gives you 100% control and lower variable cost, but limits your engineering velocity. Buying is expensive but allows your engineers to focus on your core business value.
*   **Perfect Architecture vs. Time to Market**: The best architecture is the one that survives the current phase of the company. Finding the "Good Enough" architecture is a core Staff skill.

### 5. When NOT to Use
*   **Heavy Process for Startups**: If you are a 3-person startup trying to find Product-Market fit, writing exhaustive ADRs is a waste of time. Agility > Documentation in Phase 1.
*   **Technical Dictatorships**: An RFC is a *Request for Comment*, not a mandate. Don't use it to force a pet technology on a team without addressing their feedback.

### 6. Interview Focus
*   **Decision Making**: "Walk me through a technical decision you made where there was no obvious 'right' answer. How did you decide?" (Hint: Talk about evaluating tradeoffs and ADRs).
*   **Influencing**: "How do you convince non-technical leadership that we need to stop building features for a month to rewrite the billing system?" (Hint: Tie tech debt to business metrics like delayed delivery or churn).
*   **Project Rescue**: "You join a project that is 3 months behind schedule. What do you do in your first week?"

### 7. Common Mistakes
*   **Ignoring the 'Consequences' in ADRs**: An ADR needs to clearly state what the downside of a decision is. Every decision has a downside.
*   **The "Rewrite it in Rust" Syndrome**: Proposing a massive architectural rewrite just because a technology is "new" or "cool," without a clear ROI (Return on Investment) for the business.
*   **Failing to Sunk Cost**: Refusing to abandon a 3-month project that is clearly failing, instead throwing more engineers at it. (The "Mythical Man-Month" fallacy).
