## RepositoryService

### 流程定义和启动

* 管理流程定义文件xml以及静态资源的服务

* 对特定流程的暂停和激活

* 流程定义启动权限管理

  指定一个流程是哪些用户或者用户组的人可以启动

### 相关成员

* 部署文件构造器DeploymentBuilder
* 部署文件查询器DeploymentQuery
* 流程定义文件查询对象ProcessDefinitionQuery构建查询条件

### 可以获取的对象

* 流程部署文件对象Deployment

* 流程定义文件对象ProcessDefinition

* 流程定义的Java格式BpmnModel

  流程定义文件的Java格式

### 代码实例

#### 流程部署

```java
Deployment deploy = repositoryService.createDeployment()
                .addClasspathResource("diagram/holiday4.bpmn")//添加bpmn资源
                .addClasspathResource("diagram/holiday4.png") //添加png资源
                .name("请假流程-流程变量")//部署的名字
                .deploy();//执行部署
```

#### 流程定义查询

```java
ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();

        //4.设置条件，并查询当前定义的所有流程
        //查询条件：流程定义的key=holiday
//        processDefinitionQuery.deploymentId("");
//        processDefinitionQuery.processDefinitionNameLike("");
//        processDefinitionQuery.processDefinitionKey("");
        //返回结果：比如请教流程修改了三次，那么list的size就是3
//        processDefinitionQuery.listPage();
        List<ProcessDefinition> holidayList = processDefinitionQuery.processDefinitionKey("holiday")
                .orderByProcessDefinitionVersion()   //设置排序方式，根据流程定义的版本号进行排序
                .desc()
                .list();
```

#### 流程定义删除

```java
repositoryService.deleteDeployment(deploymentId);
```

#### 定义启动的用户和用户组

```java
            //设置和删除启动的用户或者用户组
repositoryService.addCandidateStarterUser(processDefinition.getId(),"userId");
repositoryService.addCandidateStarterGroup(processDefinition.getId(),"userGroup");
repositoryService.deleteCandidateStarterUser(processDefinition.getId(),"userId");
repositoryService.deleteCandidateStarterGroup(processDefinition.getId(),"userGroup");
```



## RuntimeService

流程控制，暂停，挂起，完成，查询正在运行的信息，流程数据获取

* 启动流程及对流程数据的控制
* 流程实例（ProcessInstance）与执行流（Execution）查询
* 触发流程操作、接收消息和信号

启动流程及变量管理

* 启动流程的常用方式（id，key，message）

  用key最关键，id每次部署都会变化

* 启动流程的可选参数

  businessKey，variables(key和value的上下文变量)，tenantId(多租户的关系)

* 流程变量（variables）的设置和获取

流程实例与执行流

* 流程实例（ProcessInstance）表示一次工作流业务的数据载体

* 执行流（Execution）表示流程实例中具体的执行路径

  如果是只有一条线，那么流程实例等于执行流

* 流程实例接口继承于执行流

### 代码实例

#### 启动流程实例

```java
//3.根据流程定义的key，创建流程实例。可以打开bpmn文件，在左边的id位置找到key，
        //3.1 创建流程实例方式1：通过key进行启动
        // 也可以在数据的`act_re_procdef`表的KEY_列中找到。
//        ProcessInstance instance = runtimeService.startProcessInstanceByKey("holiday");

        //3.2 创建流程实例方式2：通过key进行启动，且设置业务id
        //第一个参数是部署id，第二个参数是act_ru_execution表中的业务id，用于关联业务表的记录id
//        ProcessInstance instance = runtimeService.startProcessInstanceByKey("holiday", "1001");

        //3.3 创建流程实例方式3: 通过key启动，且设置流程变量
        //设置Assignee UEL的值，在bpmn的流程图（见holiday2.bpmn），各个流程的执行人分别是${assignee0},${assignee1},${assignee2}
//        Map<String,Object> map = new HashMap<>();
//        map.put("assignee0","zhangsan");
//        map.put("assignee1","lishi");
//        map.put("assignee2","wangwu");
////        ProcessInstance instance = runtimeService.startProcessInstanceByKey("holiday2", map);
//        //第二个参数是businessKey
//        ProcessInstance instance = runtimeService.startProcessInstanceByKey("holiday2","10001", map);

        //3.4 创建流程实例方式4：通过key启动，且设置流程变量，且流程变量是java对象类型
        //设置条件的UEL值，在bpmn的流程图（见holiday4.bpmn），在流程连线上设置${holiday.num<=3}，${holiday.num>3}
        String key = "myProcess_1"; //holiday4.bpmn的流程定义id
        Map<String,Object> map = new HashMap<>();
        Holiday holiday = new Holiday();
        holiday.setNum(1F);
        map.put("holiday", holiday);
        //在实例启动的时候设置流程变量
        //通过部署id进行启动
//        runtimeService.startProcessInstanceById("id")
        //通过key进行启动
        ProcessInstance instance = runtimeService.startProcessInstanceByKey(key,"10002", map);
	
	    //3.4 创建流程实例方式5：通过instanceBuilder进行启动
        ProcessInstanceBuilder processInstanceBuilder = runtimeService.createProcessInstanceBuilder();
        ProcessInstance instance2 = processInstanceBuilder.businessKey("businessKey001")
                .processDefinitionKey("process")
                .variables(map)
                .start();
```

#### 启动和暂停

```java
ProcessInstance processInstance = processInstanceQuery.processInstanceId("12511").singleResult();
        //是否被暂停， true：是，false：否
        boolean suspended = processInstance.isSuspended();
        String instId = processInstance.getId();
        if(suspended){
            //激活指定部署id的全部实例
            runtimeService.activateProcessInstanceById(instId);//也可以用key，但是我们已经能获取到id
            System.out.println("流程定义实例："+instId+"激活");
        }else{
            //暂停指定部署id的全部实例
            runtimeService.suspendProcessInstanceById(instId);
            System.out.println("流程定义实例："+instId+"挂起（暂停）");
        }
```

#### 获取流程变量

#### 流程触发

使用trigger触发ReceiveTask节点，两种触发方式：

* 触发信号捕获事件signalEventReceived

  全局的

* 触发消息捕获事件messageEventReceived

  流程实例的

## TaskService

运行中的人工task，可以指定权限，指定的用户和用户组，同时也可以设置和获取上下文用户变量

## IdentityService

对用户或者用户组的管理，创建用户和用户组，维护用户和用户组之间的关系

## FormService

解析流程定义中的表单，对数据格式进行渲染，启动表单和usertask表单

## HistoryService

提供对运行结束的流程实例的查询功能，也提供了流程维度和用户任务维度的删除操作，方便统计流程执行中的变化，也可以获取执行过程中的过程数据和快照数据

## ManagementService

对流程的基础管理，一般使用比较少，一些job的管理，还提供一些操作，可以获取数据库表名和表结构的一些方法。

## DynamicBpmService

6.0新增的Service，可以对动态的流程定义模型做修改，不推荐使用

