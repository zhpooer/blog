title: 传智播客day52-Activiti项目实例
date: 2014-07-05 09:11:56
tags:
- 传智播客
- activiti
---

# 实现步骤 #
1. 绘制流程图(eclipse插件)
2. 准备业务模型, 只要保证模块CRUD基础功能都能使用
3. 流程部署部署管理
  1. 发布新流程(文件上传)
  2. 部署管理(删除部署)
  3. 流程定义管理(查看规则流程图, 历史查看)
4. 任务查看(私有/共有任务查看)
  1. 查看任务(办理任务, 接受任务)
  2. 查看当前流程图
5. 任务办理
6. 监听器, 完成业务动态办理

扩展任务
1. 多出口任务
2. 批注的添加和查看

# Activiti和SSH集成 #

集成的核心, 把对应框架的核心类交给Spring管理
(如果有事务或者数据源, 也得交给Spring管理)

~~~~~~
<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
    <!-- 数据源  -->
    <property name="dataSource" ref="dataSource"></property>
    <!-- 建表策略 -->
    <property name="databaseSchemaUpdate" value="true"> </property>
    <!-- 事务 -->
    <property name="transactionManager" ref="transactionManager"></property>
</bean>
<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
    <property name="processEngineConfiguration" ref="processEngineConfiguration"></property>
</bean>
<bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService"></bean>
<bean id="taskService" factory-bean="processEngine" factory-method="getTaskService"></bean>
<bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService"></bean>
<bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService"></bean>
<!-- action -->
<bean id="workflowAction" class="WorkFlowAction" scope="prototype">
    <property name="workflowService" ref="workflowService"></property>
</bean>
<!-- service -->
<bean id="workflowService" class="WorkFlowServiceImpl" >
    
</bean>
~~~~~~

activiti具有额外功能,定时器

控制器 action 功能,获取页面取得的参数

~~~~~~

public class WorkFlowServiceImpl implements IWorkFlowService {
    @Resource private RepositoryService repositoryService;
    
    public String newDeployment(String processName, File processFile){
        DeploymentBuilder builder = repositoryService.createDepoyment();
        InputStream in = new FileinputStream(processFile);
        ZipInputStream zipInputStream = new ZipinputStream(in);
        builder.name(processName)
               .addZipInputStream(zipInputStream);
        builder.deploy();
    }
    public List<Deployment> getAllDeployments(){
        return repositoryService.createDeploymentQeury()
                     .orderbyDeploymentTime()
                     .desc().list()
    }
    public List<> getAllDeploymentDefinitions(){
        return repositoryService.createProcessDefinitionQuery().orderbyKey().asc().orderbyversion.dsc().list()
    }
}

public class WorkFlowAction extends BaseAction{
    public static final String LIST = "list";
    public static final String MAIN = "main"
    public static final String DEPLOY_SUCCESS = "deploy_success";
    
    private IWorkFlowService workFlowService;    
    public String execute(){
        List<Depoyment> ds = workFlowservice.getAllDeployments();
        putContext("ds", ds);

        List<ProcessDefinition> pds = workFlowservice.getAllProcessDefinitions();
        putContext("pds", pds);
        
        return SUCCESS;
    }
    privatge String name;
    private File file;
    public String newDeplyment(){
        workflowService.newDeployment(name, file);
        return ;
    }
}
~~~~~~

~~~~~~
<action name="workflow_*" class="workflowAction" method="{1}">
    <result> /WEB-INF/views/workflow/deployment.jsp </result>
    <result name="deploy_success" type="redirectAction">
        workflow.action
    </result>
</action>
~~~~~~

~~~~~~
<!-- deployment.jsp -->
<!-- 部署信息查看 -->
<table>
    <tr>
        <th>编号</th>
        <th>名称</th>
        <th>发布时间</th>
        <th>操作</th>
    </tr>
    <s:iterator value="#ds" >
        <tr>
            <td> <s:property value="id"></s:property> </td>
            <td> <s:property value="name"></s:property> </td>
            <td>
<s:date name="deploymentTime" format="yyyy-MM-dd" hh:mm:ss=""></s:date>
            </td>
            <td> 删除 </td>
        </tr>
    </s:iterator>
</table>

<!-- 流程定义信息查看 -->
<table>
    <tr>
        <th>编号</th>
        <th>名称</th>
        <th>Key</th>
        <th>版本</th>
        <th>部署id</th>
        <th>规则文件名</th>
        <th>规则图片名</th>
        <th>操作</th>
    </tr>
    <s:iterator value="#pds" >
        <tr>
            <td> <s:property value="id"></s:property> </td>
            <td> <s:property value="name"></s:property> </td>
            <td> <s:property value="key"></s:property> </td>
            <td> <s:property value="version"></s:property> </td>
            <td> <s:property value="deploymentId"></s:property> </td>
            <td> <s:property value="resourceName"></s:property> </td>
            <td> <s:property value="diagramResourceName"></s:property> </td>
            <td> 查看规则流程图 查看历史 </td>
        </tr>
    </s:iterator>
</table>
<!-- 上传流程图 -->

<s:form action="workflow_newDeployment" enctype="multipart/form-data">
    流程名称:
    <s:textfield name="name"></s:textfield>
    文件文件:
    <s:file name="file"></s:file>
    <s:submit value="上传流程"></s:submit>
</s:form>
~~~~~~

# 启动流程 #

需要思考的问题
1. 如何通过业务对象启动对应的流程
        让业务对象的特性和流程的Key有一定关系
        可以以业务类的类名作为流程的key
2. 流程运行过程中需要些什么(流程变量)
3. 如何根据流程找到业务对象
        根据id和类名
4. 如何根据业务对象找到流程(流程实例), 根据 "businessKey", 业务键
~~~~~~
// businessKey = classname + id
// processDefinitionKey 和 businessKey 是唯一约束
runtimeService.startProcessInstanceByKey(processDefinitionKey, businessKey, variables);
runtimeService.createProcessInstanceQuery()
~~~~~~
5. 面对流程修改的解决方案
        不严重: 老流程按照老规则流转, 新流程按照新规则流转
        严重: 冻结老流程, 不允许老流程再进行审批, 所有的类似流程都按照新规则进行流转

# 请假流程启动 #
~~~~~~
// LeaveBillAction, 启动请假业务流程
public String startProcess(){
    // 获取业务对象ID
    // 调用业务逻辑
    leaveBillService.startProcess(id);
}

public LeaveBillServiceImpl extends ILeaveBillService{
    // 启动流程
    //   修改业务对象状态
    //   封装流程需要参数
    //   启动流程
    public void startProcess(Long id){
        LeaveBill bill = get(id);
        if(bill!=null) {
            bill.setState(1);
            update(bill);

            Map<String, Object> vars = new HashMap<String, Object>();
            vars.put("inputUser", bill.getUser());
            vars.put("days", bill.getDays());
            vars.put("id", bill.getId());
            vars.put("class", bill.getClass().getSimpleName());
            
            // 启动流程
            String processKey = bill.getClass().getSimpleName();
            workflowService.startProcess(processKey, vars);
        }
    }
}

// workflowService
public void startProcess(String processKey, Map vars){
    String businessKey = processKey + "." + var.get("id");
    runtimeService.startProcessInstanceByKey(processKey, businessKey, vars);
}
~~~~~~

# 任务管理 #
~~~~~~
// WorkFlowAction
public String listTask(){
    Employee currentUser = session.get("loginUser");
    if(currentUser != null) {
        // 私有任务查看
        List<Task> pTasks = workflowService.getPersonalTask(currentUser.getName());
    }
    // 公有任务查看
    return LIST_TASK; // task.jsp, 办理, 查看当前流程图
}

// workflowService
public List getPersonalTask(String assinee) {
    taskService.createTaskQuery()...;
}
~~~~~~


## 查看当前流程图 ##

~~~~~~
// workflowAction
public Strinig viewCurrentImage(){ // type=stream
    ProcessDefinition pd = workflowService.getProcessDefinitionByTaskId(taskId);
    resourceName = pd.getDiagramName();
    deploymentId = pd.getDeploymentId();
    
    // 获取坐标, x: activity.x; y: activity.y; width & height
    Map<String, Object> currentActivityCoordinates =
        workflowService.getCurrentActivityCoordinates(taskId);
    putContext(asc, currentActivityCoordinates);
    return CURRENT_IMAGE;  // return current.jsp <img src=resource.../>
}

// workflowService
public ProcessDefinition getProcessDefinitionByTaskId(String taskId){
    Task task = getTask(taskId);
    String processDefinitionId = task.getProcessDefinitionId();
    return repositoryService.createProcessDefinitionQuery()
             .processDefinitionId(processDefinitionId);
}

public ActivityImpl getActivity(String taskId){
    
    Task task = getTask(taskId);
    // 通过taskId 获取 Execution
    Execution execution = getExecution(task.getExcutionId());
    String activityId = execution.getActivityId();

    // 由与活动对象访问特别频繁, 所以设计者就把Activity信息放入了缓存中
    // 而创建XXQuery对象, 只是进行了sql拼装, 只能到数据库汇总查询 获取不到缓存中的内容
    // 所以下面这一句不行
    // ProcessDefinitionEntity pd = getProcessDefinitionByTaskId(taskId);
    ProcessDefinitionEntity pd = repositoryService.getProcessDefinition(task.getProcessDefinitionId);
    ActivityImpl activity = pd.findActivity(activityId)
    return ActivityImpl;
}

public Execution getExecution(String executionId) {
    return runtimeService.createExecutionQeury().excutionId(executionId);
}
public Task getTask(String taskId){
    return taskService.createTaskQeury()...;
}
~~~~~~

![关系图](/img/activiti_api.png)


# 办理任务 #
在任务列表中, 提供一个 "办理任务" 按钮, 打开后打开任务表单,
同时, 在表单的最下方提供"办理任务"操作

在办理任务过程中, 如何找到业务表单?
* 解决方案一(动态): 在task节点上, 制定"表单属性", 存入规则, 在办理当前任务时,
可以根据预先制定的表单项, 通过HTML DOM动态生成任务表单
* 解决方案二(静态): 预先制定好任务表单(HTML或者JSP页面), 通过form key 属性,
和这个表单建立关联
        taskForm.jsp, 这个表单需要业务数据
        **.audit.action 在每一个业务控制器中, 提供audit方法,
        由此方法加载业务数据, 然后返回业务对象相关流程表单

~~~~~~
// formKey: leaveBillAudit.action
// workflowAction
public String viewTaskForm(){
    // 通过taskId获取 formKey
    String formKey = workflowService.getTaskFormKey(taskId);
    putContext("formUrl", formKey + "?id=" + objId + "&taskId=" + taskId);
    Long objId = workflowService.getObjId(String taskId);
    return TASK_FORM; // redirectAction ${formUrl}_audit.action
}

// workflowService
public String getTaskFormKey(String taskId) {
    Task task = getTask(taskId);
    String processDefinitionId = task.getProcessDefinitionId();
    String taskDefinitionKey = task.getTaskDefinitionKey();
    String formKey = formSservice.getTaskFormKey(processDefinitionId, taskDefinitionKey);
    // 方式二:
    // formService.getTaskFormData(taskId);
    // formKey = formData.getFormKey();
    return formKey;
}

public Long getObjId(String taskId) {
    String variableName = "objId";
    return taskService.getVariable(taskId, variableName);
    // 方式二:
    // 通过task获取流程实例
    // 获取businessKey >> leabeBill.5, 截取
}

// leaveBillAction
public String audit(){
    // 获取需要加载业务对象的唯一标识
    // 加载业对象
    // 放入值栈
}
~~~~~~


# 完成任务 #
~~~~~~
// workflowActioin
public String completeTask(){
    workflowService.completeTask(String taskId);
    return TASK; // workflow_listtask.action
}

// workflowService
public void completeTask(String taskId) {
    taskService.complete(taskId);
}
~~~~~~

# 流程监听器完成业务操作 #

任务监听器
1. 创建一个普通类来充当监听器, 需要实现TaskListener接口
2. 实现notify方法的逻辑
3. 需要把它们配置到流程中任务节点上

执行监听器
1. 创建一个普通类充当监听器, 需要实现ExecutionListener接口
2. 实现notify方法的逻辑
3. 需要把它们配置到流程中的任意节点上

~~~~~~
// 自动安排审核者
public class MangerTaskHandler implements TaskListener {
    @Override
    public void notify(DelegateTask delegateTask) {
        // 获取到任务申请人的唯一标识
        String inputUser = delegateTask.getVariable("inputUser");
        WebApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext);
        IEmployService employeeService = ctx.getBean("employeeService",IEmployeeService.class);
        // 通过唯一标识获取相对应的对象
        Employee emp = employeeService.getEmployeeByName(inputUser);
        //通过员工对象获取,他的经理
        Employee manager = emp.getManager();
        delegateTask.setAssignee(manager.getName());
    }
}

// 请假流程结束时自动审批处理
public class LeaveBillAuditHandler implements ExcutionListener {
    public void notify(){
        // 获取请假单Id
        Long objId = execution.getVariable("objId");
        // 获取需要请假服务对象
        ILeaveBillService leaveBillService = get;
        // 调用请假服务对象上的审核逻辑, 完成请假单状态的修改
        leaveBillService.auditing(objId); 
    }
}

// leaveBillService
public void auditing(Long id) {
    LeaveBill bill = get(id);
    if(bill!=null) {
        bill.setState(2);
        update(bill);
    }
}
~~~~~~

# 多出口任务 #
1. 根据当前任务的出口集合`condition${outgoing=="驳回"}`, 动态生成按钮
2. 根据选择的按钮, 执行需要的流程
  1. 传入用户选择的决策信息
  2. 把决策的信息放入流程变量中
  3. 在规则流程中, 根据不同的流程出口, 指定流转条件

~~~~~~
// LeaveBillAction
public String audit(){
    List<String> transNames = leaveBillService.getTasksTransNames(taskId);
    putContext("transNames", transNames); // <submit value="${transname}" name="${tansname}">
}

// leaveBillService
public List<String> getTasksTransNames(taskId) {
    // 业务对象职责单一
    return workflowService.getTaskTransNames(taskId);
}

// workflowservice
public List<String> getTaskTransNames(taskId){
    List<String> list = new ArrayList<>();
    // 通过任务获取活动对象
    ActivityImpl acitivy = this.getActivityByTaskId(taskId);
    // 通过活动获取所有的出口信息集合
    List<PvmProcessElement> trans = acitvity.getOutgoingTransition();
    // 迭代所有出口信息集合, 获取出每个出口的name属性值
    for(PvmTransition trans : trans) {
         String name = (String) trans.getProperty("name");
         if(StringUtils.isNotBlank(name)){
             list.add(name);
         }
    }
    // 兼容, 如果集合到这里长度还是为0, 那么说明,
    // 当前任务的出口只有一个, 并且这个出口是没有name属性值的
    if(list.size()==0) {
        list.add("办理任务");
    }
    returnt list;
}
~~~~~~

# 完成任务 #

~~~~~~
// workflowAction
public String completeTask() {
    workflowService.completeTask(taskId, outGoing);
}

// workflowService
public String completeTask(String taskId, String outGoing){
    taskService.setVariable(taskId, "outGoing", outGoing);
    taskService.complete(taskId);
}

~~~~~~

# 批注添加 #

~~~~~~
public void addComment() {
    String taskId = "608";
    Task task = taskService.createTaskQeury().taskId(taskId).singleResult();
    // 设置用户上下文
    Authentication.setAuthenticatedUserId('张三');
    taskService.addComment(taskId, task.getProcessInstanceId(),"天气不错");
}
public void getComment() {
    String taskId = 607;
    List<Comment> taskComments = taskService.getTaskComments(taskId);
    // TODO
}
~~~~~~
