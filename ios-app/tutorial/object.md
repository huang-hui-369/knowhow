# Everything Is an Object in swift

Swift has three kinds of object type: classes, structs, and enums

`1` is an instance of a struct

# Modules

顶级命名空间是模块。您的应用程序是一个模块，因此是一个命名空间；默认情况下，命名空间的名称是应用程序的名称。如果我的应用程序名为 MyApp，那么如果我在文件的顶层声明一个类 Manny，则该类的真实名称是 MyApp.Manny。但我通常不需要使用那个真实姓名，因为我的代码已经在同一个命名空间中，并且可以直接看到 Manny 这个名字。