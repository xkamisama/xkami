- 子查询 练习重点
- 数据库对象 理解个别使用
- plsql块 了解

### 子查询
select * from emp where sal>(select sal from emp where empno=7654);

any
select * from emp where sal>any(select avg(sal) from emp group by deptno);

in
select * from emp where sal in (select sal from emp where comm>100);

exists
select * from emp where exists (select sal from emp where comm>100);
select * from dept where exists (select * from emp where emp.deptno = dept.deptno);
### 视图
conn system/a123;
grant create view to xkami;
conn xkami/a123;
create view v_minsl_emp
as
select e.empno empno,e.ename ename,d.dname dname from
   emp e,dept d
   where e.deptno=d.deptno
  and e.sal = (select min(sal) from emp);

### 序列

create sequence seq_dept
increment by -3
start with 9999
maxvalue 9999
minvalue 0
nocache

### 索引
create index index_emp_job
on emp(job);
create index index_emp_job
on emp(job,ename);
