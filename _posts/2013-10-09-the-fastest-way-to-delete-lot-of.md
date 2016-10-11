---
layout: post
title: The fastest way to delete a lot of records in a TSQL table.
date: '2013-10-09T21:44:00.004-06:00'
author: Sanjiv Kawa
tags:
- Drop
- Row
- Table
- Records
- T-SQL
- Select
- Alter
- Schema
- Database
- Delete
- TSQL
- Fast
- Column
comments: true
---
This isn't Security related research, but it could come in handy for anyone looking to delete a large amount of data from TSQL databases.

I recently had a task where I had to delete well over 50,000 rows based on a common column value within a TSQL database. Originally I thought that a simple conditional DELETE statement would do the trick, for example:

{% highlight css %}
delete from DBNAME.DBO.TABLE1 where DATE between 2005 and 2012
{% endhighlight %}

The issue here is that the database could be so bogged down that the DELETE query could take greater than 20 seconds to remove a single record. This means it would take 11.5 days to delete 50,000 records on a single looping query. No exaggeration.

I decided that the least expensive way to do this (in terms of time) would be to do the following:

Always back up the database, do it twice.

Create a second table (TABLE2) that has the exact same schema as the original table (TABLE1). This can be done pretty easily using the CREATE TO script built into MS SQL Management Studio.

Move all the data we want, lets say records newer than 2012, from TABLE1 into TABLE2 using a pretty standard SELECT statement. You might run into some errors here if the Identity property is set on the table. So first, lets figure out if identity is set on any tables in the database:

{% highlight css %}
select column_name, table_name from INFORMATION_SCHEMA.COLUMNS
where TABLE_SCHEMA = 'dbo'
and COLUMNPROPERTY (object_id(table_name), column_name, 'isIdentity') = 1
order by table_name
{% endhighlight %}

If Identity isn't set, we can move all the rows that are newer than 2012 from TABLE1 to TABLE2:

{% highlight css %}
insert DBNAME.DBO.TABLE2
select * from DBNAME.DBO.TABLE1
where DATE between '2012-01-01' and '2099-01-01'
{% endhighlight %}

Find all the dependant Foreign Key mappings that TABLE1 has and make sure you record the name(s) of them. Thanks <a href="http://stackoverflow.com/questions/483193/how-can-i-list-all-foreign-keys-referencing-a-given-table-in-sql-server">stackoverflow</a>:

This displays all the other tables that have Foreign Keys that rely on TABLE1:

{% highlight css %}
select t.name as TableWithForeignKey, fk.constraint_column_id as FK_PartNo , c.name as ForeignKeyColumn
from sys.foreign_key_columns as fk
inner join sys.tables as t on fk.parent_object_id = t.object_id
inner join sys.columns as c on fk.parent_object_id = c.object_id and fk.parent_column_id = c.column_id
where fk.referenced_object_id = (select object_id from sys.tables where name = 'TABLE1')
order by TableWithForeignKey, FK_PartNo
{% endhighlight %}

This displays the Foreign Key names that are within the tables that have Foreign Keys which rely on TABLE1.

{% highlight css %}
select distinct name from sys.objects where object_id in
(   select fk.constraint_object_id from sys.foreign_key_columns as fk
    where fk.referenced_object_id =
     (select object_id from sys.tables where name = 'TABLE1')
)
{% endhighlight %}

Drop the Foreign Key associations on various dependant tables, then drop TABLE1.

{% highlight css %}
alter table DBNAME.DBO.TABLE3 drop fk_TABLE3_TABLE1

alter table DBNAME.DBO.TABLE4 drop fk_TABLE4_TABLE1

alter table DBNAME.DBO.TABLE5 drop fk_TABLE5_TABLE1

...

drop table DBNAME.DBO.TABLE1
{% endhighlight %}

Recreate TABLE1 using the same schema as TABLE2 using the same CREATE TO script in step 1.

Re-associate the Foreign Keys to the new TABLE1.

{% highlight css %}
alter table DBNAME.DBO.TABLE3 with check add constraint fk_TABLE3_TABLE1 foreign key([ID])
references DBNAME.DBO.TABLE1 ([ID])
on delete cascade
go

alter table DBNAME.DBO.TABLE3 check constraint fk_TABLE3_TABLE1
go

...
{% endhighlight %}

Copy all of the data from TABLE2 to TABLE1. Then drop TABLE2.

{% highlight css %}
insert DBNAME.DBO.TABLE2
select * from DBNAME.DBO.TABLE1
where DATE between '2012-01-01' and '2099-01-01'
go

drop table DBNAME.DBO.TABLE2
{% endhighlight %}

The query speeds were significantly faster, depending on your database size this can take less than an hour to do. It is much better than waiting 11.5 days and hypothetically causing downtime for a clients critical application.

Hope this helps.
