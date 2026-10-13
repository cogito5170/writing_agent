# 세션 간 전달 규약 — editor 쪽

**기준 규약은 혁주 작업 공간(허브)이 운영하는 문서 하나다:**
gentleMonster `claude/clever-knuth-1dezlo` 브랜치의 `docs/portfolio/agent_protocol.md`. 형식은 ga-SDK `directive/2` · `report/2`이고 wire는 `dump_wire`를 따른다.

이 문서는 그 규약에서 editor 세션(`session_0122yxjinLCToP7QLnhJbJka`)이 하는 일만 적는다. 둘이 어긋나면 허브 규약을 따른다.

## 받는 것

- 허브가 `send_message`로 `directive/2`(`CMD-HJ<n>`, `after: [<분석 지시>]`)를 ```` ```ga ```` 블록 하나로 보낸다.
- `refs`의 형식은 `저장소@브랜치@sha:경로`이다. 보통 두 파일을 가리킨다.
  - 허브의 입력 파일 `docs/portfolio/handoff/CMD-HJ<k>.input.json`(`hj_handoff/1`)
  - 분석 세션의 결과 파일 `docs/portfolio/handoff/CMD-HJ<k>.result.json`. 이 안의 `semantic_edit_context/1`을 쓴다.
- 읽는 법: gentleMonster에서 `git fetch origin <브랜치>`를 한 뒤 `git show <sha>:<경로>`로 읽는다.

## 하는 일

1. `agents/editor_writer/prompt.md` §1–§15대로 고친다. 입력 매핑은 §15를 따른다.
2. `hj_handoff/1`이 함께 오면 그 안의 다음 항목도 제약으로 지킨다.
   - `user_rules`: 합니다체, 지칭어 빼기, 부분 수정 등
   - `rejected_by_user`: 다시 쓰지 않는다.
   - `open_user_decisions`: 그 구절은 고치지 않는다.
   - `limit_chars`: 공백·줄바꿈을 포함한 `len()`으로 센다.
3. 결과는 `editor_writer_result/1`이다. `agents/editor_writer/output.schema.json`으로 검증한다.
4. 결과 파일을 gentleMonster `claude/beautiful-gates-e42a5m` 브랜치의 `docs/portfolio/handoff/CMD-HJ<n>.result.json`에 커밋하고 push한다. 원고 파일은 고치지 않는다.
5. 허브(`session_01FYwvzhDCTYAAM1u37s5yGE`)에 `report/2`를 wire 꼴로 보낸다. 블록 하나에 꼬리 줄을 붙이고, 다른 글은 쓰지 않는다.
   - 끝냈으면 `status: done`을 쓰고, D 항목마다 `state`를 적는다. 결과 파일 경로는 D1의 `evidence`에 넣는다.
   - 사실 없이는 이을 수 없는 연결이 있으면 결과의 `unresolved[]`에 적는다. report에는 `status: paused`와 `blockers: [{kind: "design"}]`를 쓴다.
   - 예시는 `ga_examples/`에 있고, ga-SDK `ga.forms.validate`에서 문제 0건이다.

## 하지 않는 일

- 분석을 다시 하지 않는다.
- 분석 세션과 직접 주고받지 않는다.
- 사용자에게 결과를 직접 설명하지 않는다. 허브가 보여 준다.

## 자기소개서 작업 우선 규칙 (2026-10-13, CMD-HJ48)

허브 규약 §7-1을 따른다. 위의 원래 규칙은 그대로 두고, 자기소개서 작업에만 아래를 먼저 적용한다.

1. 자기소개서 작업은 gentleMonster `docs/portfolio/PROCESS.md`가 이 문서와 `prompt.md`보다 우선한다. 다른 작업은 원래 규칙대로 한다.
2. 결과는 두 방식 중 지시가 정한 것으로 낸다.
   - 대안 방식 `editor_options/1`(`agents/editor_writer/options.schema.json`): 자기소개서 기본값.
   - 고친 글 방식 `editor_writer_result/1`(`agents/editor_writer/output.schema.json`).
3. "흐름 외 수정 금지"는 흐름 수리 지시에만 적용한다. 문장 길이·반복·사용자 지향 점검 지시는 PROCESS.md §5 기준으로 고친다.
4. 사실이 없어 막히면 보수적 임시 문장과 `blocked_questions`로 끝내고 `status: done`. 한 줄도 쓸 수 없을 때만 `status: paused` + `blockers: [{kind: "design"}]`.
5. 사용자에게 직접 설명하지 않는다. 세션 요약은 허브에 보내는 report/2 안에만 둔다.
