# 1. jquery selctor

## 1. 判断能不能获取到jquery的对象时，不能用null判断，要用length>0判断

```js
let hitem = $("input:text[name='uname']");
if(hitem.length>0) {
	// got html item
	alert(hitem.val())
}
// or 
if(hitem[0]) {
	// got html item
	alert(hitemval())
}
```



# 2. html checkbox

```html
<!-- 1. get checkbox value
-->
<input type="checkbox" name="bike" id="bike" > I have a bike
<!-- get defaule checkbox value
checkbox如果不指定vlue的话，默认的value为"on",这和check不check没关系
alert(document.getElementById("bike").value);   on
$("input:checkbox[name='bike']").val()  -- jquery
$("[name='bike']").val()  -- jquery
$("#bike]").val()  -- jquery
-->
<input type="checkbox" name="bike" id="bike" value="bike"> I have a bike
<!-- get checkbox value
如果指定vlue的话，则为value的值
alert(document.getElementById("bike").value);   bike
-->

<!-- 2. get checkbox checked
-->
<input type="checkbox" name="bike" id="bike" > I have a bike
<!-- get checkbox attribute checked
获取input的属性直接.后跟属性
alert(document.getElementById("bike").checked);   false
$("#bike]).prop('checked')"  -- jquery
-->
<input type="checkbox" name="bike" id="bike" checked> I have a bike
<!-- get checkbox attribute checked
获取input的属性直接.后跟属性
alert(document.getElementById("bike").checked);   true
-->

<!-- 3. set checkbox checked
-->
<input type="checkbox" name="bike" id="bike" > I have a bike
<!-- set checkbox attribute checked
设置checked为true
document.getElementById("bike").checked = 'true';
$("#bike]).prop('checked', true)"  -- jquery
alert(document.getElementById("bike").checked);   true
-->
<!-- set checkbox attribute unchecked
document.getElementById("bike").checked = '';
$("#bike]).prop('checked', false)"  -- jquery
alert(document.getElementById("bike").checked);   false
-->
```

```html
<!-- 复数的checkbox -->
<input type="checkbox" name="intrest" id="bike" > I have a bike
<input type="checkbox" name="intrest" id="car" > I have a car
<input type="checkbox" name="intrest" id="book" > I have a book
```



```js
// 考虑到复数的checkbox，使用jquery可以用以下方法
// set checkbox
$("[name='intrest']").val(["bike","car"]);

// get checkbox
let checkedItems = $("[name='intrest']:checked");
// 不支持forEach()方法
// 使用for(let item in checkedItems) ，可枚举的属性值太多了
for(let i=0;i<checkedItems.length;i++){
    alert($(checkedItems[i]).val());
};
```



# 3. html radio

```html
<input type="radio" name="gender" value="1">male
<br>
<input type="radio" name="gender" value="2">female

```

```js
// 基本上和checkbox用法一样 属性 checked="true" 就是选中
// 1. get radio value
let items = document.getElementsByName("gender");
alert(items.length);
alert(items[0].checked);
alert(items[1].checked);
// jquery
alert($("[name='gender']:checked").val());

// 2. set raio value
let items = document.getElementsByName("gender");
items[0].checked = "true" // 设置male为checked
items[1].checked = ""
// jquery
$("[name='gender']").val(["1"]); // 设置male为checked 注意val()的参数必须为数组（因为可以取出复数的元素）
```

# 4. outerHtml

```js
$("#elm").prop(‘outerHTML’);
```

