---
layout: post
title: 同步的分步式事务
categories:
- MySQL
- 分步式事务
feature_image: "https://picsum.photos/2560/600?image=390"
---

### 背景
业务需要在后台管理界面创建一类特殊账户，将在多个系统插入记录，要保证所有环节都通过才算成功。
正常情况下都可以成功，但是也可能由于重启等原因不能进行下去，多个调用有一些需要串行，即后面的请求要依赖前面的结果。

### 方案

#### 首先明确业务的参与者：

调用方：后台管理

被调用方： 各系统

流程控制： 后台管理统一把控，简单起见都用串行的，此功能不关注时间

#### 设计思路

首先设计一张表，关键字段如下：

```sql
CREATE TABLE `tb_process_step` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `step` tinyint(4) DEFAULT NULL,
  `extra_info` text COLLATE utf8mb4_bin,
  `request_content` text COLLATE utf8mb4_bin,
  `retry_times` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

```

step： 代表当前执行到第几步 终态为100，定时任务会轮询此字段；

request_content： 初始的请求内容为text类型；

extra_info：为额外信息，保存一些中间结果，供下一个step使用;

retry_times: 重试次数，如果重试次数过多则发告警，人工介入，暂时还没发过。


首先后台管理发起请求，先将请求原始数据入库此时step=0，然后触发pipeline，一个一个的执行。


```java

public interface IProcess {
    Boolean process(ProcessStep processStep);
}

@Service
public class ProcessService{
    public IProcess process1(){
        return (processStep) -> {
            // call remote service1
            // processStep.extraInfo = newinfo;
        };
    }
    
    public IProcess process2(){
        return (processStep) -> {
            // call remote service2
            // processStep.extraInfo = newinfo;
        };
    }
    
    public IProcess processn(){
        return (processStep) -> {
            // call remote servicen
            // processStep.extraInfo = newinfo;
        };
    }
}

public class Caller{
    
    //do schedule
    public void scedule(){
        list = selectUndoneTask();
        list.foreach(doProcess(processStep));
    }
    
    public void doProcess(ProcessStep processStep){
            
            ProcessService service = new ProcessService();
            TreeMap<Integer, IProcess> processes = new TreeMap<>();
            processes.put(1, service.process1());
            processes.put(2, service.process2());
            processes.put(3, service.process4());
            processes.put(n, service.processn());
            
            int step = processStep.step;
            for (Integer key : processes.keySet()) {
                if (key < step) {
                    continue;
                }
                Boolean suc = processes.get(key).process(processStep);
                if (suc) {
                    step++;
                    updateStep(step);
                } else {
                    log.error("执行到第"+step+"步失败");
                    break;
                }
            }
            step = 100;
            updateStep(step); //事务完结   
        }
}

```

#### 执行过程
* 任务创建时先入tb_process_step表，此时step=0
* 执行Caller.doProcess，此时执行的是一条任务链，每执行完一步更新一次step，最后一步置成100终结
* 如果中间步骤失败，退出执行
* scedule 是一个定时1min钟一次，查找未完成的任务，断续执行，记录retrytimes,如果超过阈值则发告警，技术介入处理

<script async src="https://www.googletagmanager.com/gtag/js?id=UA-135360671-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-135360671-1');
</script>