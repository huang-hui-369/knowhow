# dataverse 不仅是数据库而且提供web api
https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/

# Use Web API
通过web api的方式访问dataverse api

## Client-side JavaScript using Web API in model-driven apps.
https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/webapi/get-started-web-api-client-side-javascript

# use sdk
通过使用.net sdk的方式访问dataverse api


# Dataverse 创建自定义消息
以下两种功能都允许创建可从 Web 服务调用的新消息。自定义流程操作将继续提供一种无代码方式来声明性地定义同步业务逻辑。自定义流程操作一直是同步的“实时”工作流，因此不适合转换为使用 Power Automate

- 自定义流程操作
- Create and use Custom APIs

## Use Custom Process Actions with code
自定义流程操作的业务逻辑是使用工作流实现的。当您创建自定义流程操作时，关联的实时工作流会自动注册以在消息执行管道的主要操作阶段执行。

有两种使用代码扩展自定义流程操作的方法：使用自定义工作流活动或通过在阶段注册插件

### custome workflow activity
https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/workflow/workflow-extensions

### plugin

许多开发人员创建自定义流程操作只是为了创建新消息，而没有在工作流中实现任何逻辑。相反，它们为自定义操作创建的消息注册插件以实现其所有逻辑。自定义 API 功能使此模式成为开发人员扩展 Dataverse 的一流功能

https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/plug-ins


## Create and use Custom APIs
https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/custom-api

### Create a custom API

1. Define the Custom API
   自定义 API 创建了一条新消息，可以通过 Web API 或组织服务 SDK 调用该消息。

   有三种方式定义 a custom API

- By manually entering data in available forms. More information: Create a Custom API in the maker portal
- By using the web services, such as the Web API or Organization Service. 
- By editing solution files.

2. write the plug-in to implement it.
   编写插件来实现您的自定义 API 的主要操作与编写任何其他类型的插件没有什么不同，只是您不使用插件注册工具来设置特定步骤。

   **guide for write plug-in**

   https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/tutorial-write-plug-in

# modify-query-preoperation-stage
https://docs.microsoft.com/zh-cn/powerapps/developer/data-platform/org-service/samples/modify-query-preoperation-stage