---
title: "Mysql抖动"
date: 2020-03-29T14:28:45+08:00
tags: ["mysql"]
---

# 脏页

当内存数据页和磁盘数据也不一致的时候我们称这个内存页为`脏页`。内存页中的数据写入磁盘后，内存中的数据和磁盘中的数据保持一致称为`干净页`。

# flush

内存页中的数据和磁盘上的数据同步的过程叫flush，这时数据库发生抖动。

数据库flush的四种情况：

1. InnoDB 的redo log写满了。
2. 系统的内存页不够用。
3. MySQL认为系统空闲的时候会将脏页flush。
4. MySQL正常关闭的时候会将脏页flush。

# 脏页的比例计算

不要将脏页的比例经常接近`75%`,计算公式Innodb_buffer_pool_pages_dirty/Innodb_buffer_pool_pages_total

##  Innodb_buffer_pool_pages_dirty

```sql
 select VARIABLE_VALUE from performance_schema.global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
```

## Innodb_buffer_pool_pages_total

```slq
select VARIABLE_VALUE  from performance_schema.global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
```

