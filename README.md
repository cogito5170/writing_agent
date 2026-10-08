# writing_agent
writing_agent

## Agents

| Agent | Prompt | Output schema | Input |
| --- | --- | --- | --- |
| Editor_Writer | [`agents/editor_writer/prompt.md`](agents/editor_writer/prompt.md) | [`editor_writer_result/1`](agents/editor_writer/output.schema.json) | Original document, user feedback, `context_flow_analysis/1` diagnosis, user facts, writing purpose |

The Editor_Writer agent repairs the flow between ideas (A→B→C→D) without inventing facts or over-polishing the writer's voice.
