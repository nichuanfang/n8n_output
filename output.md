是的，调用 runtimeService.deleteProcessInstance() 删除流程实例后，流程实例的历史数据通常也会被删除或标记为已删除状态，但具体行为取决于流程引擎的配置和版本。

以 Activiti 或 Flowable 为例：

- runtimeService.deleteProcessInstance() 主要删除运行时数据（如 act_ru_* 表中的数据）。
- 历史数据（如 act_hi_procinst 表）默认不会立即删除，而是保留用于审计和查询。
- 如果配置了历史数据的级别（history level）和清理策略，或者调用了历史数据的删除接口（如 historyService.deleteHistoricProcessInstance()），才会删除 act_hi_procinst 表中的对应数据。

总结：
- 直接调用 runtimeService.deleteProcessInstance() 不一定会删除 act_hi_procinst 表中的数据。
- 需要额外调用历史数据删除接口或配置自动清理，才能删除历史流程实例数据。

如果你需要彻底删除流程实例及其历史数据，建议同时调用对应的历史数据删除方法。