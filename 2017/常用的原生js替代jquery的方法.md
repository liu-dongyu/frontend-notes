参考网址：[u dont need jq](http://blog.garstasio.com/you-dont-need-jquery/)

#### dom ready

- jquery

```
$(function(){});
$(document).ready(function(){})
```

- js

```
// ie9+
document.addEventListener('DOMContentLoaded',function(){});
```

#### dom listen

- jquery

```
$(e).on()
```

- js

```
// ie8+
element.attachEvent('on'+evnt, function() {});
// ie9+
element.addEventListener('evnt', function() {}, false);
```

#### by id

- jquery

```
$('#myElement');
```

- js

```
document.getElementById('myElement');
document.querySelector('#myElement');
```

#### by class

- jquery

```
$('.myElement');
```

- js

```
document.getElementsByClassName('myElement');
document.querySelectorAll('.myElement');
```

#### by tag name

- jquery

```
$('div');
```

- js

```
document.getElementsByTagName('div');
document.querySelectorAll('div');
```

#### get/set attributes

- jquery

```
$(e).attr('data-test');
$(e).attr('data-test','true');
```

- js

```
element.getAttribute('data-test');
element.setAttribute('data-test','true');
```

#### dom prepend/append

- jquery

```
$(e).prepend();
$(e).append();
```

- js

```
// prepend
element.insertAdjacentHTML('afterbegin','something u want to insert');
// append
element.insertAdjacentHTML('beforeend','something u want to insert');
```

#### add / remove css class

- jquery

```
$(e).addClass('xxx');
$(e).removeClass('xxx');
```

- js

```
// ie9+
element.classList.add('xxx');
element.classList.remove('xxx');

element.className += 'xxx';
element.className = element.className.replace(/^bold$/, '');
```

#### change css style

- jquery

```
$(e).css('display','none');
```

- js

```
element.style.dispaly('none');
```

#### get dom parent

- jquery

```
$(e).parent();
```

- js

```
element.parentNode
```

#### hasClass

- jquery

```
$(e).hasClass('xxxx');
```

- js

```
element.classList.contains('xxx');
element.className.match(/xxx/);
```

#### 事件委托

- jquery

```
$(parentEle).on(click, childEle, function() {})
```

- js

```
document.getElementById('parent').addEventListener('event', function(ev){
  var ev = ev || window.event;
  var target = ev.target || ev.srcElement;
  if(target.nodeName.toLowerCase! == 'li') {
    console.log(target)
  }
})
```
