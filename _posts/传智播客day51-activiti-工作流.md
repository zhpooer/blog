title: 传智播客day51-Activiti 工作流
date: 2014-07-02 09:17:28
tags:
- 传智播客
- activiti
---

# 典型流程 #

一个请假流程
1. 员工申请
2. 部门经理审批
3. 老板审批

提取模型类
~~~~~~
class Employee{
    int role;  // 普通员工 | 部门经理 | 老板
}

class LeaveBill { // 请假单
    // 初始录入, 申请请假, 作废(-1),
    // 经理审批通过, 老板审批, 老板驳回, 经理驳回
    int state;
    Employee leaveUser;
    Date leaveDate;
    String reason;
    int days;

    Employee auditManager;
    Date managerAuditTime;

    Employee auditBoss;
    Date bossAuditTime;
}
~~~~~~

如果增加一个审批者, 那么业务流程会改变,
如果是手工编码, 那么就要做花费很大的时间,
所以要使用工作流系统

## 传统设计的弊端 ##
业务模型相关的属性和流程相关的属性, 都放到业务对象中年

传统逻辑和流程相关的逻辑, 都放到业务逻辑中

# 概念 #
工作流(Workflow), 业务过程的部分或整体在计算机应用环境下的自动化,
主要解决的是"使用多个参与者之间按照某种预定义的规则传递文档,
信息或任务的过程自动进行, 从而实现某个预期业务目标, 或者促使此目标的实现"

工作流管理系统: 完成工作流的定义和管理,
按照系统中预定义的规则执行工作流实例,
为企业业务系统运行提供了一个软件支撑环境

工作流逻辑: 规则

工作流实例: 按照规则一次实际流转

工作流管理联盟

# Activiti #

Activiti5 是由 Alfresco 软件在 2010 年 5 月 17 日发布的业务流程管理(BPM)框架 ,它
是覆盖了业务流程管理、工作流、服务协作等领域的一个开源的、灵活的、易扩展的
可执行流程语言框架。

持久层是ibaits, 通过 webService, 提供服务

## 工作流引擎 ##
这是 Activiti 工作的核心。负责生成流程运行时的各种实例及数据、监控和管理
流程的运行。

## BPM2.0 ##

业务流程建模与标注(Business Process Model and Notation,BPMN) ,描述
流程的基本符号,包括这些图元如何组合成一个业务流程图(Business Process
Diagram)


流程图: 描述一系列先后顺序的图, 也叫活动图

## 使用 activiti #
1. 安装 activiti bpm designer eclipse 插件
2. 导入 activiti jar包
3. 搭建环境
  * 代码进行配置
~~~~~~
// 使用
// 创建单机的流程引擎配置
ProcessEngineConfiguration config = ProcessEngineConfiguration
    .createStandaloneProcessEngineConfiguration()
    .setJdbcUrl("jdbc:mysql://localhost:3306/activiti?createDatabaseIfNotExist=true");
    .setJdbcDriver("com.mysql.jdbc.Driver")
    .setJdbcUsername("root")
    .setJdbcPassword("mikeesirius")
    // 自动建表
    .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
    
// 创建流程引擎
ProcessEngine processEngine = config.buildProcessEngine();
~~~~~~
  * 配置文件方式, `activiti.cfg.xml`
~~~~~~
<!-- config = ProcessEngineConfiguration.createProcessEngineFromeResrouce("activiti.cfg.xml") -->
<!-- ProcessEngine processEngine = config.buildProcessEngine(); -->
<bean id="processEngineConfiguration" class="StandaloneProcessConfiguration">
    <property name="databaseSchemaUpdate" value="true"></property>
    <property name="jdbcUrl" value=""></property>
    <property name="jdbcDriver" value=""></property>
    <property name="jdbcUsername" value=""></property>
    <property name="jdbcPassword" value=""></property>
</bean>
~~~~~~
4. 使用 Activiti API
~~~~~~

// 创建核心对象
private ProcessEngine processEngine = ProcessEngines.getDefalutProcessEngine();

// 基本流程: 
// 通过核心对象获取需要的服务
// 调用服务的方法完成操作

// 发布流程规则
public void deploy(){
    RepositoryService repotoryService =
        processEngine.getRepositoryService();
    // 创建发布配置对象
    DeploymentBuilder builder = repositoryService.createDeployment();
    // 做配置
    builder.addClasspathResource("helloworld.bpmn")
           .addClasspathResource("helloworld.png")
    // 发布流程
    buiilder.deploy();
}
// 启动流程
public void startProcess(){
    RuntimeService runtimeService = processEngine.getRuntimeService();
    String processDefinitionId = "myProcess:1:4";
    runtimeService.startProcessInstanceById(processDefinitionId);
}
// 查看任务
public void queryTask(){
    // 找到对应的服务
    TaskService taskService = processEngine.getTaskService();
    // 通过服务, 创建查询对象(XXQuery)
    TaskQeury query  = taskService.createTaskQuery();

    // 添加查询条件
    String assignee = "张三";
    query.taskAssignee(assignee)
         .orderByTaskCreateTime().desc();
         
    // 执行查询
    List<Task> list = query.list();
    for(Task task : list) {
        println(task);
    }
}

// 办理任务
public void complete {
    TaskService taskService = processEngine.getTaskService();
    String taskId = "104";
    taskService.complete(taskId);
}
~~~~~~


## 核心API ##

| 核心 API   | 介绍 |  作用 |
|--------------|
| ProcessEngineConfiguration | 流程引擎配置对象 | 添加数据库连接配置和数据库建表策略 |
| ProcessEngine      | 核心对象, 流程引擎对象   | 大管家, 管理各种服务 |
| ProcessDefinition  | 流程定义对象      |  规定了流程包含了哪些活动, 以及各种活动的执行顺序 |
| ProcessInsctance   | 流程实例对象      |  按照规则实际的一次执行

## activiti 各个服务 ##

| Activiti 各个服务 |
|------------|
| RepositoryService | 管理流程的定义  |
| RuntimeService  | 执行管理, 启动, 推进, 删除流程实例  |
| TaskService     | 任务管理, 管理任务的查看和办理等操作  |
| HistoryService  | 历史记录管理  |
| IdentityService | 组织机构管理  |
| FormService     |     可选, 任务表单管理  |
| ManagerService  |    |

## 流程规则管理(ProcessDefinition) ##

1. 发布规则
2. 查看流程定义信息
3. 删除流程
4. 查看流程图

没有修改流程
1. 如果老流程问题不大, 按照老流程
2. 如果老流程有问题, 按照新流程重写开始

流程很多, 可以由多个流程系统管理器(processeEngine)
~~~~~~
// 发布规则
//  会在数据库三张表中产生数据
// act_ge_bytearray 2条记录, 原始记录文件
// act_ge_procdef 1条记录, 流程记录表, 存放流程定义的属性信息, name key version deploymentid
// act_ge_deployment 1条记录, 存放流程定义的显示名和部署时间
public void deploy(){
    RepositoryService repositoryService = processEngine.getRepositoryService();
    DeploymentBuilder builder = repositoryService.createDeployment();
    // 做配置
    builder.name("请假v1")
           .addClasspathResource("leaveflow.bpmn")
           .addClasspathResource("leaveflow.png"); //如果不设置, 会自动生成, 会有中文问题
    // 发布流程
    buiilder.deploy();
}

// 流程定义
public void queryProcessDefinition() {
    ProcessDefinition query = repositoryService.createProcessDefinitionQuery();
    query // 过滤条件
         .processDefinitionId(processDefinitionId)
         .deploymentId(deploymentId)
         .processDefinitionKeyLike()
         .orderByProcessDefinitionVersion.asc()
    // 执行查询
    List<ProcessDefinition> pds = query.list();
       //.listPage(firstResult, maxResults)
       // .count();
       // .singleResult(); // 如果返回记过数目大于1, 那么直接报错
    for(ProcessDefinition pd : pds) {
        // 由 {key}:{version}:{随机数}
        pd.getId();
        pd.getName(); // 流程定义文件 name 属性
        pd.getKey();  // 流程定义文件 id 属性
        // 如果发布流程和已存在流程的 key 相同, 会在当前key范围最高版本上加一
        // 如果是全新的流程 就是 1
        pd.getVersion();
    }
}
// 删除流程
public void delProcess() {
    // 普通删除, 根据Id删除
    String deploymentId = "***";
    // 如果有流程正在进行, 就会报错, 不允许删除; 如果删除成功, 会保留历史
    repositoryService.deleteDeployment(deploymentId);
    // 级联删除
    // 会删除规则关联的所有信息, 以及历史, 比如正在进行的流程, 也会被删除
    // 推荐给管理员使用
    repositoryService.deleteDeployment(depolymentId, true);
}

// 查看流程图
public void viewImage() {
    // 得到 *.bpmn, *.png
    List<String> names = repositoryService.getDepoymentResourceNames(deploymentid);
    InputStream in = repositoryService.getResourceAsStream(deploymentId, resourceName);
}
~~~~~~


## 流程执行(实例) ##
步骤:
1. 启动流程
2. 查看任务
  * 私有任务
  * 公有任务
3. 接手(认领)任务, 针对共有任务
4. 办理任务
5. 查看流程状态(进度)

任务和流程实例的关系: 任务是对流程实例的扩充, 如任务时间?,
一个 Task 节点和 Execution 节点是 1 对 1 的情况,在 task 对象中
使用 Execution_来标示他们之间的关系


流程实例, 特点:
1. 一个流程只有一个流程实例
2. 流程实例ID不会变
3. 流程实例永远指向当前活动的节点
4. 如果在单线程流程中, 执行对象就是流程实例；
如果在并发流程中, 流程实例会在分支处产生一个执行对象(execution)
作为根(这个就是流程实例), 然后为每个分支下的活动节点创建execution对象,
统一把它们挂在根下面

流程实例(instance)是一个特殊的执行对象(execution),
永远代表的是作为根的执行对象

执行对象(execution), 代表一次活动. 完成第一个节点,
或删除描述第一个活动的执行对象, 然后生成描述第二个执行对象

流程的开始, 就是从开始节点到第一个节点的过程

~~~~~~
// 启动流程, 会改变两张表 act_ru_task  act_ru_execution
public void startProcess() {
    // 通过规则id, 启动确定的一个流程
    ProcessInstance pi =
        runtimeService.startProcessInstanceById(processDefinitionId);
    // 通过key, 启动最新版本的流程
    ProcessInstance pi =
        runtimeService.startProcessInstanceByKey(processDefinitionKey);
}

// 私有任务查看
public void queryPersonalTask(){
    // 获取查询对象
    TaskQuery query = taskService.createTaskQuery();
    // 添加查询条件
    query // 过滤条件
         .taskAssignee(assignee)
         // 排序条件
         .orderByTaskCreateTime().desc();
    // 执行查询
    List<Task> list = query.list();
    for(Task task : list ) {
        task.getName();
        task.getAssignee();
        task.getCreateTime();
    }
}

public void completeTask(){
    taskService.complete(taskId);
}

// 公有任务, 配置流程图时, 去掉 assignee 加上 candidate
public void viewState(){
   // 查询流程实例
   ProcessInstance pi = runtimeService.createProcessInstanceQuery()
                 .processInstanceId(processInstanceId).singleResult
   pi.getActivityId(); // 当前流程状态
}

// 查询共有任务
public void queryCommonTask() {
    // 获取查询对象
    TaskQuery query = taskService.createTaskQuery();
    // 添加查询条件
    query // 过滤条件
         .taskCandidateUser(assignee)
         // 排序条件
         .orderByTaskCreateTime().desc();
}
// 接手任务
public void claim() {
    // 现在就可以获取 user的任务 query.taskAssignee(assignee)
    taskService.claim(taskId, userId);
}
~~~~~~

## 流程变量 ##

如请假流程中的请假天数, 审批意见, 使用Map来存储

流程变量, task, 以及 Execution 都是直接和 流程实例相关的

主要步骤
1. 启动流程
2. 设置流程变量
3. 获取流程变量

~~~~~~
// 启动时, 设置流程变量
public void startProcess(){
    Map<String, Object> vars = new HashMap<>();
    vars.put("test", "测试流程变量");
    ProcessInsctance pi =
        runtimeService.startProcessInstanceByKey(key, vars);
}

public void getVar() {
    // 获取包含流程变量的对象 -- 代办任务
    Task task = taskService.createTaskQuery()
                 .taskAssignee(assignee).list().get(0);
    //通过任务获取流程变量
    taskService.getVariables(taskId); // 获取多个流程变量
    taskService.getVariable(taskId, variableName); 获取单个流程变量
}

// 添加流程变量
public void setVar(){
    taskService.setVariable(taskId, varkey, varvalue);
    taskService.setVariables(taskId, variables);

    // 在完成的时候, 创建
    taskService.complete(taskId, variables);
    // 需要序列化
    taskService.setVariable(taskId, "附件", new MyFile("请假福建", new File("")));
}
~~~~~~

## 历史 ##
Activiti 默认提供了4种历史级别
* none: 不保存任何历史记录, 提高系统性能
* activiti:保存所有的流程实例, 任务, 获得信息
* audit: 默认级别, 添加了表单属性的历史
* full: 最完整的历史记录, 保存的流程变量信息

历史管理只提供了查询功能
* 查询历史流程实例
* 查询历史活动

~~~~~~
// 流程实例
public void queryHistoryProcessInstance(){
    // 创建查询对象
    HistoryProcessInstantceQuery query = historyService
                    .createHistorycProcessInstanceQuery();
    // 添加查询条件
    query
        .processDefinitionKey(key)
        .finished() // 已经结束的
        .orderbyProcessInstanceStartTime().desc();
    List list = query.list();
    // 执行查询
    for(HistoricInstance h : list){
        h.getStartTime();
        h.getDurationInMillis();
        h.getProcessVariables;
        h.getId();
        h.getProcessDefinitionId();
    }
}
// 历史活动
public void queryHistoryActivity(){
     HistoricActivityInstanceQuery query =
         historyService.createHistoricActivityInstanceQuery()
     query.processInsctanceId(processInstanceId)
          .list()
      for(HistoricActivityInstance h : list) {
          h.getActivitId();
          h.getActivityName();
          h.getActivityType();
          h.getAssignee();
      }
}
~~~~~~

# 顺序流 #

# 节点 #
* 开始节点
* 结束节点
* 任务节点
* 网关节点
* 事件节点

# 测试流程 #

步骤:
* 发布流程
* 启动流程
* 检查流程状态

~~~~~~
public class BPMNTestUtils {
    private ProcessEngine processEngine = ProcessEngine.getDefalutProcessEngine();
    protected RepositoryService repositoryService - processEngine.getRepositoryService();
    protected RuntimeService runtimeService = processEngine.getRuntimeService()

    public void deployProcess(){
        DeploymentBuilder builder = repositoryService.createDeployment();
        InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream(resourceName);
        InputStream inputStream = this.getClass().getResourceAsStream(resourceName);
        InputStream inputStream = this.getClass().getResourceAsStream(resourceName);
        builder.addInputStream(resourceName, inputStream);
        builder.deploy();
    }

    public ProcessInsctance getProcessDefinitionById(String pid) {
        return runtimeServcie.createProcessInstanceQuery().processInstanceId(pid).singleResult();
    }

    public void test(){
        deployProcess("start.bpmn");
        ProcessInstance pi = runtimeService.startProcessInstanceByKey(processDefinitionKey);
        pi.isEnded();
    }
    
    // 机器任务节点的流程(recieve task)
    publci void testRecieveTask(){
        Exceution e1 = runtimeService
            .createExecutionQeury()
            .processInstanceId(pid)
            .activityId("汇总当前销售额").singleResult();
        /* 执行汇总任务 */
        // 因为是机器自动任务, 所以没有生成 Task
        // 完成任务
        runtimeService.singal(e1.getId());
        
        // 给老板发短信
        Exceution e2 = runtimeService
            .createExecutionQeury()
            .processInstanceId(pid)
            .activityId("给老板发短信").singleResult();
        /* 执行给老板发短信 */
        runtimeService.getVariable(e2.getId(), "money");
        runtimeService.signal(e2.getId());
        
        // 判断流程状态
        // 通过启动时缓存的 "流程实例id", 使用 runtimeService取查询,
        // 如果得不到, 说明流程已经结束了
        pi = runtimeService
                .createProcessInstanceQuery()
                .processInstanceId(pid)
                .singleResult();
    }
    
    // 用户任务节点的流程(user task)
    public void testUserTask(){
        // 发布
        deployProcess("task1.bpmn");
        // 启动
        ProcessInstance pi = runtimeService.startProcessInstanceByKey("task1");
        String pid = pi.getId();
        
        //获取第一个节点
        // 如果一个节点是 UserTask,
        // 那么流程到达此节点时, 数据库会产生两个对象(Execution 和 Task)
        // 推动任务进行有两个种方式:
        //    runtimeService.signal(excutionId);
        //    taskService.complete(taskId); 删除 task对象 同时删除和task关联的 Execution 对象
        String assignee = "员工";
        // 查询当前启动流程中, 员工用户的任务
        Task task = taskService.createTaskQuery().taskAsignee(assignee)
            .processInstanceId(pid).list().get(0);
        // 完成任务
        taskService.complete(taskId);
        assertNull(getProcessDefinitionById(pid));
    }
}
~~~~~~

# 动态任务分配 #

1. 在 bpmn 图中, 设置 `assignee` 为 `${inputUser}`
2. 在启动流程时, 设置流程变量`inputUser`
3. 编写任务处理器, 在 bpmn 中注册任务管理器
~~~~~~
// 任务处理器
public class ManagerTaskHandler implements TaskListener {
    public void notify(DelegateTask delegateTask) {
        // 设置任务办理者
        delegateTask.setAssignee(assignee);
        // 设置任务候选者
        delegateTask.addCandidateUser(userId);
    }
}
~~~~~~

# 网关(gateWay) #

网关用来控制流程的流向
* 排他网关(Exclusive), 设置分支流向(相当于if)
  * 在 bpmn 图中 设置condition, `${money>1000}`,
  如果没有覆盖全部条件, 抛错, 状态回归到上一级.
  也可以设置 `default flow` 设置默认流程,
  但默认出口上不能设置条件(condition)
* 并行网关(parallelGateWay), 同时进行的两个任务
  * 当流程进入到并行网关, 会产生 n+1 个 execution,
  一个父execution, n 个子 execution.
  当子分支全部汇合, 进入到下一个流程

# Tip #
发布文件名称, 必须以 `.bpmn` 结尾
