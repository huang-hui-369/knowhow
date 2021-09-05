# How to build Responsive Power App＃
https://www.youtube.com/watch?v=1o2L0DADzKQ

![](img/2021-09-05-22-20-40.png)

![](img/2021-09-05-22-22-19.png)

display --> Scale to fit --> off
![](img/2021-09-05-22-19-58.png)

insert --> search container
![](img/2021-09-05-22-25-01.png)

flexable width --> 会自动按最大填充width
![](img/2021-09-05-22-30-20.png)

![](img/2021-09-05-22-34-14.png)

customize size
app --> advanced
![](img/2021-09-05-22-36-38.png)

Parent.width

# navigate form
https://www.youtube.com/watch?v=9gI9OscTLD0

迁移到指定页面
![](img/2021-09-05-22-42-31.png)

## 设置FormModel new or edit
![](img/2021-09-05-23-16-42.png)

## new，edit，view 都用同一个Form

### new
设置 FormModel 为 edit
设置 form item的数据源来源于变量,这样的话，当数据不存在时，就相当于创建了一条数据
(Form item 决定了要显示的数据的来源)
在button 按下时，设置变量 = 默认的数据源中的数据
![](img/2021-09-05-22-50-07.png)

![](img/2021-09-05-22-47-26.png)

### view
当点击view button时，设置变量 = ThisItem 来获取数据
![](img/2021-09-05-23-33-45.png)

这样没有办法区分是viewModel还是editModel
设置变量 varFromModel 来确定是viewModel还是editModel
当点击view button时，设置变量 varFromModel = FormMode.View
![](img/2021-09-05-23-45-38.png)

当点击edit，new button时 设置变量 varFromModel = FormMode.Edit

### 设置FormModel=变量
![](img/2021-09-05-23-43-46.png)


## 如果验证不通过设置button不可用
![](img/2021-09-05-22-58-04.png)

回退到前一页面
![](img/2021-09-05-23-00-37.png)

迁移前，resetForm
![](img/2021-09-05-23-03-47.png)

将三个form的数据更新到db
![](img/2021-09-05-23-07-00.png)

确认更新成功或者失败
![](img/2021-09-05-23-08-42.png)

# Power Apps form data validation 
https://www.youtube.com/watch?v=XvevCq6qhX4

# Form model
https://www.youtube.com/watch?v=BnzaSDYl8mA

# PowerApps Left Navigation Component
https://www.youtube.com/watch?v=dP74npyyvGc

# Power Apps Navigation Menu Component (2 level menu)
https://www.youtube.com/watch?v=3S0h2nODcxM






![](img/2021-09-05-22-13-05.png)

![](img/2021-09-05-22-13-37.png)

