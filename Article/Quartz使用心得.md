title: Quartz使用心得
date:  2017-05-12
tags: Quartz
categories: 
   - 后端
   - Quartz   
------

Quartz是一个第三方任务调度的框架，可以定时，间隔特定天数，还可以在某个月的某一些天的某个时间点执行相应的任务。功能非常强大。下面是我使用的一些心得

### Quartz API 关键的几个接口 ###

1. Scheduler：跟任务调度相关的最主要的API接口
2. Job：你期望任务调度执行的组件定义（调度器执行的内容），都必须实现该接口
3. JobDetail：用来定义Job的实例
4. Trigger：定义一个指定的Job何时被执行的组件，也叫触发器。
5. JobBuilder：用来定义或创建JobDetail的实例，JobDetail限定了只能是Job的实例
6. TriggerBuilder：用来定义或创建触发器的实例

### 使用笔记 ###

``` bash
ISchedulerFactory sf = new StdSchedulerFactory();
 IScheduler sched = sf.GetScheduler();
 IJobDetail job = JobBuilder.Create<Test>()
   .WithIdentity("job1", "group1")
   .Build();
 ITrigger trigger = TriggerBuilder.Create()
                    .WithIdentity("trigger1", "group1")
                    .StartAt(DateBuilder.DateOf(23, 30, 0))
                    .WithCalendarIntervalSchedule(x => x.WithIntervalInDays(3))
                    .EndAt(DateBuilder.DateOf(22, 0, 0))
                    .Build();
sched.ScheduleJob(job, trigger);
sched.Start();
-----Test是要执行的任务类，一定要集成IJob---------
Public class Test:IJob
{
      public void Execute(JobExecutionContext context)
      {
        Console.WriteLine("Test");
      }
}
```

1. 创建`StdSchedulerFactory`实例
2. 创建`Scheduler`实例
3. 创建`JobBuilder`,其中`Test`为你要执行的类，一定要继承IJob接口
4. 创建`TriggerBuilder`

#### Test需要传入参数 ####

改造如下

``` bash
IJobDetail job = JobBuilder.Create<Test>()
    .WithIdentity("myJob", "group1") 
    .UsingJobData("param1", "Hello World!")
    .UsingJobData("param2", 3.141f)
    .Build();

public class Test : IJob
{
    public void Execute(JobExecutionContext context)
    {
        JobKey key = context.JobDetail.Key;

        JobDataMap dataMap = context.JobDetail.JobDataMap;

        string jobSays = dataMap.GetString("param1");
        float myFloatValue = dataMap.GetFloat("param2");

        Console.WriteLine("Test");
    }
}
```

#### WithSimpleSchedule 使用注意事项 #### 

> 如果你只需要你的 job 在某个特定的时刻执行一次, 或者在某一个时刻重复执行几遍, 使用WithSimpleSchedule

``` bash
.WithSimpleSchedule(
        x => x.WithIntervalInHours(48).RepeatForever()
    )
```

- WithIntervalInHours间隔多少小时，也有间隔分钟和秒的方法
- RepeatForever()表示一直执行，另一个方法->WithRepeatCount(10),表示执行10次
- StartAt(DateBuilder.FutureDate(5, IntervalUnit.Minute)) - StartAt,表示开始执行时间，具体参数可以看里面的重载
- EndAt(DateBuilder.DateOf(22, 0, 0))，表示结束时间，同StartAt类似

#### WithCalendarIntervalSchedule 使用注意事项 ####

``` bash
HolidayCalendar cal = new HolidayCalendar();
cal.AddExcludedDate(someDate);

sched.AddCalendar("myHolidays", cal, false);

ITrigger t = TriggerBuilder.Create()
    .WithIdentity("myTrigger")
    .ForJob("myJob")
    .WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
    .ModifiedByCalendar("myHolidays") // but not on holidays
    .Build();

// .. schedule job with trigger

ITrigger t2 = TriggerBuilder.Create()
    .WithIdentity("myTrigger2")
    .ForJob("myJob2")
    .WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(11, 30)) // execute job daily at 11:30
    .ModifiedByCalendar("myHolidays") // but not on holidays
    .Build();
```

#### 使用时候的一些注意点 ####

- StartAt 设置的时间如果比当前时间小的话，程序会立即执行
- 如果间隔N天或者某一段时间，建议使用WithCalendarIntervalSchedule
- 需要排除节假日的用WithCalendarIntervalSchedule
- 具体用什么Schedule可以根据实际的测试来选择