Title: How to skip MySQL replication counter
Slug: how-to-skip-mysql-replication-counter
Date: 2008-10-07 19:19
Category: MySQL
Tags: mysql, replication
Author: Sukumar Yethadka
Summary: Skip MySQL replication counter with these easy steps.

There are such times when MySQL replication stops when you run certain updates
on the master and the slave fails. Like for example you may have run a create
table or an alter table on the master and further inserts but these DDLs get
skipped and the inserts into these non-existent tables cause the slave to
error out and stop.

A simple way out of this is to run the missing DDLs on the slave and push the
counter by a step. This is not recommended as this might cause data
inconsistency.

Use the below sql to skip the counter by 1.

    :::sql
    SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
    START SLAVE;

Just remember to use the value 1 for any SQL statement that does not use
AUTO_INCREMENT or LAST_INSERT_ID(), otherwise you will need to use the value
2. Statements that use AUTO_INCREMENT or LAST_INSERT_ID() take up 2 events in
the binary log.

To skip lots of duplicate errors, you can set this in my.cnf

    :::text
    slave-skip-errors = 1062
