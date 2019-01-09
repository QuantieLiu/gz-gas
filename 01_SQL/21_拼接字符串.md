
#### 

<li>原数据
  
```
a  1
a  2
b  5
c  6
b  2
```

<li>预想效果图

```
a  1，2
b  5，2
c  6
```

聚合函数，在group语句中使用，可以将多行的字符串按分组整合成一个字符串

```
语法：
GROUP_CONCAT([DISTINCT] expr [,expr ...][ORDER BY {unsigned_integer | col_name | expr}[ASC | DESC] [,col_name ...]][SEPARATOR str_val])
假定：有一张表是UserRole存储User对应Role关系，其有两列userId,RoleCode，我们可以通过GROUP_CONCAT来取出用逗号分隔的用户角色

select userId,GROUP_CONCAT(RoleCode SEPARATOR  ',') from UserRole
需要注意，GROUP_CONCAT函数默认的最大可连接字符串的长度是1024，如果连接的字符串长度超过1024的话会被截断，不过我们可以通过设置group_concat_max_len的值来修改GROUP_CONCAT的最大长度。

例如：
SET SESSION group_concat_max_len= 99999;
select userId,GROUP_CONCAT(RoleCode SEPARATOR  ',') from UserRole
```

<li>实际应用
  

```
--单表
select tt.id,GROUP_CONCAT(tt.ROLE_NAME SEPARATOR  ',') as role_name from sys_uer_role tt GROUP BY tt.id

--基于三表联查
select tt.id,GROUP_CONCAT(tt.ROLE_NAME SEPARATOR  ',') as role_name from (select u.id,r.ROLE_NAME 
from sys_user u 
JOIN sys_user_role ur on u.id=ur.user_id 
JOIN sys_role r on ur.ROLE_ID=r.ID)tt
GROUP BY tt.id
```
