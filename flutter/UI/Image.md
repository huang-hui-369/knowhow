Image

File imgFile = File(filepath);
假设 filepath /tmp/a.jpg
Image.file(imgFile);

现象
如果更新了/tmp/a.jpg，但是flutter的Image.file(imgFile)却不更新

原因
我查看到，Image.file 实际上会将 image 设置为 FileImage 这个 ImageProvider。FileImage 的代码中，在进行 operator 时，只判断了文件路径和缩放比例。正是因为如此，我们文件路径不变，缩放比例不变时，Flutter会认为我们还是用的原图，并不会重新进行加载。

解决办法
可以利用Image.memory(List<int> _data);

```
List<int> _data;

 void initState() {
   File file = File(filepath);
    o = await file.open(mode: FileMode.read);
    // response.data is List<int> type
    _data = await o.read(file.lengthSync());
  }

  Widget build(BuildContext context) {

    child: Image.memory(_data),

  }

```