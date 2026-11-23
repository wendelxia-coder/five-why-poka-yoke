# 防错记录 / Error-proofing record

> 把对策做成过关前必须过的检查，写进脚本、测试或 AI 步骤，不要只写在备忘里。
> Turn the countermeasure into a check that must pass at the gate. Put it in a script, a test, or an AI step, not only in a note.

## 基本信息 / Basics

- 项目 / Project:
- 日期 / Date:
- 记录人 / Recorded by:
- 对应的 5 Why 记录（如有）/ Related 5 Why record (if any):

## 错误类型 / Error class

（同一类错误怎么识别，不要只写这一次的表面现象。）

（How to recognise this class of error, not only the surface of this one case.)

-

## 对应的根因 / Related root cause

-

## 先问 / First questions

- 能不能让这类错误根本没法发生？ / Can we make this class of error impossible?
- 不能的话，能不能在过关前用脚本或 AI 检查把它卡住？ / If not, can a script or an AI check catch it and hold the gate?
- 是否出现了「以后小心点」？若有，删掉重写。 / Did "be more careful" appear? If yes, remove it and rewrite.

## 档次 / Level

选能做到的最高档：消除 > 卡住 > 示警。
Pick the strongest level that is feasible: eliminate > block > alert.

- [ ] 消除 / Eliminate：错误没法发生
- [ ] 卡住 / Block：错误一出现，流程就停
- [ ] 示警 / Alert：允许继续，但必须醒目标出

选用这一档的原因 / Why this level:

## 做法 / What was put in place

-

## 写在哪里 / Where it lives

- 代码、测试、过关脚本或格式约束的名称 / Name of the code, test, gate script, or schema:
- 项目规范或检查清单中的位置 / Location in the project's norms or checklist:

## 验收 / Verification（必做 / required）

- 再引入的错误或同类变体 / Error (or same-class variant) that was reintroduced:
- 结果：被消除、被卡住，还是被标出 / Result: eliminated, blocked, or flagged:
- 正常、有效的输入是否仍通过 / Do valid inputs still pass:
- 验收证据（日志、测试输出、截取的检查结果）/ Evidence (log, test output, captured check result):

## 仍需人签字的部分 / Left to human sign-off

-

## 摘要 / Summary

| 字段 / Field | 内容 / Content |
| --- | --- |
| 错误类型 / Error class |  |
| 档次 / Level |  |
| 做在哪里 / Where implemented |  |
| 验收 / Verification |  |
| 仍需人签字 / Human sign-off |  |
