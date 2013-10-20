Title: Codeigniter: Separating reads and writes for scaling MySQL
Slug: codeigniter-separating-reads-and-writes-for-scaling-mysql
Date: 2008-10-23 19:19
Category: Codeigniter
Tags: codeigniter, replication
Author: Sukumar Yethadka
Summary: Separate database reads and writes on codeigniter.

Generally websites average a ratio of 9:1 or more for reads:writes for their
applications which makes MySQL replication as one of the ways to scale you web
application. The simplest configuration is to separate reads and writes with
all the reads coming from the slave servers.

**MySQL Database replication**

We can implement this in <a href="http://codeigniter.com/"
target="_blank">codeigniter</a> using database groups.  Below is a sample
execution.

Create two active groups - one for read and one for write.

File: system/application/config/database.php

    :::php
    $active_group = 'write';
    $active_record = TRUE;

    $db['read']['hostname'] = 'localhost';
    $db['read']['username'] = 'root';
    $db['read']['password'] = '';
    $db['read']['database'] = 'read_database_name';
    $db['read']['dbdriver'] = 'mysql';
    $db['read']['dbprefix'] = '';
    $db['read']['pconnect'] = TRUE;
    $db['read']['db_debug'] = FALSE;
    $db['read']['cache_on'] = FALSE;
    $db['read']['cachedir'] = '';
    $db['read']['char_set'] = 'utf8';
    $db['read']['dbcollat'] = 'utf8_general_ci';

    $db['write']['hostname'] = 'localhost';
    $db['write']['username'] = 'root';
    $db['write']['password'] = '';
    $db['write']['database'] = 'write_database_name';
    $db['write']['dbdriver'] = 'mysql';
    $db['write']['dbprefix'] = '';
    $db['write']['pconnect'] = TRUE;
    $db['write']['db_debug'] = FALSE;
    $db['write']['cache_on'] = FALSE;
    $db['write']['cachedir'] = '';
    $db['write']['char_set'] = 'utf8';
    $db['write']['dbcollat'] = 'utf8_general_ci';

Create separate database connections to access read and write databases
separately where needed (constructor is generally a good place for this).

    :::php
    $read_db = $this->load->database('read', TRUE);
    $write_db = $this->load->database('write', TRUE);

You can now go about running queries in the usual way.

    :::php
    /* For reads */
    $query = $read_db->get('table_name');

    foreach ($query->result() as $row)
    {
        echo $row->title;
    }

    /* For writes */
    $data = array(
            'title' => $title,
            'name' => $name,
            'date' => $date
            );

    $write_db->insert('mytable', $data);

In case you want to try this on an application you are already running, you
can leave the &#8220;default&#8221; connection group intact and create only
the &#8220;read&#8221; connection group. Use it where you think you are
running read heavy queries.

## Old Comments

**Stinky Tofu**

This is an interesting approach.  Just wondering though, if you have a master
db that is for writes only and a slave db for reads only, then instead of
asking the programmer to specify which database to read/write from/to in their
code, why not set it up so that whenever I call $db->insert,
$db->update, or $db->delete, then automatically use the master db to
perform the write, if I am calling $db->query, then the code should be
smart enough to connect to the slave db for the reads.  Would this be a safer
approach?  It would help prevent human error.  Curious to find out what you
think about this approach.

---

**Sukumar Yethadka**

@Stinky Tofu
Yes, that would be quite a good approach and can be implemented
just by extending the database model.  My preference is slightly different
though. I like to use fat controllers and thin models (most of the logic is in
the controller). Because of this, I have a base model where I have all the
commonly used methods defined (different way of CRUD) and all other models
just extend from it. This way I don&#8217;t need to decide what query goes
where everyday but, just at the time of writing the model.

---

**Naren**

Very interesting take. Why don&#8217;t you create a database helper/extend the
database class to abstract away the need to keep track of which DB
you&#8217;re supposed to write to, and read from.

---

**Brad Proctor**

You really need to be careful with this.  What happens if you make an insert
and the user is redirected to see the results and a select is occurs of what
was just inserted, but the the master hasn&#8217;t replicated yet.  The user
sees nothing.  To fix this, that select needs to occur on the master.
