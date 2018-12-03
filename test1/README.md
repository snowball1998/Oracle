# 实验一：分析SQL执行计划，执行SQL语句的优化指导

## 实验内容：
	对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。首先运行和分析教材中的样例：本训练任务目的是查询两个部
	门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
	设计自己的查询语句，并作相应的分析，查询语句不能太简单。

### 查询1：
```SQL
	set autotrace on
   	SELECT d.department_name，count(e.job_id)as "部门总人数"，
	avg(e.salary)as "平均工资"
	from hr.departments d，hr.employees e
	where d.department_id = e.department_id
	and d.department_name in ('IT'，'Sales')
	GROUP BY department_name;
```
![image](https://github.com/snowball1998/Oracle/blob/master/test1/1.png)
### 查询2：
```SQL
	 set autotrace on
	SELECT d.department_name，count(e.job_id)as "部门总人数"，
	avg(e.salary)as "平均工资"
	FROM hr.departments d，hr.employees e
	WHERE d.department_id = e.department_id
	GROUP BY department_name
	HAVING d.department_name in ('IT'，'Sales');
```
![image](https://github.com/snowball1998/Oracle/blob/master/test1/2.png)
### 查询1执行计划：
	查询1的SQL语句是直接从hr.departments d和hr.employees e两个表中查询出"部门总人数"和"平均工资",
	然后通过where子句进行约束限制,最后通department_name过department_name来排列显示。
### 查询2执行计划：
	查询2的SQL语句是直接从hr.departments d和hr.employees e两个表中查询出"部门总人数"和"平均工资",
	然后通过where子句进行约束限制然后又通过department_name来排列显示,最后通过having in 来过滤显示内容。
### 分析查询1和查询2的SQL语句谁较优： 
	（1）查询1的Cost(%CPU)普遍比查询2的Cost（%CPU）消耗低
	（2）通过查询时间可以看出查询1的SQL语句比查询2的SQL的SQL语句查询时间低
	（3）因为Oracle存在缓存的机制，多运行几次后依旧是查询1语句较优    	
				
### 最优查询：	

![image](https://github.com/snowball1998/Oracle/blob/master/test1/3.png)
				     	

