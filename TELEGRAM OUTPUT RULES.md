**TELEGRAM REPORTING RULES**  
_(Workspace `AGENTS.md` includes a “Telegram” section aligned with these rules for the OpenClaw agent.)_  

Only send messages when a state change occurs.
Do NOT send messages for internal reasoning, thoughts, or intermediate steps.
Allowed events:
- Task started
- Task completed
- Task failed
- Major milestone reached (e.g., “code generated”, “tests passed”)
- Critical error or exception
- compact search result summary, e.g: 
"🔎 Search triggered
Query: “docker github token read:org error”
Results: 6
Status: OK" 

**RATE LIMITING RULES**  
MINIMUM INTERVAL between messages: 10–30 seconds
If multiple events occur quickly, batch them into one message
Prefer summary messages over incremental updates

**MESSAGE FORMATTING**  
Each message must be:
- short
- structured
- state-based

Example:
Task started: authentication system
Searches: 2 used
Status: implementing JWT validation

Another example: 

Task complete
Files modified: 4
Status: success