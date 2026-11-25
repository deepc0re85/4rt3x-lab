# Evidence

Stills from the recorded run that support the findings in [`../REPORT.md`](../REPORT.md). Each image is captioned with what it shows and the report section it backs.

These frames come from the author's own isolated lab, against a device the author owns, running an already-patched vulnerability (CVE-2022-40684, public since 2022 with widely available PoCs). 


## screenshots/

| File | Shows | Backs |
|---|---|---|
| `chat_kickoff.png` | The single opening prompt: one target address and the high-level goal, then the run launching. | §1, §3 |
| `chat_diagnostics.png` | The system detecting the target technology and FortiOS version during recon. | §5 (step 1) |
| `exploration_path.png` | The exploration-path graph: how the run branched from recon to the known weakness. | §4, §5 |
| `board_1080s.png` | The broadcast board (shared knowledge graph) with typed nodes: goals, intents, facts, hints. The agents' common memory and channel. | §4 |
| `findings.png` | The four findings, all rated critical, forming one chain to administrative takeover. | §6 |
| `task_components.png` | The task broken into components the agents work against. | §4 |
| `test_assets.png` | Assets the run registered and operated on. | §4, §5 |
| `tool_executions.png` | The tool-usage record: which capabilities were actually exercised through the tracked layer. | §7 |
| `traffic_request.png` | A recorded request/response entry from the run's traffic view. | §5 |
| `fortigui_640s.png` | The end state: the FortiGate administrative interface reached on the target. | §6 (result) |

## screenshots/zoom/

Close-ups for legibility.

| File | Shows | Backs |
|---|---|---|
| `exp_legend.png` | The graph legend (node types and their meaning). | §4 |
| `exp_center_flow.png` | Center of the exploration graph: the main flow between nodes. | §4, §5 |
| `exp_hints_left.png` | The hints/steering column on the board. | §4 |
| `exp_vulns_right.png` | The identified-weakness branch of the graph. | §5 |
| `tools_table.png` | Close-up of the tool-usage table. | §7 |
| `traffic_list.png` | Close-up of the traffic list. | §5 |
| `traffic_request_detail.png` | Close-up of a single traffic entry. | §5 |
| `port80_http.png` | The HTTP service (port 80) as the run saw it. | §5 (recon) |
| `port22_ssh.png` | The SSH service (port 22) as the run saw it. | §5 (recon) |
