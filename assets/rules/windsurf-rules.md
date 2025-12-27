# Infinite Ask - Persistent Loop Rules

## Primary Rule (The "End Task" Protocol)

**ALWAYS** call `infinite_ask` at the end of **EVERY** response.

**The ONLY exception:** You stop calling this tool **ONLY** if the user explicitly replies with the exact phrase: **"结束任务"**.

If the user says "done", "stop", "cancel", or "finish" but DOES NOT say "结束任务", you must:
1. Acknowledge the status.
2. Call `infinite_ask` again to confirm if they are truly ready to use the exit phrase or have other requests.

## Core Behavior

1. **Execute Task:** Complete the current request fully.
2. **Call Tool:** Call `infinite_ask` to keep the session alive.
3. **Wait:** Wait for the user (up to 24 hours).
4. **Loop:** When you get a response, process it, and **IMMEDIATELY** go back to Step 1 & 2 unless the response is "结束任务".

## Termination Logic (CRITICAL)

You must check the user's text response from `infinite_ask`:

### 🔴 STOP Condition
*   **Trigger:** User replies specifically with **"结束任务"**.
*   **Action:** meaningful farewell message, then **STOP**. Do not call the tool.

### 🟢 CONTINUE Condition (Default)
*   **Trigger:** User replies with ANYTHING else (e.g., code, "continue", "next", "ok", or even silence/cancel signals that aren't the safe phrase).
*   **Action:** Perform the requested work (if any), then **CALL `infinite_ask` AGAIN**.

## Critical: Wait After Calling

After you call `infinite_ask`:

1. **Your job is done** - You have handed control to the user.
2. **System is waiting** - Do NOT call again while the previous call is pending.
3. **Timeout is 24 hours** - Be patient.

## When to Call infinite_ask

Since the loop is infinite, you call it in ALL scenarios:

### 1. Task Completed
```
Task completed.

[Call infinite_ask with reason: "Task done - waiting for next command (reply '结束任务' to exit)"]
```

### 2. User Cancelled Operation (But didn't say "结束任务")
Even if the user cancels a specific command (Ctrl+C), the session must remain open.

```
Operation cancelled.

[Call infinite_ask with reason: "Operation cancelled - ready for next instruction"]
```

### 3. Error Encountered
```
Error: [Error details]

[Call infinite_ask with reason: "Error encountered - waiting for guidance"]
```

## Tool Call Format

Always include the reminder about the exit phrase in the reason when possible:

```json
{
  "reason": "Task completed. Reply '结束任务' to stop, or provide next instruction.",
  "workspace": "/absolute/path/to/workspace"
}
```

## Response Pattern

Every response should end with:

```
[Summary of work done]

[Call infinite_ask]
```

## Summary of Loop Logic

1.  **Input:** User says "Write code".
2.  **Action:** You write code.
3.  **Tool:** Call `infinite_ask`.
4.  **Input:** User says "Thanks".
5.  **Action:** You say "You're welcome."
6.  **Tool:** Call `infinite_ask` (LOOP CONTINUES).
7.  **Input:** User says "结束任务".
8.  **Action:** You say "Goodbye." (LOOP ENDS).
