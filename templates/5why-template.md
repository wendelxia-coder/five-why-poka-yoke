# 5 Why 记录 / 5 Why record

> 填写时只用可核对的事实。还没验证的写进「假设」，先验证再往下追。
> Use only facts that can be checked. Label unverified items as hypotheses and test them before building on them.

## 基本信息 / Basics

- 项目 / Project:
- 日期 / Date:
- 记录人 / Recorded by:
- 状态 / Status: （止损中 / 分析中 / 已验收 / 已写入规范）（containment / analysis / verified / written into norms）

## 止损 / Containment

- 受影响的结果 / Affected outputs:
- 已通知谁 / Who was told:
- 已采取的阻挡措施 / What was stopped or held back:
- 止损完成时间 / Containment completed at:

## 问题陈述 / Problem statement

（一两句话：什么事、在哪里、何时发现、谁发现、预期 vs 实际，带上单号、字段、数值、日志或文件位置。）

（One or two sentences: what, where, when found, by whom, expected vs actual, with record ids, fields, values, logs, or file locations.)

-

## 三条腿 / Three legs

### A. 发生 / Occurrence

为什么会出现这个错误结果？Why did the wrong result happen?

| 问次 / Why | 回答 / Answer | 证据 / Evidence | 事实或假设 / Fact or hypothesis |
| --- | --- | --- | --- |
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |
| … |  |  |  |

- 根因 / Root cause:
- 倒读检查（从根因用「所以」读回问题，是否每步都接得上）/ Backward read with "therefore" (does every link follow?):

### B. 发现 / Detection

为什么测试、复核或发布前的检查，没有在它出去之前抓住？Why did tests, review, or the release check not catch it before it went out?

| 问次 / Why | 回答 / Answer | 证据 / Evidence | 事实或假设 / Fact or hypothesis |
| --- | --- | --- | --- |
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |
| … |  |  |  |

- 根因 / Root cause:
- 倒读检查 / Backward read with "therefore":

### C. 体系 / System

为什么我们的流程或规范允许它发生？Why did our process or norms allow this?

| 问次 / Why | 回答 / Answer | 证据 / Evidence | 事实或假设 / Fact or hypothesis |
| --- | --- | --- | --- |
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |
| … |  |  |  |

- 根因 / Root cause:
- 倒读检查 / Backward read with "therefore":

## 停问核对 / Stop-rule check

- 是否已追到能改的流程或体系原因？ / Reached a process or system cause you can change?
- 是否停在了某个人身上？若是，继续问流程为什么会放行。 / Did the chain stop at a person? If yes, keep asking why the process allowed it.
- 若拿掉当前原因，问题还会不会再来？还会，就继续问。 / If this cause were removed, would the problem recur? If yes, keep going.

## 对策 / Countermeasures

优先级：先防止发生，再在过关前检查拦住。不要写「以后小心点」。
Hierarchy: prevent first, then catch it at a gate. Do not write "be more careful."

| 腿 / Leg | 针对的根因 / Root cause addressed | 对策 / Countermeasure | 防止或拦住 / Prevent or catch |
| --- | --- | --- | --- |
| A 发生 / Occurrence |  |  |  |
| B 发现 / Detection |  |  |  |
| C 体系 / System |  |  |  |

## 验收 / Verification

- 再跑原问题的结果 / Rerun of the original case:
- 再引入原错误（或同类变体）后，新检查是否抓住 / After reintroducing the error (or a same-class variant), did the new check catch it:
- 正常、有效的输入是否仍通过 / Do normal, valid inputs still pass:
- 是否带出新的问题 / Any new issues:

## 写入规范与横向排查 / Standardise and look sideways

- 写进了哪些文档、测试或发布清单 / Written into which documents, tests, or release checklists:
- 别处同类问题的排查范围与结果 / Sideways search: scope and findings:
- 仍未做完的事项 / Still open:

## 摘要 / Summary

- 根因 / Root causes:
- 改了什么，所以它不能再以同样方式出去 / What was changed so it cannot get out the same way:
- 还没做完的 / What remains open:
