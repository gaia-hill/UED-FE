#### createDocumentFragment()

作用：可以创建html片段，一次性插入dom中，减少js操作dom的次数，可提升dom操作的渲染效率



下面代码为在ul中添加20000个li：

```javascript
var frag=document.createDocumentFragment();
for(var i=0;i<20000;i++){
  var li=document.createElement("li");
  li.innerHTML="item"+i;
  frag.appendChild(li);
}
document.getElementById("ul").appendChild(frag);
```

frag为定义的HTML片段，将创建的li全部放到frag中之后，再一次性append到html中，只触发一次HTML渲染；