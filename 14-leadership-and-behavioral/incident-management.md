# Incident Management & Post-Mortems

### 1. Overview
At the Lead/Staff level, you are often the person paged when the system is burning down. Incident Management revolves around how you act during the fire (mitigation) and how you act after the fire (prevention). It requires remaining calm, delegating tasks, and fostering a culture of continuous learning over finger-pointing.

### 2. Key Concepts
*   **Incident Commander (IC)**: The person in charge of managing the incident. They don't fix the bug; they coordinate communication, assign roles, and keep stakeholders updated.
*   **MTTD & MTTR**: 
    *   *Mean Time to Detect (MTTD)*: How long it took to notice the fire.
    *   *Mean Time to Resolve/Recover (MTTR)*: How long it took to put the fire out.
*   **Blameless Post-Mortem (COE - Correction of Errors)**: A psychological safety principle stating that systemic failures cause outages, not "bad engineers." Focus on *how* the system allowed the mistake, not *who* typed the wrong command.
*   **Runbooks / Playbooks**: Documented, step-by-step procedures to resolve specific known issues (e.g., "What to do if the Redis cache runs out of memory").

### 3. Real-World Usage
*   **Sev-1 Outage**: The payment gateway goes down during Black Friday. The Incident Commander opens a dedicated Slack channel, assigns an engineer to investigate logs, assigns another to prep a rollback, and updates the support team every 15 minutes.
*   **The 5 Whys**: During a post-mortem, asking "Why" 5 times sequentially to get to the root cause. (e.g., Database crashed -> Why? Disk full -> Why? Logs overwhelmed disk -> Why? Debug mode was left on -> Why? Lack of staging-to-prod environment checks).
*   **Action Items (AIs)**: The most critical output of a post-mortem. E.g., "JIRA-123: Add automated test to prevent debug mode in production deployment pipeline."

### 4. Tradeoffs
*   **Mitigation vs. Investigation**: During an active incident, the primary goal is *Mitigation* (stopping the bleeding, even if it means rolling back or restarting servers), NOT finding the exact line of code that caused it (which can take days).
*   **Alert Fatigue**: Sending a PagerDuty alert to the whole team every time CPU spikes to 80% causes engineers to ignore alerts. Save pages for user-impacting symptoms.

### 5. When NOT to Use
*   **Blaming Individuals**: Writing a post-mortem that concludes "Bob deployed bad code. Action Item: Tell Bob to be more careful." This guarantees Bob will try to hide his next mistake rather than report it.
*   **Post-Mortems for Everything**: Only trigger full formal post-mortems for high-severity incidents (Sev-1 or Sev-2). Requiring them for Sev-4 (minor internal glitches) slows the team down.

### 6. Interview Focus
*   **The Outage Story**: "Walk me through the worst production outage you were involved in. What did you do, and what did the team learn?" (Hint: Talk about mitigation first, then the post-mortem process).
*   **Systemic Fixes**: "An engineer drops a production database table manually. How do you prevent this?" (Hint: Don't say 'revoke their access.' Say 'require CI/CD pipelines for all schema changes').
*   **MTTR Improvement**: "Our site takes 2 hours to recover from an outage. What strategies do you implement to lower MTTR?"

### 7. Common Mistakes
*   **Tunnel Vision**: An engineer jumping into the logs and ignoring Slack/Communicate for 40 minutes while the CEO is demanding an update.
*   **The "Human Error" Root Cause**: Declaring "Human Error" as the root cause. If human error can take down your system, your system lacks safety nets (automation, guardrails, approvals).
*   **Unfinished Action Items**: Writing a brilliant post-mortem with 5 Action Items, but burying the tickets in the backlog so the exact same outage happens 6 months later.
