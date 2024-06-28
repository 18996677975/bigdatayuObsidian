# 1、正确声明线程池
**线程池必须手动通过 `ThreadPoolExecutor` 的构造函数来声明，避免使用 `Executor` 类创建线程池，会有 OOM 风险**。



# 2、监测线程池运行状态
