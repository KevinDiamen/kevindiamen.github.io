# Web 安全 - XSS

- [介绍](#introduction)
- [SQL注入防御](#person-ssl)

<a name="introduction"></a>
## 介绍
跨站脚本攻击(Cross Site Scripting)。 恶意攻击者往Web页面里插入恶意Script代码，嵌入其中Web里面的Script代码会被执行，从而达到恶意攻击用户的特殊目的

## XSS注入防御

### 不要相信用户的输入


```
$db = new PDO(...);
$query = $db->prepare('UPDATE users SET first_name = :first_name WHERE id = :id');

$query->execute([
  ':first_name' => $_POST['first_name']
]);
```

>  - 输出处理 ： 虽然绑定参数可以保护你的query 但是数据可能是恶意的 所以你显示给用户的时候还需要做处理
>  - 过滤参数 ： 比如手机号 邮箱可以用正则表达式匹配, PHP 建议使用 filter_ var() 和 filter_input() 函数可以过滤文本并对格式进行校验

 
### 输出也需要进行过滤
存储过程是另一个方法去避免SQL注入, 他是一组完成特定功能的SQL,经过编译后存储在数据库中, 但是存储过程比较麻烦主要有几点原因:
- 调试麻烦
- 移植问题
- 逻辑在另一个应用上
