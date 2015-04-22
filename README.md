bdr_demo_ansible
================

This repository contains a set of Ansible playbooks to easily
create a simple BDR test set-up with two or more nodes running
on localhost.


Requirements
------------

- Ansible ( http://www.ansible.com/ )
- psycopg2
- BDR-enabled PostgreSQL ( http://wiki.postgresql.org/wiki/BDR_Quick_Start#Installing_the_patched_PostgreSQL_binaries )
  Note: `bdr_demo_ansible` will not build a BDR-enabled PostgreSQL;
  see the above link for details on how to do this.


Configuration
-------------

Create a file `your_hostname.yml` in the `host_vars` directory
with the following variables (default values in parentheses):

- `db_superuser` (`postgres`)
- `db_name` (`bdrdemo`)
- `bdr_bin`: path to the BDR PostgreSQL `bin/` directory
- `base_data_dir`: arbitrary directory for each BDR instance's data files
- `bdr_log_dir` (`tmp`): directory for BDR PostgreSQL log files
- `bdr_ports`: list of ports for BDR PostgreSQL instances
- `bdr_replica_src`: port number of the initial BDR PostgreSQL instance,
  from which the other instances will replicate from (if in doubt, just
  use the first entry in `bdr_ports`).

Add `your_hostname` to the `[bdr_hosts]` section of `hosts.ini`

The file `host_vars/sample_host.yml` provides a useful template containing the
parameters which will probably need to be configured.

MacPorts users: if Ansible complains about `psycopg2` being missing, it is
probably using the OS X native Python interpreter; set `ansible_python_interpreter`
to point to the MacPorts version.

Operation
---------

    $ ansible-playbook -i hosts.ini bdr_init.yml

    PLAY [all] ********************************************************************

    TASK: [bdr_init | Create data directories] ************************************
    changed: [nara] => (item=5597)
    changed: [nara] => (item=5598)
    changed: [nara] => (item=5599)

    (...)

    TASK: [bdr_init | Create BDR group] *******************************************
    changed: [osaka]

    TASK: [bdr_init | Add other nodes to group] ***********************************
    skipping: [osaka] => (item=5597)
    changed: [osaka] => (item=5598)
    changed: [osaka] => (item=5599)

    PLAY RECAP ********************************************************************
    osaka                      : ok=15   changed=13   unreachable=0    failed=0


    $ /path/to/pg_bdr/bin/psql -p 5597 bdrdemo
    psql (9.4beta2)
    Type "help" for help.

    bdrdemo=# CREATE TABLE foo(id INT);
    DEBUG:  attempting to acquire global DDL lock for (bdr (6056475237161047104,1,16384,))
    DEBUG:  sent DDL lock request, waiting for confirmation
    DEBUG:  global DDL lock acquired successfully by (bdr (6056475237161047104,1,16384,))
    CREATE TABLE
    Time: 16.755 ms
    bdrdemo=# COMMENT ON TABLE foo IS 'Created on 5597';
    COMMENT
    Time: 28.503 ms
    bdrdemo=# \q

    $ /path/to/pg_bdr/bin/psql -p 5598 bdrdemo
    psql (9.4beta2)
    Type "help" for help.

    bdrdemo=# \dt+
                          List of relations
     Schema | Name | Type  |  Owner  |  Size   |   Description
    --------+------+-------+---------+---------+-----------------
     public | foo  | table | barwick | 0 bytes | Created on 5597
    (1 row)



Playbooks
---------

Following playbooks have been defined:

- `bdr_init.yml`

  Creates and configures BDR-enabled instances

- `bdr_destroy.yml`

  Halts existing BDR-enabled instances and removes the database
  directories

- `bdr_start.yml`

  Starts previously configured but not running BDR-enabled instances

- `bdr_stop.yml`

  Stops running BDR-enabled instances