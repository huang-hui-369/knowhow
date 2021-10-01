可以使用 Power Automate 来创建逻辑，以便在画布应用中发生某个事件时执行一个或多个任务。 例如，配置一个按钮，以便用户选择它时在 SharePoint 列表中创建一个项、发送电子邮件或会议请求、将文件添加到云，或执行所有上述操作。 可以在应用中配置任何用于启动流的控件，该控件在关闭 Power Apps 的情况下仍会继续运行。

**当用户从应用中运行流时，该用户必须具有执行流中指定的任务的权限。 否则，流将失败。**

https://docs.microsoft.com/zh-cn/powerapps/maker/canvas-apps/using-logic-flows

为按钮选择 OnSelect 属性

FlowInApp.Run(TextInput1.Text)

# フローの種類

- ワークフロー
- business フロー
- UIフロー AI