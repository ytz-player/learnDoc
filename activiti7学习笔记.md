# activiti7学习笔记

### 数据表的命名规范

* **act_hi_***：'hi’表示 history，此前缀的表包含历史数据，如历史(结束)流程实例，变量，任务等等。
* **act_ge_***：'ge’表示 general，此前缀的表为通用数据，用于不同场景中。
* **act_evt_***：'evt’表示 event，此前缀的表为事件日志。
* **act_procdef_***：'procdef’表示 processdefine，此前缀的表为记录流程定义信息。
* **act_re_***：'re’表示 repository，此前缀的表包含了流程定义和流程静态资源(图片，规则等等)。
* **act_ru_***：'ru’表示 runtime，此前缀的表是记录运行时的数据，包含流程实例，任务，变量，异步任务等运行中的数据。Activiti只在流程实例执行过程中保存这些数据，在流程结束时就会删除这些记录。

### 数据表分类

* 通用数据(act_ge_)

  |      表名       |                             解释                             |
  | :-------------: | :----------------------------------------------------------: |
  | act_ge_bytearry |          二进制数据表，存储通用的流程定义和流程资源          |
  | act_ge_property | 系统相关属性，属性数据表存储整个流程引擎级别的数据，初始化表结构时，会默认插入三条记录。 |

  

* 流程定义表(act_re_)

  |       表名        |        解释        |
  | :---------------: | :----------------: |
  | act_re_deployment |     部署信息表     |
  |   act_re_model    | 流程设计模型部署表 |
  |  act_re_procdef   |   流程定义数据表   |

* 运行实例表(act_ru_)

  |       **表名**        |               **解释**               |
  | :-------------------: | :----------------------------------: |
  | act_ru_deadletter_job | 作业死亡信息表，作业失败超过重试次数 |
  |  act_ru_event_subscr  |             运行时事件表             |
  |   act_ru_execution    |         运行时流程执行实例表         |
  |  act_ru_identitylink  |           运行时用户信息表           |
  |  act_ru_integration   |             运行时积分表             |
  |      act_ru_job       |           运行时作业信息表           |
  | act_ru_suspended_job  |           运行时作业暂停表           |
  |      act_ru_task      |           运行时任务信息表           |
  |   act_ru_timer_job    |          运行时定时器作业表          |
  |    act_ru_variable    |           运行时变量信息表           |

  

* 历史流程表(act_hi_)

  |      **表名**       |            **解释**            |
  | :-----------------: | :----------------------------: |
  |   act_hi_actinst    |           历史节点表           |
  |  act_hi_attachment  |           历史附件表           |
  |   act_hi_comment    |           历史意见表           |
  |    act_hi_detail    | 历史详情表，提供历史变量的查询 |
  | act_hi_identitylink |       历史流程用户信息表       |
  |   act_hi_procinst   |         历史流程实例表         |
  |   act_hi_taskinst   |         历史任务实例表         |
  |   act_hi_varinst    |           历史变量表           |

  

* 其他表

  |     **表名**     |           **解释**           |
  | :--------------: | :--------------------------: |
  |   act_evt_log    | 流程引擎的通用事件日志记录表 |
  | act_procdef_info |    流程定义的动态变更信息    |

  ​		**——以上原文链接：https://blog.csdn.net/zhouchenjun001/article/details/103629559**

### processRuntime启动流程实例

````java
StartProcessPayload startProcessPayloadBuilder = ProcessPayloadBuilder
                    .start()
                    .withProcessDefinitionKey("test")
                    .withBusinessKey("123456")
                    .withName("testName")
                    .build();
            processInstance = processRuntime.start(startProcessPayloadBuilder);
````

相关表：

* ACT_RE_PROCDEF

  ````sql
  Preparing: select distinct RES.* from ACT_RE_PROCDEF RES WHERE RES.KEY_ = ? order by RES.APP_VERSION_ desc LIMIT ? OFFSET ? 
  Parameters: test(String), 2147483647(Integer), 0(Integer)
  Total: 1
  ````

  > 根据**processDefinitionKey**查询流程定义，获取流程id

* ACT_RE_DEPLOYMENT

  ````sql
  Preparing: select * from ACT_RE_DEPLOYMENT where VERSION_ = (select max(VERSION_) from ACT_RE_DEPLOYMENT) and NAME_ = ? and PROJECT_RELEASE_VERSION_ IS NOT NULL 
  Parameters: SpringAutoDeployment(String)
  Total: 1
  ````

  > 查询流程部署 本例为Spring自动部署

* ACT_RE_PROCDEF

  ````sql
  Preparing: select * from ACT_RE_PROCDEF where ID_ = ? 
  Parameters: test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String)
  Total: 1
  ````

  > 根据流程id，查询流程定义

* ACT_HI_TASKINST

  ````sql
  Preparing: insert into ACT_HI_TASKINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, NAME_, PARENT_TASK_ID_, DESCRIPTION_, OWNER_, ASSIGNEE_, START_TIME_, CLAIM_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TASK_DEF_KEY_, FORM_KEY_, PRIORITY_, DUE_DATE_, CATEGORY_, TENANT_ID_ ) values ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
  Parameters: 45f65ce2-3ac3-11ec-8126-000c29ca7ff6(String), test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cf-3ac3-11ec-8126-000c29ca7ff6(String), process_01(String), null, 审批流程01(String), null, 00001(String), 2021-11-01 11:24:50.845(Timestamp), null, null, null, null, sid-3a0616ea-3151-430d-a136-70c3390f5f2d(String), null, 50(Integer), null, null, (String)
  Updates: 1
  ````

  > 添加一条历史任务数据

* ACT_HI_PROCINST

  ````sql
  Preparing: insert into ACT_HI_PROCINST ( ID_, PROC_INST_ID_, BUSINESS_KEY_, PROC_DEF_ID_, START_TIME_, END_TIME_, DURATION_, START_USER_ID_, START_ACT_ID_, END_ACT_ID_, SUPER_PROCESS_INSTANCE_ID_, DELETE_REASON_, TENANT_ID_, NAME_ ) values ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
  Parameters: 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 123456(String), test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), 2021-11-01 11:24:50.844(Timestamp), null, null, other(String), sid-f8cce656-623a-4f08-88eb-a40e68883a07(String), null, null, null, (String), testName(String)
  Updates: 1
  ````

  > 添加一条历史流程数据

* ACT_HI_ACTINST

  ````sql
  Preparing: insert into ACT_HI_ACTINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, ACT_ID_, TASK_ID_, CALL_PROC_INST_ID_, ACT_NAME_, ACT_TYPE_, ASSIGNEE_, START_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TENANT_ID_ ) values (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) , (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) 
  Parameters: 45f65ce0-3ac3-11ec-8126-000c29ca7ff6(String), test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cf-3ac3-11ec-8126-000c29ca7ff6(String), sid-f8cce656-623a-4f08-88eb-a40e68883a07(String), null, null, null, startEvent(String), null, 2021-11-01 11:24:50.845(Timestamp), 2021-11-01 11:24:50.845(Timestamp), 0(Long), null, (String), 45f65ce1-3ac3-11ec-8126-000c29ca7ff6(String), test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cf-3ac3-11ec-8126-000c29ca7ff6(String), sid-3a0616ea-3151-430d-a136-70c3390f5f2d(String), 45f65ce2-3ac3-11ec-8126-000c29ca7ff6(String), null, process_01(String), userTask(String), 00001(String), 2021-11-01 11:24:50.845(Timestamp), null, null, null, (String)
  Updates: 2
  ````

  > 添加两条历史节点数据
  >
  > 	1. 开始节点
  >  	2. 用户节点

* ACT_HI_IDENTITYLINK

  ````sql
  Preparing: insert into ACT_HI_IDENTITYLINK (ID_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_) values (?, ?, ?, ?, ?, ?) , (?, ?, ?, ?, ?, ?) 
  Parameters: 45f635ce-3ac3-11ec-8126-000c29ca7ff6(String), starter(String), other(String), null, null, 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 45f683f3-3ac3-11ec-8126-000c29ca7ff6(String), participant(String), 00001(String), null, null, 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String)
  Updates: 2
  ````

  > 添加两条历史用户信息数据
  >
  > 	1. 开始动作
  >  	2. 用户动作

* ACT_RU_EXECUTION

  ````sql
  Preparing: insert into ACT_RU_EXECUTION (ID_, REV_, PROC_INST_ID_, BUSINESS_KEY_, PROC_DEF_ID_, ACT_ID_, IS_ACTIVE_, IS_CONCURRENT_, IS_SCOPE_,IS_EVENT_SCOPE_, IS_MI_ROOT_, PARENT_ID_, SUPER_EXEC_, ROOT_PROC_INST_ID_, SUSPENSION_STATE_, TENANT_ID_, NAME_, START_TIME_, START_USER_ID_, IS_COUNT_ENABLED_, EVT_SUBSCR_COUNT_, TASK_COUNT_, JOB_COUNT_, TIMER_JOB_COUNT_, SUSP_JOB_COUNT_, DEADLETTER_JOB_COUNT_, VAR_COUNT_, ID_LINK_COUNT_, APP_VERSION_) values (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) , (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) 
  Parameters: 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 123456(String), test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), null, true(Boolean), false(Boolean), true(Boolean), false(Boolean), false(Boolean), null, null, 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 1(Integer), (String), testName(String), 2021-11-01 11:24:50.844(Timestamp), other(String), false(Boolean), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 1(Integer), 45f635cf-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), null, test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), sid-3a0616ea-3151-430d-a136-70c3390f5f2d(String), true(Boolean), false(Boolean), false(Boolean), false(Boolean), false(Boolean), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), null, 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), 1(Integer), (String), null, 2021-11-01 11:24:50.844(Timestamp), null, false(Boolean), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 1(Integer)
  Updates: 2
  ````

  > 添加两条运行时流程实例数据

* ACT_RU_TASK

  ````SQL
  Preparing: insert into ACT_RU_TASK (ID_, REV_, NAME_, BUSINESS_KEY_, PARENT_TASK_ID_, DESCRIPTION_, PRIORITY_, CREATE_TIME_, OWNER_, ASSIGNEE_, DELEGATION_, EXECUTION_ID_, PROC_INST_ID_, PROC_DEF_ID_, TASK_DEF_KEY_, DUE_DATE_, CATEGORY_, SUSPENSION_STATE_, TENANT_ID_, FORM_KEY_, CLAIM_TIME_, APP_VERSION_) values (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
  Parameters: 45f65ce2-3ac3-11ec-8126-000c29ca7ff6(String), process_01(String), 123456(String), null, 审批流程01(String), 50(Integer), 2021-11-01 11:24:50.845(Timestamp), null, 00001(String), null, 45f635cf-3ac3-11ec-8126-000c29ca7ff6(String), 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), test:1:dad6c495-3ac1-11ec-8126-000c29ca7ff6(String), sid-3a0616ea-3151-430d-a136-70c3390f5f2d(String), null, null, 1(Integer), (String), null, null, 1(Integer)
  Updates: 1
  ````

  > 添加一条运行时任务数据

* ACT_RU_IDENTITYLINK

  ````sql
  Preparing: insert into ACT_RU_IDENTITYLINK (ID_, REV_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_, PROC_DEF_ID_) values (?, 1, ?, ?, ?, ?, ?, ?) , (?, 1, ?, ?, ?, ?, ?, ?) 
  Parameters: 45f635ce-3ac3-11ec-8126-000c29ca7ff6(String), starter(String), other(String), null, null, 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), null, 45f683f3-3ac3-11ec-8126-000c29ca7ff6(String), participant(String), 00001(String), null, null, 45f635cd-3ac3-11ec-8126-000c29ca7ff6(String), null
  Updates: 2
  ````

  > 添加两条运行时用户信息数据
  >
  >  	1. 开始动作
  >  	2. 用户动作

### 任务列表查询

````java
taskRuntime.tasks(Pageable.of(0, 10));
````

相关表：

* ACT_RU_TASK

* ACT_RU_IDENTITYLINK

  ````sql
  Preparing: select distinct RES.* from ACT_RU_TASK RES left join ACT_RU_IDENTITYLINK I_OR0 on I_OR0.TASK_ID_ = RES.ID_ WHERE ( RES.OWNER_ = ? or (RES.ASSIGNEE_ = ? or (RES.ASSIGNEE_ is null and I_OR0.TYPE_ = 'candidate' and (I_OR0.USER_ID_ = ? or I_OR0.GROUP_ID_ IN ( ? ) ))) ) order by RES.ID_ asc LIMIT ? OFFSET ? 
  Parameters: other(String), other(String), other(String), otherTeam(String), 10(Integer), 0(Integer)
  Total: 0
  ````

  > 查询**运行时任务表**左联**运行时用户信息表**

