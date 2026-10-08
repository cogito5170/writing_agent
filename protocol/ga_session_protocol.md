# 세션 간 전달 규약 (ga-sdk semantic wire)

혁주 작업 공간이 사용자 요구를 받아 analysis 세션과 editor 세션에 넘긴다. 분석과 수정은 두 세션이 하고, 작업 공간은 결과만 사용자에게 보여 준다.

이 문서는 gentleMonster `docs/portfolio/agent_protocol.md`(작업 공간이 만든 규약)의 **§1 봉투와 전송 방식만 바꾼다.** §2 본문 형식, §3 경로, §4 workspace_rules, §5 사용자 출력은 그대로 쓴다.

## 1. 세션

| 역할 | `to` / `from` 값 | 세션 | 결과 파일을 올리는 gentleMonster 브랜치 |
|---|---|---|---|
| 작업 공간 | `workspace` | 혁주 작업 공간 `session_01FYwvzhDCTYAAM1u37s5yGE` | `claude/clever-knuth-1dezlo` |
| analysis | `analysis` | Context Analysis Agent `session_01GGPjmZVBdKmygvWvYsFFv6` | `claude/modest-cannon-ha8zs3` |
| editor | `editor` | Editor_Writer Agent `session_0122yxjinLCToP7QLnhJbJka` | `claude/beautiful-gates-e42a5m` |

- 세 세션 모두 gentleMonster를 갖고 있다. 큰 본문은 여기에 파일로 주고받는다.
- analysis와 editor는 서로 직접 주고받지 않는다. 모든 메시지는 작업 공간을 거친다.

```
사용자 → 작업 공간 ─directive/2 (CMD-WS<n>, to analysis)→ analysis 세션
                  ←report/2 (evidence: analysis_result.json)─
         작업 공간 ─directive/2 (CMD-WS<n+1>, to editor, after)→ editor 세션
                  ←report/2 (evidence: edit_result.json)──────
사용자 ← 작업 공간 (agent_protocol.md §5의 결과만 출력)
```

## 2. 전송 (wire)

- **도구:** `mcp__claude-code-remote__send_message`. `session_id`는 §1 표의 세션.
- **본문:** ga-sdk 폼(ga-SDK `DESIGN.md` §32 S1) 하나를 minified JSON으로 쓴 ```` ```ga ```` 블록 하나뿐이다. ga-sdk `forms.dump_wire(head)`의 출력과 같다(끝의 `---`와 footer 줄은 넣어도 되고 빼도 된다). 블록 밖에 산문을 쓰지 않는다.
- **사람이 읽을 말**은 `note` 필드에 280자 이내로 쓴다.
- **쓰는 폼:** `directive/2`, `report/2`, `notify/1` 세 가지. 필드 규칙은 ga-sdk `ga.forms`를 따른다. 모르는 필드를 넣으면 HARD 오류가 난다. 그래서 원고나 진단 같은 본문은 head에 넣지 않는다. 파일로 올리고 URL로 건다.

## 3. 본문 파일

- 위치: gentleMonster `docs/portfolio/agent_log/<지시 id>/`
- URL은 커밋 sha를 고정해서 쓴다:
  `https://github.com/cogito5170/gentleMonster/blob/<sha>/docs/portfolio/agent_log/<지시 id>/<파일>`
- 받는 쪽은 §1 표의 브랜치를 `git fetch origin <branch>` 한 뒤 `git show <sha>:<path>`로 읽는다.
- 파일을 커밋하는 범위:
  - 작업 공간: 요청 파일, 그리고 원고·기록 파일 전부.
  - analysis와 editor: 자기 브랜치의 `agent_log/<지시 id>/` 아래 결과 파일 하나만. 원고 파일은 고치지 않는다.

| 파일 | 쓰는 쪽 | 형식 |
|---|---|---|
| `CMD-WS<n>/analysis_request.json` | 작업 공간 | `analysis_request/1` = agent_protocol.md §2-1 본문 + `"schema"`, `"task"`, `"workspace_rules"`(§4 그대로) |
| `CMD-WS<n>/analysis_result.json` | analysis | task가 revise면 `semantic_edit_context/1`, diagnose/verify면 `context_flow_analysis/1` (§2-2 필수 항목 포함) |
| `CMD-WS<m>/edit_result.json` | editor | `semantic_edit_result/1` (agent_protocol.md §2-4) |

- editor에게는 새 요청 파일을 만들지 않는다. 작업 공간은 `analysis_request.json`과 `analysis_result.json`의 URL을 그대로 `refs`에 넣는다. 이것이 §2-3 "고치지 않고 그대로 전달"이다.
- propose_options처럼 editor만 부르는 요청이면 작업 공간이 `CMD-WS<m>/edit_request.json`(§2-3 형식)을 만든다.

## 4. 폼 쓰는 법

### 4-1. `directive/2` (작업 공간 → analysis·editor)

- `id`: `CMD-WS<번호>`. 작업 공간이 1부터 차례로 매긴다. 사용자 요구 하나가 지시 1~2개가 된다.
- `rev`: 처음은 1. rev 1에는 `scope`(`S<n>`)와 `done_when`(`D<n>`)이 있어야 한다. 내용을 바꿔 다시 보내면 rev를 올리고 바뀐 항목만 `changes`에 쓴다.
- `to`: `analysis` 또는 `editor`.
- `goal`에 task(diagnose · revise · propose_options · verify)를 적는다.
- `why`에 사용자 말 원문을 짧게 적는다.
- `refs[0]`은 요청 본문 파일, 마지막은 이 규약의 URL.
- editor 지시에는 `after: ["CMD-WS<analysis 지시>"]`를 붙이고, `refs`에 analysis 결과 URL을 넣는다.

### 4-2. `report/2` (analysis·editor → 작업 공간)

- `handled[].status`:
  - `done`: 끝냄.
  - `paused`: 사용자 판단이 필요함.
  - `declined`: 이 세션의 일이 아님.
- `items[]`는 지시의 `done_when`과 같은 `D<n>`이다. `state`는 `met`, `unmet`, `blocked`, `na` 중 하나.
- 결과 파일 URL은 D1의 `evidence`에 넣는다.
- `commits`: 결과 파일을 올린 커밋.
- 일을 못 할 때(agent_protocol.md §2-5 `error`): 결과 파일에 `{ "reason", "missing" }`를 쓰고, report에는 `status: paused`, 해당 item을 `blocked`로 둔다. 이유는 `blockers: [{kind: "design", what: "..."}]`에 적는다. 빈칸을 추측해서 채우지 않는다.
- 사용자가 골라야 할 것(`needs_user_decision`)이 있으면 결과 파일에 적고, report는 `paused` + `blockers`로 알린다.

### 4-3. `notify/1`

- 규약을 처음 받으면 작업 공간에 `kind: "ack"`, `ref`: 이 규약 URL로 한 번 보낸다. 그러면 세션 간 전송이 되는지 확인된다.

## 5. 작업 공간이 하는 일 / 하지 않는 일

**한다:**
- 사용자 요구를 task로 나눈다(agent_protocol.md §3).
- 요청 파일을 커밋하고 directive를 보낸다.
- report를 받으면 결과 파일을 읽어 §5 형식으로 사용자에게 보여 준다.
- 받은 wire 메시지 원문을 `agent_log/`에 남긴다.
- 사용자가 고른 안을 원고에 반영한다.

**하지 않는다:**
- 원고를 직접 진단하거나 고쳐 쓰지 않는다.
- 에이전트 결과를 바꿔서 전달하지 않는다.
- 진행 과정과 wire 메시지를 사용자에게 보여 주지 않는다.

## 6. 예시

`ga_examples/`의 head는 모두 ga-sdk `ga.forms.validate`로 문제 0건을 확인했다(ga-SDK `claude/CMD-GA57`). `<..._sha>`는 실제 커밋 sha로 바꾼다.

| 파일 | 내용 |
|---|---|
| `01_directive_analysis.json` | 작업 공간 → analysis, revise 요청 |
| `02_report_analysis.json` | analysis → 작업 공간, 완료 |
| `03_directive_editor.json` | 작업 공간 → editor, analysis 결과를 그대로 전달 |
| `04_report_editor_blocked.json` | editor → 작업 공간, 사용자 판단 필요 |
| `05_notify_ack.json` | 규약 수신 확인 |

wire에 실리는 모양(01번):

````
```ga
{"schema":"directive/2","id":"CMD-WS1","rev":1,"to":"analysis","goal":"self_intro 원고의 흐름을 진단하고 고칠 곳을 semantic_edit_context/1로 정한다 (task: revise)","why":"사용자 피드백: 문맥이 안 이어지는 느낌","scope":[...],"done_when":[...],"refs":["https://github.com/cogito5170/gentleMonster/blob/<workspace_sha>/docs/portfolio/agent_log/CMD-WS1/analysis_request.json","https://github.com/cogito5170/writing_agent/blob/claude/beautiful-gates-e42a5m/protocol/ga_session_protocol.md"],"change_size":"implementation"}
```
````

## 7. editor 결과 형식

editor 세션의 프롬프트(`agents/editor_writer/prompt.md`) §11에는 `editor_writer_result/1`이 정의돼 있다. 이 규약으로 불릴 때는 작업 공간 §5 출력에 필요한 `semantic_edit_result/1`(before · after · options · char_count · needs_user_decision)로 돌려준다. 대응 관계:

| editor_writer_result/1 | semantic_edit_result/1 |
|---|---|
| `final_document` | `revised_text` |
| `changes[]` | `changes[]` (+ before · after · options) |
| `unresolved[]` | `untouched[]` · `needs_user_decision[]` |
| `preserved` | `fact_check` |

수정 원칙(prompt.md §1–§14)은 그대로 적용한다.
