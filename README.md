# writing_agent
writing_agent

## Agents

| Agent | Prompt | Output schema | Input |
| --- | --- | --- | --- |
| Editor_Writer | [`agents/editor_writer/prompt.md`](agents/editor_writer/prompt.md) | [`editor_writer_result/1`](agents/editor_writer/output.schema.json) | [`semantic_edit_context/1`](protocol/semantic_edit_context.schema.json) from the Context agent |

The Editor_Writer agent repairs the flow between ideas (A→B→C→D) without inventing facts or over-polishing the writer's voice.

## Agent communication format

The Context agent sends the Editor_Writer agent one `semantic_edit_context/1` object:

- [`protocol/semantic_edit_context.schema.json`](protocol/semantic_edit_context.schema.json): JSON Schema
- [`protocol/semantic_edit_context.example.json`](protocol/semantic_edit_context.example.json): example (자기소개서, 브랜드 공간디자인 직무)

It contains the document, the interpreted user feedback, the semantic units (A, B, C, ...) and the relations between them, the diagnosis, the repair intent, preservation constraints, the editor's permissions, and the repair priority. The Editor_Writer agent replies with `editor_writer_result/1`.

## Session protocol (ga-sdk wire)

[`protocol/ga_session_protocol.md`](protocol/ga_session_protocol.md): how the 혁주 작업 공간 session hands user requests to the analysis and editor sessions as ga-sdk `directive/2` / `report/2` / `notify/1` forms over `send_message`. Payloads travel as sha-pinned files in gentleMonster `docs/portfolio/agent_log/`. Validated example heads are in [`protocol/ga_examples/`](protocol/ga_examples/).
