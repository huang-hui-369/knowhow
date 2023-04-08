### Making connections

A connection lets one object know where another object is in memory so that the two objects can communicate. There are two kinds of connections that you can make in Interface Builder: outlets and actions. An outlet is a reference to an object. An action is a method that gets triggered by a button or some other view that the user can interact with, like a slider or a picker.

#### 设置引用label的outlets

1. ####  click  adjus Editor Options then in popup menu click Assistant，就会显示ViewController.swift

   #### ![image-20221208154019843](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208154019843.png)

   ![image-20221208153633839](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208153633839.png)

2. 鼠标右键点击label1在popup menu中选择 Referencing Outlets,drag it to source

   ![image-20221208153826147](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208153826147.png)

   就会生成对label1的引用，就可以在ViewController中访问label1

   ![image-20221208154432754](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208154432754.png)

#### 设置button action

#### 方法一

1. 在storyboard中选中button 按下control键同时drag button 到 ViewController source

   ![image-20221208171701864](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208171701864.png)

2. 在popUp menu中

   connection 选择 Action

   name 栏 输入 button click 函数名

   ![image-20221208171939286](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208171939286.png)

   就会在ViewController 生成 func buttonClick

   ![image-20221208172131045](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208172131045.png)

方法二

1. 在ViewController中加入以下代码

   ```swift
   @IBAction func showNextQuestion(_ sender: UIButton) {
   
   }
   ```

2. 在storyBoard中选中button ，按下control键同时drag button 到 ViewController（storyBoard中）在popup menu中选择showNextQuestion 函数

   ![image-20221208172507433](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208172507433.png)

###  Checking connections in the connections inspector

![image-20221208172820657](D:\github\knowhow\ios-app\tutorial\firstApp\005-ib-MakeConnections.assets\image-20221208172820657.png)

