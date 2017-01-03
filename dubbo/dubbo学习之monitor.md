# dubbo 学习之monitor #
dubbo在monitor里面使用了ReentrantLock、ConcurrentHashMap、ScheduledFuture、ScheduledExecutorService、AtomicReference等并发工具类，可以好好消化一下他是如何处理并发问题的
#API
## MonitorService
## MonitorFactory
## Monitor
Monitor集成了Node和MonitorService
# dubbo的默认实现  
## DubboMonitor
## DubboMonitorFactroy