# MYSQL语句 

## 链接

- 内链接

相同的

```sql
SELECT a.user_name,a.over,b.over FROM user1 a INNER JOIN user2 b ON a.user_name=b.user_name;
```

-左外连接

左全部

```sql
SELECT a.user_name,a.over,b.over FROM user1 a LEFT JOIN user2 b ON a.user_name = b.user_name;
```

-右外连接

右全部

```sql
SELECT b.user_name,b.over,a.over FROM user1 a RIGHT JOIN user2 b ON a.user_name = b.user_name WHERE a.user_name IS NOT NULL;
```

-全连接

全部连接

```sql
SELECT a.user_name,a.over,b.over FROM user1 a LEFT JOIN user2 b ON a.user_name = b.user_name UNION ALL SELECT b.user_name,b.over,a.over FROM user1 a RIGHT JOIN user2 b ON a.user_name = b.user_name;
```

-交叉连接

笛卡尔基，CROSS JOIN 不需要连接条件，实际中用处不大

```sql
SELECT a.user_name,a.over,b.over FROM user1 a CROSS JOIN user2 b ;
```

- 使用join更新表

场景：更新同时存在于两张表中的某个字段,mysql不能更新在from从句中出现的表

```sql
UPDATE user1 a JOIN (SELECT b.user_name FROM user1 a JOIN user2 b ON a.user_name = b.user_name) b ON a.user_name = b. user_name set a.over = '齐天大圣' ;
```

- 使用join来优化子查询

```sql
SELECT a.user_name,a.over,(SELECT over FROM user2 b WHERE a.user_name = b.user_name) AS over2 FROM user1 a ; 
效率太低
修改后
SELECT a.user_name,a.over,b.over AS over2 FROM user1 a LEFT JOIN user2 b ON a.user_name = b.user_name;
```

- 使用join来优化聚合子查询

```sql
查询四人组打怪最多的时间
SELECT a.user_name,b.timestr,b.kills FROM user1 a JOIN user_kills b ON a.id = b.user_id WHERE b.kills = (SELECT MAX(c.kills) FROM user_kill c WHERE c.user_id = b.user_id);
优化下
SELECT a.user_name,b.timestr,b.kills FROM user1 a JOIN user_kills b ON a.id = b.user_id JOIN user_kills c ON c.user_id = b.user_id GROUP BY a.user_name,b.timestr,b.kills HAVING b.kills = MAX(c.kills);
```

- 如何实现分组选择

什么是分组选择，有一些记录有多个分类，我们需要在某个分类中的某些个数据，就是分组选择。
例如最受欢迎的分类课程，一般情况，先确定分类，再去查询最受欢迎，但是太麻烦

```sql
SELECT a.user_name,b.timestr,b.kills FROM user1 a join user_kills FROM user1 a JOIN user_kills b ON a.id = b.user_id WHERE user_name = '孙悟空' ORDER BY b.kills DESC LIMIT 2；
取经四人组杀人排行前两条
孙悟空，猪八戒，沙僧，要执行三次，
优化后
SELECT d.user_name,c.timestr,kills FROM (SELECT user_id,timestr,kills,(SELECT count(*) FROM user_kills b WHERE b.user_id = a.user_id and a.kills < b.kills) AS cnt FROM user_kills a GROUP BY user_id,timestr,kills) c JOIN user1 d ON c.user_id = d.id WHERE cnt <= 2;
```