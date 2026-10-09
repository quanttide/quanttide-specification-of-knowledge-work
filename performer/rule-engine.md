# 规则引擎

规则引擎（RuleEngine）是执行者之一：由程序当场核，机械比对。`executor` 取 `rule`，即把这一步交给规则引擎。

`rule` 判据的判法由平台定义，不进定义——定义只声明这一步由规则引擎核（见 [工作流](../process/workflow.md)）。

核过即落一条 `is_succeeded` 为 `true` 的工作记录（见 [工作记录](../process/work-record.md)）。
