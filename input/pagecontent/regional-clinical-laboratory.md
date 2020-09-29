### 业务背景

&emsp;&emsp;区域临床检验中心是在建立和完善检验质量管理和控制的基础上，以医疗服务机构为主体，以实验室资源和检验信息共享为目标，集成共性技术及医疗服务关键技术，建立区域协同临床检验中心，提供常规检验、生化检验、免疫检验、微生物检验、分子检验等设备结果数据采集、结果报告书写、发布与回传。支持接收外来检验样本进行检验，实现检验标本跨机构管理。实现区域内检验结果互认。


### 场景简介

&emsp;&emsp;远程申请及样本采集管理、样本管理、设备数据采集、结果报告管理、查询统计等。


### 场景流程

#### 定义流程的资源

&emsp;&emsp;场景所涉及的流程及流程中的活动是通过以下两个FHIR的资源来定义的：

- 活动定义：[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)：活动定义资源，定义在业务流程中每一个活动步骤，描述其在流程中的作用。
- 流程定义：[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)：通过计划义定资源可以对活动定义资源进行组装，并描述活动之间的先后关系，触发条件等信息，该资源可描述一个完整的业务流程。

#### 场景中的活动与流程的定义

区域临床检验分为“检验申请单”、“检验结果回传”两个部分，
使用[ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)资源分别定义如下：

- [检验申请单-ActivityDefinition定义](ActivityDefinition-ad-regional-clinical-laboratory.html)
- [检验结果回传-ActivityDefinition定义](ActivityDefinition-ad-regional-clinical-laboratory-response.html)

> 使用[PlanDefinition](http://www.hl7fhir.cn/R4/plandefinition.html)资源定义区域临床检验的业务流程：
[区域临床检验-PlanDefinition定义](PlanDefinition-pd-regional-clinical-laboratory.html)，
它通过其Action元素将以上两个步骤组装起来。每个Action关联一个步骤。可以通过此流程定义资源实例实现流程自动化。
临床开单系统发起申请单，区域临床检验系统回传结果报告

#### 活动实例资源
 [ActivityDefinition](http://www.hl7fhir.cn/R4/activitydefinition.html)活动定义中有一个kind元素，此元素描述了该活动具体使用哪种资源类型,即活动实例资源类型。
比如[检验申请单-ActivityDefinition定义](ActivityDefinition-ad-regional-clinical-laboratory.html)的活动实例资源类型为继承自ServiceRequest资源类型的[LaboratoryRequest](StructureDefinition-regional-clinical-laboratory.html)资源。
此场景中涉及的活动实例资源如下：

- [LaboratoryRequest](StructureDefinition-regional-clinical-laboratory.html):检验申请单资源，该资源描述检验申请单。
- [LaboratoryReport](StructureDefinition-regional-clinical-laboratory.html):检验结果回传资源，该资源描述检验结果。

上述流程定义、活动定义、活动实例资源的关系图如“图 1-区域临床检验流程定义关系图”
![流程定义](PlanDefinition-ActivityDefinition-Task-Relationship.png)

> 流程启动后，每个活动步骤将产生一个活动实例资源的新实例，其示例如下：

- [检验申请单示例](Appointment-HospitalReferral-example.html)
- [检验结果回传示例](AppointmentResponse-HospitalReferralResponse-example.html)


### 数据交换方式与结构

FHIR的数据交换方式支持RESTful、SOA、消息交换等。本场景目前只定义了消息交换和RESTful的方式，其他方式在进一步定义中。

#### 消息交换数据报文结构

[FHIR消息交换框架](http://www.hl7fhir.cn/R4/messaging.html)要求的应用程序符合“ FHIR消息传递”，它支持大多数消息组件、ESB等消息引擎，支持异步消息和消息应答机制。

数据交换使用的消息体（报文）通过[Bundle](http://www.hl7fhir.cn/R4/bundle.html)（资源捆束）组装数据。
本场景中所有的数据交换使用的消息体结构遵循以下规则：

- 1、资源捆束中的“Bundle.type”元素固定为“message”。
- 2、资源捆束内的第一个条目(entry)中的资源必须是[MessageHeader](http://www.hl7fhir.cn/R4/messageheader.html)（消息头）。
- 3、资源捆束内的第二个条目中的资源必须是描述关于业务流程活动步骤的活动实例资源。
- 4、资源捆束内的其它条目中应该包含对应步骤的相关业务资源。

> 资源捆束里的各条目中的资源的相互关系是通过消息头引用活动实例资源，活动实例资源再关联业务资源来展现的。示意图如 “图 2-消息交换数据报文结构图”

![数据结构](structure-bundle.png)

资源捆束中条目中的资源类型在不同的业务活动（步骤）中是不同的，
本规范集使用[MessageDefinition](http://www.hl7fhir.cn/R4/messagedefinition.html)定义每个活动数据交换的消息体结构，
遵循[住院双转流程定义](PlanDefinition-pd-regional-clinical-laboratory.html)中的Action元素对应的步骤定义。每个活动步骤的数据交换消息体结构的定义如下：

- 一、[检验申请单-MessageDefinition定义](MessageDefinition-md-regional-clinical-laboratory.html)：
在临床开单系统发起申请单时使用的消息体结构的定义。
- 二、[检验结果回传-MessageDefinition定义](MessageDefinition-md-regional-clinical-laboratory-response.html)：
区域临床检验返回结果的消息体结构的定义。

#### RESTful数据报文结构

[FHIR-RESTful API框架](http://www.hl7fhir.cn/R4/messaging.html)要求的应用程序符合“RESTful规范”，在这个RESTful框架中，事务是使用HTTP请求/响应机制直接在服务器资源上执行的。 API不直接负责身份验证、授权和审计收集——有关更多信息，请参阅[安全页面](http://www.hl7fhir.cn/R4/security.html)。 这里介绍的交互都是指的同步模式，异步模式的使用见[异步使用模式](http://www.hl7fhir.cn/R4/async.html)。

RESTful API请求数据通过[Bundle](http://www.hl7fhir.cn/R4/bundle.html)（资源捆束）组装数据。
本场景中所有的数据交换使用的消息体结构遵循以下规则：

- 1、资源捆束中的“Bundle.type”元素固定为“transaction”或者“transaction-response”。
- 2、资源捆束内的第一个条目中的资源必须是描述关于业务流程活动步骤的活动实例资源，该资源必须符合[区域临床检验流程定义](PlanDefinition-pd-regional-clinical-laboratory.html)中的Action元素对应的步骤定义中约束的结构定义规范。
- 3、资源捆束内的其它条目中应该包含对应步骤的相关业务资源。

<span id="coderestful"></span>

> 资源捆束里的各条目中的资源的相互关系是通过活动实例资源关联业务资源来展现的。示意图如 “图 3-RESTful数据报文结构图”

![数据结构](structure-RESTful-bundle.png)

资源捆束中条目中的资源类型在不同的业务活动（步骤）中是不同的，
本规范集遵循[区域临床检验流程定义](PlanDefinition-pd-regional-clinical-laboratory.html)中的Action元素对应的步骤定义。

#### 相关业务资源  

在本场景中数据交换所涉及的相关业务资源（即资源捆束中可能包含的业务资源）如下：

- [Patient](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Patient.html)：
患者资源，接受医疗健康服务的个人或动物，医疗过程是以患者为中心的。对交叉索进行中国本地化约定。
- [Practitioner](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Practitioner.html)：
医护人员资源，直接或间接参与提供医疗健康服务的人员。
- [PractitionerRole](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-PractitionerRole.html)：
医护人员岗位资源，医务人员提供医疗服务时的岗位相关信息，包括所属组织、科室、角色/岗位等。
- [Hospital](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Hospital.html)：
医院资源，描述医疗机构（医院）的基本信息。
- [EndemicArea](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-EndemicArea.html)：
病区资源，描述医院病区的基础信息。
- [Department](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Department.html)：
科室/部门资源，描述医院科室/部门的基础信息。
- [Ward](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Ward.html)：
病房资源，描述医院病房的基础信息。
- [Sickbed](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Sickbed.html)：
病床资源，描述医院病床的基础信息。
- [Encounter](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Encounter.html)：
就诊资源，描述患者就诊信息。
- [Specimen](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-Specimen.html)：
标本资源，描述实验室样本的信息。
- [ServiceRequest](https://build.fhir.org/ig/HL7China/CN-CORE-R4/StructureDefinition-ServiceRequest.html)：
检验申请资源，描述检验申请单信息。

> 资源捆束中包含的活动实例资源和业务资源相互关联以准确表达业务流程状态和业务数据信息，其相互关系图如 “图 4-业务资源类图”

![业务类图](Class.png)


### 数据交互流程

数据交互流程遵循[区域临床检验流程定义](PlanDefinition-pd-regional-clinical-laboratory.html)定义的流程规则和步骤，采用消息交换方式或者RESTful方式交换数据。具体流程 如 “图 5-区域临床检验流程图”

![流程图](sequence.png)

#### 消息交互方式的流程
1. 临床开单系统发起检验申请单。
该步骤符合流程定义“区域临床检验流程定义”中的第一个action节点定义的步骤-检验申请单，
按照[检验申请单消息定义](MessageDefinition-md-regional-clinical-laboratory.html)的消息结构使用
“Bundle”组装数据并发送。
具体示例参见： [检验申请单消息交换示例](Bundle-regional-clinical-laboratory-example.html)。
2. 区域临床检验系统回传检验结果。
该流程符合流程定义“区域临床检验流程定义”中的第二个action节点定义的步骤-检验结果回传，
按照[检验结果回传消息定义](MessageDefinition-md-regional-clinical-laboratory-response.html)的消息结构使用
“Bundle”组装数据并发送。
具体示例参见：[检验结果回传消息交换示例](Bundle-regional-clinical-laboratory-response-example.html)。

#### RESTful交互方式的流程

1. 临床开单系统发起检验申请单。
该步骤符合流程定义“区域临床检验流程定义”中的第一个action节点定义的步骤-检验申请单，按照[3.1.4.2-RESTful数据报文结构章节中的“图 3-RESTful数据报文结构图”](#coderestful) 结构定义使用“Bundle”组装数据并发起RESTful请求。
具体示例参见： [检验申请单RESTful接口的Bundle示例](Bundle-regional-clinical-laboratory-RESTful-example.html)。
2. 区域临床检验系统回传检验结果。
该流程符合流程定义“区域临床检验流程定义”中的第二个action节点定义的步骤-检验结果回传，按照[3.1.4.2-RESTful数据报文结构章节中的“图 3-RESTful数据报文结构图”](#coderestful) 结构定义使用“Bundle”组装数据并发起RESTful请求。
具体示例参见：[检验结果回传RESTful接口的Bundle示例](Bundle-regional-clinical-laboratory-response-RESTful-example.html)。
