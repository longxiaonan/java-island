### sql实例
```sql
create table employee(
`id` int,
`name` varchar(20),
`sex` char(2),
`date` date,
`addr` varchar(50),
`work` varchar(10)
);
-- 向员工表中插入5条记录，其中第2条生日为空，第5条住址为空，第1条和第3条的性别为空
-- 第一条记录:
insert into db1.employee (id, name, date, addr, work)
values (001, '孙悟空', '2010-01-01', '花果山', '程序员');
-- 第二条记录:
insert into db1.employee (id, name,sex , addr, work)
values (002, '猪八戒', '男', '花果山', '农民');
-- 第三条记录:
insert into db1.employee (id, name, date, addr, work)
values (003, '白骨精', '2010-01-01', '白骨洞', '卖骨头');
-- 第四条记录:
insert into db1.employee (id, name, sex, date, addr, work)
values (004, '蜘蛛精', '女', '1990-3-01', '蜘蛛洞', '织网');
-- 第五条记录:
insert into db1.employee (id, name, sex, date, work)
values (005, '唐僧', '男', '2016-9-30', '念经');
-- 修改第三条的性别, 通过条件修改多项的值
update employee set sex='男',addr='佛山市' where id=3;
-- 将性别为空的员工，性别修改为女. 判断是否为空
update employee set sex='女' where sex is null;
-- 将生日为空的员工，修改生日为'1995-11-24'. date需要引号
update employee set date='1995-11-24' where date is null;
-- 删除ID为4的员工. 删除记录
delete from employee where id=4;
SELECT * from employee;
```

### 查询语句
#### 数据准备
```sql
create table student(
    id int,
    name varchar(20),
    chinese float,
    english float,
    math float
);
insert into student(id,name,chinese,english,math) values(1,'张小明',89,78,90);
insert into student(id,name,chinese,english,math) values(2,'李进',67,53,95);
insert into student(id,name,chinese,english,math) values(3,'王五',87,78,77);
insert into student(id,name,chinese,english,math) values(4,'李一',88,98,92);
insert into student(id,name,chinese,english,math) values(5,'李来财',82,84,67);
insert into student(id,name,chinese,english,math) values(6,'张进宝',55,85,45);
insert into student(id,name,chinese,english,math) values(7,'黄蓉',75,65,30);
```
#### 查询语句
```sql
-- 查询表中所有学生的信息。
select * from student;
-- 查询表中所有学生的姓名和对应的英语成绩。
select name,english from student;
-- 过滤表中英语成绩的重复数据
select DISTINCT(english) from student;
-- 使用别名表示学生分数。
select name 姓名,english 英语成绩 from student;
-- 查询姓名为李一的学生成绩
select * from student where name='李一';
-- 查询英语成绩大于等于80分的同学
select * from student where english > 80;
-- 查询总分大于250分的所有同学
select * from student where chinese+english+math > 250;
-- 查询所有姓李的学生英语成绩。
select name,english from student where name like '%李%';
-- 查询英语>90或者总分>220的同学
select * from student WHERE english>90 OR chinese+english+math > 220;
-- 对数学成绩排序后输出。
select * from student GROUP BY math desc;
-- 对总分排序后输出，然后再按从高到低的顺序输出
select *, chinese+english+math as 总分 from student ORDER BY chinese+english+math desc;
-- 对姓李的学生成绩排序输出
select * from student where name like '李%' ORDER BY chinese+english+math desc;
-- 统计一个班级共有多少学生？
select count(id) from student;
-- 统计数学成绩大于90的学生有多少个？
select count(id) from student WHERE math > 90;
-- 统计总分大于250的人数有多少？
select count(id) from student WHERE chinese+english+math > 250;
-- 统计一个班级数学总成绩？
select sum(math) from student;
-- 统计一个班级语文、英语、数学各科的总成绩
select sum(math),sum(chinese),sum(english) from student;
-- 统计一个班级语文、英语、数学的成绩总和
select sum(math+chinese+english) from student;
select sum(math)+sum(chinese)+sum(english) from student;
-- 统计一个班级语文成绩平均分
select AVG(chinese) from student;
-- 求一个班级数学平均分？
select AVG(math) from student;
-- 求一个班级总分平均分
select avg(math+chinese+english)from student;
select avg(math)+avg(chinese)+avg(english) from student;
-- (语文总分+英语总分+数学总分)/人数=每一科的平均分累加和
-- 求班级最高分和最低分（数值范围在统计中特别有用）
```

