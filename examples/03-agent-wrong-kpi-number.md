# 示例（虚构）/ Example (fictional)：指标摘要代理把工具里没有的数字写进了周报

> 本案例是为了演示写法而编的，不是真实事件。项目名、频道、工单、数值、文件路径和日志均为虚构。不要把它当成某个真实代理或真实业务的记录，也不要当成使用人数或效果统计。
> This case was invented to show the format. It is not a real event. The project, channel, ticket, figures, file paths, and logs are fictional. Do not treat it as a record of a real agent or business, or as usage or outcome statistics.

## 基本信息 / Basics

- 项目 / Project: 虚构项目「灯塔周报」的指标摘要代理 `metrics-brief` / Fictional project "Lighthouse Weekly", agent `metrics-brief`
- 日期 / Date: 2026-04-06
- 记录人 / Recorded by: 代理维护人（示例角色）/ agent maintainer (example role)
- 状态 / Status: 已验收，已写入规范（仅在本虚构案例内）/ verified and written into norms (inside this fictional case only)

## 止损 / Containment

- 受影响的结果 / Affected outputs: 2026-04-06 09:02:28 发到频道 `#lighthouse-brief` 的第 14 周摘要称「Week 14 GMV was 240,000 EUR」。指标接口该周 GMV 为 182400 EUR。该帖已可能被晨会使用。
- 已通知谁 / Who was told: 频道内的运营值班、晨会主持、代理维护人（均为示例角色）。
- 已采取的阻挡措施 / What was stopped or held back: 在原帖下立即更正为 182400 EUR，并标明以 `warehouse.kpi.weekly` 为准；将原帖标为存疑，停止用它做周会判断；暂停 `metrics-brief` 的自动发送，改为人工确认后再发；排查 09:02 之后是否还有同类自动帖。
- 止损完成时间 / Containment completed at: 2026-04-06 09:44。

## 问题陈述 / Problem statement

2026-04-06 09:35，运营值班对照仓库指标表时发现：虚构项目「灯塔周报」的代理 `metrics-brief` 于 09:02:28 发到 `#lighthouse-brief` 的第 14 周摘要写道「Week 14 GMV was 240,000 EUR, up nicely week on week。」工具 `get_kpi` 在 09:02:11 返回的记录为 `period=2026-W14`，`kpi=gmv`，`value=182400`，`currency=EUR`，`source=warehouse.kpi.weekly`。预期是对外数字与工具返回值一致；实际发出的 GMV 是 240000，且帖子里没有 `source` 字段。发送前检查 `pre_send_checks` 只要求 `reply` 为非空且不超过 2000 字。

On 2026-04-06 at 09:35, the duty operator compared the warehouse KPI table with the post. The `metrics-brief` agent had posted to `#lighthouse-brief` at 09:02:28: "Week 14 GMV was 240,000 EUR, up nicely week on week." The `get_kpi` tool at 09:02:11 had returned `period=2026-W14`, `kpi=gmv`, `value=182400`, `currency=EUR`, `source=warehouse.kpi.weekly`. The figure in the post should have matched the tool value. It was 240000, and the post had no `source` field. The pre-send checks (`pre_send_checks`) only required `reply` to be non-empty and at most 2000 characters.

## 三条腿 / Three legs

### A. 发生 / Occurrence

为什么会出现这个错误结果？Why did the wrong result happen?

| 问次 / Why | 回答 / Answer | 证据 / Evidence | 事实或假设 / Fact or hypothesis |
| --- | --- | --- | --- |
| 1 | 发出去的 240,000 来自模型生成的 `reply` 字段，不是工具返回的 `value=182400`。The posted 240,000 came from the model's `reply` field, not from the tool's `value=182400`. | 运行记录 `run/2026-04-06-090211.json`：工具结果 `value=182400`；组装后的 `reply` 原文含「240,000」；发送日志 09:02:28 与该 `reply` 一致。 | 事实 / Fact |
| 2 | 组装步骤只把模型的 `reply` 字符串拿去发送，并不把 `get_kpi.value` 写入正文。The assembler sent the model's `reply` string as-is and did not write `get_kpi.value` into the body. | `assemble_brief.py` 中 `post_body = model.reply`；该文件没有读取 `tool.value` 或 `tool.source` 的代码。 | 事实 / Fact |
| 3 | 输出格式只要求一个自由文本字段，模型可以任意填写数字。The output schema required only a free-text field, so the model could write any number. | `schemas/brief-output.json` 仅有必填字段 `reply`（string）。没有必填的 `gmv_value`、`source`，也没有「必须等于工具返回值」的约束。 | 事实 / Fact |
| 4 | 后来给代理接上了 `get_kpi`，但没有改组装方式和输出格式，数字仍由模型直接写。The `get_kpi` tool was added later; the assembler and schema were not changed, so the model still wrote the number. | 变更记录 `CHANGELOG.md`：2026-03-18 增加 `get_kpi`；同日未改 `assemble_brief.py` 与 `schemas/brief-output.json`。 | 事实 / Fact |

- 根因 / Root cause: 摘要里的指标数字由模型直接写入自由文本，组装步骤和输出格式都没有强制使用工具返回值。 / The KPI figure in the brief was written by the model as free text. Neither the assembler nor the output schema required the tool value.
- 倒读检查 / Backward read: 格式和组装都不绑定工具返回值，所以模型写的 240,000 能进入 `reply`；所以该数字被原样发出。链子接得上。 / The schema and assembler did not bind the tool value, so 240,000 could enter `reply` and was sent as written. The chain follows.

### B. 发现 / Detection

为什么测试、复核或发布前的检查，没有在它出去之前抓住？Why did tests, review, or the release check not catch it before it went out?

| 问次 / Why | 回答 / Answer | 证据 / Evidence | 事实或假设 / Fact or hypothesis |
| --- | --- | --- | --- |
| 1 | 发送是自动的：模型一返回 `reply`，检查通过就发到频道，没有人在发送前对过工具结果。Sending was automatic: once the model returned `reply` and the checks passed, the post went out. Nobody compared it with the tool result before send. | `send_brief.py`：`pre_send_checks` 通过后立即调用 `channel.post`。09:02:11 至 09:02:28 之间没有人工确认记录。 | 事实 / Fact |
| 2 | 发送前检查只看「有没有正文」和「是否超长」，不看数字是否等于 `get_kpi.value`。The pre-send checks only tested that a body existed and was not too long, not that the figure equalled `get_kpi.value`. | `pre_send_checks.py` 仅有 `reply` 非空和 `len(reply) <= 2000`。仓库中不存在把正文数字与 `tool.value` 对照的函数或测试。 | 事实 / Fact |
| 3 | 评测集只看语气和格式，没有「工具返回 182400、模型写成 240000，必须卡住」的样本。The eval set scored tone and format. It had no fixture where the tool returns 182400, the model writes 240000, and the run must be blocked. | `evals/brief-cases.md` 列出 8 条样本，评分项为「完整句子」「语气中性」「不超过 2000 字」。其中没有数字对照，也没有「再引入错误数字」的验收项。 | 事实 / Fact |

- 根因 / Root cause: 发送前没有必须通过对照检查；评测也不覆盖「模型写了与工具不符的数字」这类错误。 / There was no required pre-send comparison, and the evals did not cover a model figure that disagrees with the tool.
- 倒读检查 / Backward read: 没有对照检查，自动发送只会放过非空短文本，所以 240,000 能在 09:02:28 发出，直到 09:35 才被人对照发现。链子接得上。 / With no comparison check, automatic sending only required a short non-empty text, so 240,000 could go out at 09:02:28 and was only found by a human at 09:35. The chain follows.

### C. 体系 / System

为什么我们的流程或规范允许它发生？Why did our process or norms allow this?

| 问次 / Why | 回答 / Answer | 证据 / Evidence | 事实或假设 / Fact or hypothesis |
| --- | --- | --- | --- |
| 1 | 代理上线清单把「工具已接通」和「抽看几条语气」写成了完成标准，没有写「对外数字必须来自工具」。The launch checklist treated "tool connected" and "a few replies reviewed for tone" as done. It did not say that outward figures must come from the tool. | 《代理上线清单》第 4 条「已配置 `get_kpi`」，第 5 条「维护人抽看 5 条摘要的语气」。全文没有「数字来源」或「与工具值对照」。 | 事实 / Fact |
| 2 | 指标摘要被当成写一段话的任务，而不是往外送受约束数字的任务。The brief was treated as writing a paragraph, not as publishing a constrained figure. | 《代理上线清单》和 `evals/brief-cases.md` 的验收语言都是「摘要是否完整、是否得体」，没有「数字与数据源一致」。 | 事实 / Fact |
| 3 | 规范里没有要求：改代理后必须再引入一次同类错误，证明关卡能拦住。The norms did not require a re-injection test after a change: reintroduce the same class of error and prove the gate catches it. | 发布说明「测试」一节只要求「跑评测集」。没有「再引入错误数字」或「关卡验收」项。 | 事实 / Fact |

- 根因 / Root cause: 项目规范没有把「对外指标必须来自工具」和「用再引入错误来验收关卡」写成上线和发布的固定步骤。 / The project's norms did not make "outward KPIs must come from the tool" or "verify the gate by reintroducing the error" a required launch or release step.
- 倒读检查 / Backward read: 规范不要求绑定数据源，也不要求关卡验收，所以组装方式、输出格式和发送前检查都没有这一项；所以错误数字可以发出。链子接得上。 / Because the norms did not require a bound source or a gate test, the assembler, schema, and pre-send checks omitted them, and the wrong figure could go out. The chain follows.

## 停问核对 / Stop-rule check

- 三条腿都追到了能改的组装方式、发送前检查和上线规范，没有停在「模型胡写」或「维护人没看仔细」。 / All three legs end at a changeable assembler, pre-send check, and launch norm. They do not stop at "the model made it up" or "a person was careless."
- 若只把这一条帖子改成 182400，同类错误仍会在其他周、其他指标上再发生，所以不能停在这一次更正。 / If only this post is corrected to 182400, the same class of error can still go out in other weeks and for other KPIs. Do not stop at this one correction.

## 对策 / Countermeasures

每条对策都要落成防错，而不是写完对策再另做一步。不要写「以后小心点」或「下次把提示词写清楚一点」。能把自由文本换成模板或占位符，就不要写解析规则去追金额写法。
Each countermeasure must be realised as error-proofing, not followed by a separate step. Do not write "be more careful" or "write a clearer prompt next time." If free text can be replaced by a template or a placeholder, do not write a parser to chase amount phrases.

| 腿 / Leg | 针对的根因 / Root cause addressed | 对策 / Countermeasure | 防止或拦住 / Prevent or catch |
| --- | --- | --- | --- |
| A 发生 / Occurrence | 数字由模型写入自由文本，未绑定工具值 | 对外指标句由模板生成，数字和来源取自 `get_kpi`：`{period} {kpi} was {value} {currency} (source: {source}).` 模型不再填写 `value`。`notes` 升到消除：不得出现阿拉伯数字或数量词；若要提到工具里的值，只能写占位符（如 `{tool.value}`），由组装步骤替换。不把解析金额当作主关卡。 / The outward KPI sentence is rendered from a template; `value` and `source` come from `get_kpi`. The model does not supply `value`. `notes` is raised to eliminate: no Arabic digits and no quantity words; to mention a tool value, write only a placeholder such as `{tool.value}`, which the assembler substitutes. Amount-parsing is not the main gate. | 防止 / Prevent |
| B 发现 / Detection | 发送前不对数字，评测也不覆盖这类错误 | 发送前关卡失败即拦：工具结果缺失、工具报错或超时、检查脚本自己报错或超时，一律不发送，不回退成模型自写数字。指标句必须是模板按工具字段填出的结果。`notes` 的检查在占位符替换之前进行：只允许字母、空白、标点和已登记的占位符；出现数字或数量词则卡住。替换之后，数字只应出现在占位符被填入的位置。组合表（见下）写进评测集，每次发布都跑。 / The pre-send gate fails closed: a missing tool result, a tool error or timeout, or an error or timeout in the check itself, all block the send, with no fallback to a model-written figure. The KPI sentence must be the template filled from the tool fields. The `notes` check runs before placeholder substitution: only letters, whitespace, punctuation, and registered placeholders are allowed; any digit or quantity word is blocked. After substitution, digits should appear only where a placeholder was filled. The combination table (below) goes into the eval set and runs on every release. | 拦住 / Catch |
| C 体系 / System | 上线规范不要求数据源绑定和关卡验收 | 在《代理上线清单》增加：对外数字必须来自工具或模板，不得由模型直接写；`notes` 不得含数字，只能用占位符引用工具值；发布前必须先完成组合表且每格测试通过。禁止把「提示词里写了要使用工具」或「解析几种金额写法」当作完成标准。 / Add to the launch checklist: outward figures must come from a tool or template, not from the model; `notes` must not contain numbers, only placeholders for tool values; before release, the combination table must be complete and every cell must pass. "The prompt tells the model to use the tool" or "we parse a few amount formats" is not a done standard. | 防止 + 写入规范 / Prevent and write into norms |

## 验收 / Verification

先列出这一类错误的组合表，表里每一格一条测试，再引入原错误。这些测试写进 `evals/brief-cases.md`，每次发布都跑。以下均在虚构的测试通道中执行，不是真实流量。

First enumerate the combination table for this class of error, with one test per cell, then reintroduce the original error. The tests are added to `evals/brief-cases.md` and run on every release. All of the following were run in a fictional test channel, not on live traffic.

错误类型 / Error class：代理发出的指标数字不是来自工具，或 `notes` 自由文本里出现数字 / 数量词，或在工具或关卡不可用时仍然发送。

| 编号 / Id | 工具状态 / Tool state | 指标句 / KPI sentence | notes | 期望 / Expected | 结果 / Result |
| --- | --- | --- | --- | --- | --- |
| T01 | `value=182400` | 模板按工具填充 / template from tool | 无数字：`No fulfilment backlog reported.` | 发送 / send | 通过 / pass |
| T02 | `value=182400` | 模板按工具填充 / template from tool | 仅占位符（检查在替换前）：`The figure is {tool.value}.`（组装后替换为 182400） / placeholder only; checked before substitution | 发送 / send | 通过 / pass |
| T03 | `value=182400` | 模板按工具填充 / template from tool | 空 / empty | 发送 / send | 通过 / pass |
| T04 | `value=182400` | 旧 `reply` 路径：「Week 14 GMV was 240,000 EUR」 | 空 / empty | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T05 | `value=182400` | 模板字段被改成 240000 / template field tampered to 240000 | 空 / empty | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T06 | `value=182400` | 模板按工具填充 / template from tool | 含阿拉伯数字：`... 240,000 ...` | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T07 | `value=182400` | 模板按工具填充 / template from tool | `GMV was 240k` | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T08 | `value=182400` | 模板按工具填充 / template from tool | `0.24 million` | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T09 | `value=182400` | 模板按工具填充 / template from tool | `将近二十五万` | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T10 | 未调用或返回空 / missing or empty | 任意 / any | 任意 / any | 卡住，不回退 / block, no fallback | 通过（已拦住）/ pass (blocked) |
| T11 | 工具报错 / tool error | 任意 / any | 任意 / any | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T12 | 工具超时 / tool timeout | 任意 / any | 任意 / any | 卡住 / block | 通过（已拦住）/ pass (blocked) |
| T13 | `value=182400` | 模板按工具填充 / template from tool | 无数字 / no digits | 检查脚本报错或超时 → 卡住 / check error or timeout → block | 通过（已拦住）/ pass (blocked) |

- 再跑原问题 / Rerun of the original case: 用同一条工具记录 `value=182400`、走新的模板组装再跑，发出的指标句为「2026-W14 gmv was 182400 EUR (source: warehouse.kpi.weekly).」，与工具值一致。这是上表 T01 的路径（同一工具输入，纠正后的发送路径），不是把 09:02:28 那条错误正文再发一遍。错误正文的再引入见 T04。 / Rerun with the same tool record `value=182400` through the new template assembler. The posted KPI sentence is "2026-W14 gmv was 182400 EUR (source: warehouse.kpi.weekly).", which matches the tool. This is row T01 (same tool input, corrected send path), not a resend of the wrong 09:02:28 text. Reintroduction of that wrong text is T04.
- 再引入原错误 / Reintroduction of the original error: 即上表 T04（旧路径写入 240,000）。T06–T09 是同一类在 `notes` 里的组合，不再靠解析「240k = 240000」来比对。 / That is row T04 (legacy path with 240,000). T06–T09 are the same class inside `notes`. The gate does not parse "240k = 240000" in order to compare amounts.
- 正常样本与误拦 / Valid inputs and false blocks: 上表 T01–T03 为应发送的格子，3 条全部通过，误拦 0 条。本虚构案例没有更多「真实业务」样本；若用于真实项目，应再拿一批实际会发出的正常摘要跑一遍，并记下误拦条数。 / Rows T01–T03 are the cells that should send: all 3 passed, 0 false blocks. This fictional case has no further "live" samples. On a real project, a larger batch of genuinely valid briefs should be run, and the false-block count recorded.
- 回归对比 / Regression comparison: 跑既有 8 条语气/格式样本加本表 13 格。语气样本仍通过；新增加的卡住格是本轮故意加上的，不是既有样本变坏。 / The previous 8 tone-and-format cases were run together with these 13 cells. The tone cases still pass. The new blocking cells were added on purpose in this change; no existing case got worse.
- 是否带出新的问题 / Any new issues: 无。原先写在「仍未做完」里的「0.24 million」「将近二十五万」，现在作为数量词 / 数字直接在 T08、T09 被拦住，不再靠金额解析。 / None. "0.24 million" and the Chinese quantity phrase that were previously left open are now blocked as a digit or quantity word in T08 and T09, not by parsing an amount.

## 写入规范与横向排查 / Standardise and look sideways

- 写进了哪些文档、测试或发布清单 / Written into: `assemble_brief.py` 改为模板填充，并对 `notes` 中的占位符做替换；`schemas/brief-output.json` 增加必填 `gmv_value` 与 `source`，且必须等于工具字段；`pre_send_checks.py` 改为：失败即拦，指标句必须来自模板，`notes` 不得含数字或数量词（占位符除外）；`evals/brief-cases.md` 写入上表全部格子，并列为每次发布必跑的回归；《代理上线清单》增加数据源绑定、`notes` 消除规则、组合表验收，以及关卡失败即拦。
- 别处同类问题的排查范围与结果 / Sideways search: 同一代理还可能写订单量、退货金额等其他指标；对每个已接通的 KPI 使用同一模板、同一套占位符和同一张组合表，而不是只修 GMV。原帖里的「up nicely week on week」没有对应的工具字段，已改为：没有比较字段就不得写升降。还检查了其他会自动发数字的代理入口（如日报），有同样漏洞的一并加上关卡。以上均属虚构案例中的排查范围，不是真实审计。 / The same agent may also emit order counts or return amounts; use the same template, the same placeholders, and the same kind of combination table for every connected KPI, not only GMV. The phrase "up nicely week on week" had no comparison field from a tool, so a rise/fall claim is not allowed without that field. Other auto-posting entries that emit figures (such as a daily brief) are checked for the same gap. This search is part of the fictional case, not a real audit.
- 仍未做完的事项 / Still open: 若以后要在摘要里写目标值或其他金额，必须先接通相应的工具字段，只用占位符引用，不能改回在自由文本里写数字。原虚构频道帖的更正沟通已做，不作为本类错误的根因。组合表之外若再出现新的数量词，就加进封闭词表并补一格测试。 / If a target or another amount must later appear in the brief, first connect a tool field and cite it only with a placeholder; do not go back to writing numbers in free text. The correction on the fictional channel post is done, and is not the root cause of this class of error. If a new quantity word appears outside the table, add it to the closed list and add a cell.

## 防错 / Error-proofing

| 字段 / Field | 内容 / Content |
| --- | --- |
| 错误类型 / Error class | 代理发出的指标数字不是来自工具；或 `notes` 自由文本里出现数字 / 数量词；或在工具或关卡不可用时仍然发送。 / An agent-posted KPI figure that does not come from the tool; or digits / quantity words in free-text `notes`; or a send when the tool or the gate is unavailable. |
| 档次 / Level | 指标句和 `notes` 均为「消除 / Eliminate」：数字只来自模板或占位符，模型填不进自由文本。工具缺失、工具或关卡报错 / 超时为「卡住 / Block」，且失败即拦。不以解析金额作为主关卡。 / Both the KPI sentence and `notes` are eliminate: figures come only from the template or placeholders; the model cannot write them as free text. A missing tool, or an error or timeout in the tool or the gate, is block, and the gate fails closed. Amount-parsing is not the main gate. |
| 做在哪里 / Where implemented | `assemble_brief.py`（模板与占位符替换）；`schemas/brief-output.json`；`pre_send_checks.py`（失败即拦、模板校验、`notes` 不得含数字或数量词）；`evals/brief-cases.md`（组合表，每次发布都跑）；《代理上线清单》。 |
| 验收 / Verification | 上表 T01–T13 均按期望通过；其中 T04 为再引入原错误，T08–T09 覆盖原先会漏掉的写法，T10–T13 覆盖失败即拦；应发送的 T01–T03 误拦 0 条。 / T01–T13 all passed as expected: T04 reintroduces the original error, T08–T09 cover phrasings previously left open, T10–T13 cover fail-closed; the send cells T01–T03 had 0 false blocks. |
| 仍需人签字 / Human sign-off | 若以后要在摘要里讨论目标值，需先增加对应工具字段和占位符，并补组合表的格子，不能靠「维护人注意」或在 `notes` 里恢复写数字。 / If a target value is later needed in the brief, first add a tool field and a placeholder, and add cells to the combination table. Do not rely on "the maintainer will be careful," and do not put numbers back into `notes`. |

## 摘要 / Summary

根因不是「这一次模型把数字写错了」，而是：对外指标仍由模型写在自由文本里，发送前不对照工具，上线规范也不要求关卡验收。现在指标句由模板从工具填充；`notes` 里不许出现数字，只能用占位符引用工具值（消除，而不是解析金额）；工具或关卡失败时不发送。验收先按组合表逐格测试，再引入原来的 240,000，并把这些测试列入每次发布都跑的回归。

The root cause is not "the model wrote the wrong number this once." It is that outward KPI figures were still written by the model as free text, the send path did not compare them with the tool, and the launch norms did not require a gate test. The KPI sentence is now filled from the tool by a template; `notes` may not contain numbers, only placeholders for tool values (elimination, not amount-parsing); and a tool or gate failure does not send. Verification enumerates a combination table with a test per cell, then reintroduces the original 240,000, and those tests run on every release.
