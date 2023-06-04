> "海森bug"是一种在计算机编程中的术语，指的是那些在试图调试或检查时会改变行为或消失的bug。其名称源于量子力学中的海森堡不确定性原理，即我们无法同时准确知道一个粒子的位置和速度。

# 测试

混沌测试（Chaos Engineering）是一种软件测试方法，旨在发现分布式系统中的潜在故障和性能问题。它通过模拟各种异常条件和故障来测试系统的鲁棒性和可靠性。混沌测试通常会在生产环境中进行，以模拟真实世界的情况，以确保系统能够在各种异常和故障情况下正常运行。

以下是混沌测试的一些最佳实践：
1. 选择合适的混沌工具：选择一个适合你的系统的混沌工具，如 Chaos Monkey、Pumba、Gremlin 等。
2. 定义测试场景：定义测试场景是非常重要的。这有助于你了解你要测试什么、如何测试以及你要实现什么目标。
3. 先从小规模的测试开始：在开始混沌测试之前，先从小规模的测试开始，并逐步增加测试的复杂性和规模。
4. 测试和监控系统：在进行混沌测试时，需要监控系统的各种指标，如 CPU 使用率、内存使用率、网络延迟等。
5. 记录和分析测试结果：对测试结果进行记录和分析是非常重要的。这可以帮助你发现问题并做出相应的改进。
6. 恢复系统：在进行混沌测试时，需要有充分的准备来恢复系统。这可以帮助你在测试过程中快速恢复系统，并确保不会对生产环境造成任何负面影响。

# 一致性语义
Four options for RPC semantics:
-   Naive (above, broken, no guarantees)
-   At least once (NFS, DNS, lab 1b, only possible if you are willing to block forever in the case that the network goes down permanently or the server goes down permanently)
-   At most once (common)
-   Exactly once (lab 1c, only possible if you are willing to block forever in the case that the network goes down permanently or the server goes down permanently）


