# Consistency Models & Conflict Resolution

### 1. Overview
Consistency models define the contract between a data store and its clients regarding the visibility of updates. In a distributed system, there is a fundamental spectrum from "Strict" (Safe but slow) to "Eventual" (Fast but fuzzy). Senior engineers must choose the model that fits the specific business risk.

### 2. Key Concepts
*   **Strong Consistency**: A read is guaranteed to return the most recent write. (Linearizability).
*   **Eventual Consistency**: If no new updates are made, all reads will eventually return the same value. No guarantee on "when."
*   **Causal Consistency**: Ensures that operations that are causally related are seen in the same order by all nodes.
*   **Monotonic Reads**: Once a client has read a value, they will never see an older version of that value.
*   **Conflict Resolution**: Strategies for when two nodes update the same data simultaneously (Last-Writer-Wins, Merkle Trees, CRDTs).

### 3. Real-World Usage
*   **E-Commerce Inventory**: **Strong Consistency** is required. You can't sell the last item to two different people.
*   **Social Feed/Likes**: **Eventual Consistency** is perfectly fine. It doesn't matter if one user sees 100 likes and another sees 102 for a few seconds.
*   **Collaborative Editing (Google Docs)**: Uses **Operational Transformation (OT)** or **CRDTs** (Conflict-free Replicated Data Types) to resolve simultaneous typing into a consistent final document.
*   **Global DNS**: A classic example of **Eventual Consistency**. When you change an IP, it takes hours to propagate across the globe.

### 4. Tradeoffs
*   **User Experience vs. System Complexity**: Strong consistency feels better to users but requires complex locking or consensus that can cause the app to feel "sluggish."
*   **Last-Writer-Wins (LWW)**: Very simple conflict resolution but results in "Lost Updates" where the slower user's changes are simply deleted.
*   **CRDTs**: Mathematically elegant and ensure no data loss, but can be memory-intensive as they must store the history of changes.

### 5. When NOT to Use
*   **Strong Consistency**: Avoid for non-user-facing analytics or background logging. The performance penalty is not worth it.
*   **Eventual Consistency**: Do NOT use for financial ledgers, transactional logs, or security permission updates (where a delay in revoking access could be a vulnerability).

### 6. Interview Focus
*   **The Scenario Test**: "You are building a real-time multiplayer leaderboard. Which consistency model do you pick and why?"
*   **Conflict Scenarios**: "Two users update their 'Bio' at the exact same millisecond on different nodes. How does your database handle this?"
*   **Read-Your-Writes**: "How do you implement 'Read-Your-Writes' consistency in a system with asynchronous read replicas?" (Hint: Session stickiness or Version tracking).

### 7. Common Mistakes
*   **Assuming SQL = Strong**: Many SQL databases default to "Read Committed" isolation, which is NOT linearizable. You must explicitly set "Serializable" if you need strong guarantees.
*   **The "Eventually Consistent" Trap**: Using eventual consistency but expecting the system to behave like it's strong, leading to race conditions in business logic.
*   **Blindly Choosing LWW**: Using "Last-Writer-Wins" for critical data like account balances, where two small withdrawals at once could result in only one being recorded.
