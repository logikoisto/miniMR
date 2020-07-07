# miniMR
一个轻量级的可嵌入的mapreduce框架
# 简介
## 解决了什么问题?
  1. 编写MR类分布式任务时难以进行调试,本框架可本地部署以便于对计算任务的逻辑快速验证,提高调试效率
  2. 封装绝大多数分布式并行计算的常见问题,开发者只需要引入源码包,即可使自己的程序拥有mapreduce的功能
  3. 本着少就是更多的golang设计哲学,提供一个简洁高效的mr实现可以帮助到那些需求定制化的开发者
## 如何工作?
  1. 启动master模式运行的进程
  2. 启动worker模式运行的进程,并向master发送注册请求
  3. master与worker之间进行协议握手,并为其分配runID,构建状态描述对象
  4. master根据配置信息,初始化相应的任务创建m个map任务与r个reduce任务,并初始化到任务队列中
  5. master会构建一个m*n的矩阵记录生成的中间文件完成mapreduce中的shuffle过程
  6. worker启动后会去master拉取任务执行,可能是map也可能是reduce任务根据全局执行的任务状态进行调度
  7. 并且在此过程中要处理一些容错性问题
# 开发计划
  1. 任务调度
      - [x] 负载均衡: 根据worker节点工作负载,来决定任务分配的优先权重。
      - [x] 区域感知: 基于数据所在位置,尽量保证负载均衡的情况下就近调度。
      - [x] 计算调度: map任务完成后根据shuffle过程调度生成reduce任务并分配。
  2. 容错机制
      - [x] 任务恢复: 心跳检测失效的worker节点,调度到新的worker节点上重新执行。
      - [x] 幂等重试: 无论生成的map文件还是最终的reduce结果文件都先写入临时文件并原子重命名为最终的完成文件,以便于输出数据的幂等性。
      - [ ] 失败驱逐: 当输入文件中有异常数据导致执行失败时会上报文件已处理的偏移位置,当重新处理该任务时会跳过该条数据并打印日志。
  4. 数据统计
      - [ ] 进度统计: 在任务执行过程中将每个worker节点的任务执行状况上报并通过暴露指标进行监控。
      - [ ] 单步调试: 在mr运行期间执行到指定的断点处时,将任务状态保存在内存中并停止执行等待调试信号,来实现对MR任务的单步调试。
      
      
### 项目测试框架来源于`MIT6824`课程,感谢老师提供的基础代码
