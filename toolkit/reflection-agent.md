根据之前的讨论和折中方案，ReflectionAgent 的通用接口可以总结为两层：

一、统一基类接口（所有智能体通用，支持多态）

方法 作用 返回值
step(input: str) -> Result 执行一次完整的“生成→评估→反思→修正”迭代（简化版），返回包含 output 和 done 等字段的 Result。 Result
run(task: str) -> str 自动循环调用 step() 直到任务完成或达到最大次数，返回最终答案。 str
reset() -> None 清空反思记忆、轨迹、计数器等内部状态。 None

二、反思特有扩展接口（提供更细粒度的控制与信息）

方法 作用 返回值
refine(input: str) -> RefineResult 执行一次完整的反思迭代，返回包含初稿、改进稿、评估反馈、反思内容、改进后分数等详细信息的 RefineResult。 RefineResult
evolve(task: str, target_score: float = 0.9, max_iters: int = 5) -> EvolveResult 多次调用 refine() 直到达到目标分数或最大迭代次数，返回最终结果、迭代轮次、历史轨迹、是否收敛等信息。 EvolveResult
get_trajectory() -> List[RefineResult] 返回从上次 reset() 之后所有反思迭代的历史轨迹。 List[RefineResult]

三、接口设计原则

· 统一接口：保证所有智能体（ReAct、Reflection、Plan-and-Execute 等）都能以 step/run/reset 的方式被调度，方便构建通用执行器。
· 特性暴露：通过额外方法（refine/evolve）满足 ReflectionAgent 独特的反思分数、轨迹追踪等需求，避免统一接口的语义损失。
· 返回值规范：
  · Result 至少包含 output: str 和 done: bool，可选 metadata: dict。
  · RefineResult / EvolveResult 可设计为数据类，包含丰富字段。

四、使用示例

```python
agent = ReflectionAgent()

# 简单场景：只用统一接口
answer = agent.run(”写一封专业的道歉邮件“)

# 高级场景：利用特有接口获取反思细节
result = agent.refine(”写一封专业的道歉邮件“)
print(f”反馈: {result.feedback}, 反思: {result.reflection}“)

# 持续优化直到高质量
evo_result = agent.evolve(”写一封专业的道歉邮件“, target_score=0.95)
print(f”迭代了 {evo_result.iterations} 轮，最终分数: {evo_result.score}“)
```

这套接口既保持了与其他智能体的一致性，又充分体现了反思范式的独特价值。