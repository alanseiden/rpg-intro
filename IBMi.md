# IBM i Introduction

IBM i is an operating system created by IBM. It has very specific functionality which make it unique: including development environment & database integration.

1. File system (QSYS, library list)
2. Jobs
3. Database integration
4. Development environment (ILE / TIMI)
5. Commands

To access an IBM i menu, you will need Access Client Solutions. You can download ACS from [the IBM website here](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_72/rzajr/rzajracsintro.htm).

## File System

The primary file system on IBM is called the 'library file system' (also known as the QSYS file system). The file system is made of objects. An object can be a program `*PGM`, database table `*FILE`, library user profile `*USRPRF`, anything. A library (which is also an object `*LIB`) is what holds objects in, but a library cannot hold another library within. Both objects and libraries names can only be a **max of 10 characters**.

A simple way to navigate and search for objects on IBM i is by using the `WRKOBJ` command (Work with Objects). Simply put, it takes two parameters:

1. Object path (`LIB/OBJ`)
2. Object type (`*PGM`, `*FILE`, `*ALL`, etc)

For example:

* to find all objects in a certain library: `WRKOBJ YOURLIB/*ALL`
* to find all programs `*PGM` in a certain library: `WRKOBJ YOURLIB/*ALL *PGM`
* to find an specific object on the entire system: `WRKOBJ *ALL/YOUROBJ`
* to find all objects that begin with `abc`: `WRKOBJ *ALL/ABC*`
* to find an object on your **library list**: `WRKOBJ *LIBL/MYFILE`

### Library list

The library list plays a vital part when working or developing applications running on IBM i. Each system job has it's own library list. The library list is a similar concept to a path list on other systems. On Windows and Unix, to execute a program without a direct path it would need to be on the path list. That is the same idea on IBM i, but with libraries for objects.

Let's say this is our library list:

```
Library   
QSYS      
QSYS2     
QHLPSYS   
QUSRSYS
QGPL      
DEMOLIB   
```

If we wanted to call a program called `MYPGM` which existed in the `YOURLIB` library:

* `CALL YOURLIB/MYPGM` would call successfully because we have given a library on the `CALL` command.
* `CALL MYPGM` would fail because it would search through the libraries in the library list to find the object, but the `YOURLIB` library is not on it.

The library list may be referenced as  `*LIBL` in system commands. You can read more about the library list on [this Wikipedia page](https://en.wikipedia.org/wiki/AS/400_Library_List).

## Jobs

Unlike Windows and Unix, IBM i has a concept of jobs instead of processes. All work done on a system is performed through jobs. Each job has a unique name within the system. All jobs, with the exception of system jobs, run within subsystems.

Each active job contains at least one thread (the initial thread) and may contain additional secondary threads. Threads are independent units of work. Job attributes are shared among the threads of the job, however threads also have some of their own attributes, such as a call stack.

When you start an interactive session on IBM i, that will have it's own job (known as an interactive job, which usually sits in the `QINTER` subsystem). When you call programs from the command line, they will be run under your job. Each job also gets it's own library list, which can be controlled with a job description object (`*JOBD`)

A job has lots of characteristics and you can see these by using `WRKJOB` on the command line, which will allow you to work with your interactive job. Optionally, if you want to work with another job, you can pass in the job information as the second parameter.

```
                                 Work with Job                                  
                                                             System:   POWER8   
 Job:   QPADEV0004     User:   LALLAN         Number:   700310                  
                                                                                
 Select one of the following:                                                   
                                                                                
      1. Display job status attributes                                          
      2. Display job definition attributes                                      
      3. Display job run attributes, if active                                  
      4. Work with spooled files                                                
                                                                                
     10. Display job log, if active, on job queue, or pending                   
     11. Display call stack, if active                                          
     12. Work with locks, if active                                             
     13. Display library list, if active                                        
     14. Display open files, if active                                          
     15. Display file overrides, if active                                      
     16. Display commitment control status, if active                           
                                                                        More... 
 Selection or command                                                           
 ===>                                                                           
                                                                                
 F3=Exit   F4=Prompt   F9=Retrieve   F12=Cancel                                 
 Function key not allowed.                                                      
 ```

`More...` indicates that you can page down for more options.

 From this screen, you can find out job information like:

 * library list
 * locked files
 * current call stack
 * job log

You can also end a job from this screen by taking option `41. End job` which is useful if you have a job that is stuck in a loop or stuck waiting for something.

### Job log

When developing ILE applications, it's useful to have the job log by your side. The job log will report error messages from almost every system command and compiler. You can access a job's job log by using `WRKJOB` and taking option `10`. If you just want to access the job log of your current interactive session, you can use `DSPJOBLOG` (and then `F10` followed by `F18` to get to the bottom)

### Documentation links:

* [Jobs](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_73/rzaks/rzaksaboutjobs.htm)
* [A job's life](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_73/rzaks/rzaksjoblife.htm)
* [Job characteristics](https://www.ibm.com/support/knowledgecenter/ssw_ibm_i_73/rzaks/rzaksjobcharacter.htm)

## Integrated Database

IBM i runs a version of Db2, named Db2 for i, which is integrated into the operating system. This means that it's simple to work with SQL right away. Any file object (`*FILE`) can be accessed from Db2 for i.

Because of how long IBM i has existing, terminology has changed over time.


| Then     | Now    |
|----------|--------|
| Physical | Table  |
| Library  | Schema |
| Field    | Column |
| Record   | Row    |
| Logical  | View or Index |

 A logical file contains no data, but it defines views for one or more physical files. Using DDS (data description specifications) to create either tables/files or logicals is outdated and has been replaced with SQL. This is similar to an SQL view or index.

* An SQL view is a logical file that never has a key, but is much more powerful than a DDS described logical file, because it can contain everything that is possible with a SELECT statement, except an order by. An SQL view can be specified in an SQL statement, but also with native I/O.
* An SQL index is a keyed logical file. The query optimizer needs access paths (either in SQL indexes or keyed logical file) to retrieve the data as fast as possible.

You may also hear SQL referenced as DDL (data definition language). You can [read more here](http://ibmsystemsmag.com/ibmi/developer/modernization/a-debate--dds-vs--ddl/) about using SQL/DDL over DDS.

Here is an example of creating a table called `PRODUCTSP` in both DDS and SQL/DDL:

#### DDS

Note that DDS is also a fixed spacing definition language, meaning there is a limit on column names.

```
     A*
     A                                      UNIQUE
     A          R PRODUCTS
     A*
     A            PRID           5  0       TEXT('Product Id')
     A            PRNAME        30          TEXT('Product Name')
     A            PRDESC        60          TEXT('Description')
     A            PRPRICE        7  2       TEXT('Price')
     A            PRIMAGE       60          TEXT('Image URL')
     A            PRQTY          5  0       TEXT('Quantity Available')
     A            PRCATID        5  0       TEXT('Category Id')
     A*
     A          K PRID
     A*
```

#### SQL/DDL

Now, that same definition but in SQL - a lot easier to understand.

```sql
CREATE TABLE PRODUCTSP (
	PRID DECIMAL(5, 0) NOT NULL DEFAULT 0,
	PRNAME CHAR(30) NOT NULL DEFAULT '',
	PRDESC CHAR(60) NOT NULL DEFAULT '',
	PRPRICE DECIMAL(7, 2) NOT NULL DEFAULT 0,
	PRIMAGE CHAR(60) CCSID 37 NOT NULL DEFAULT '',
	PRQTY DECIMAL(5, 0) NOT NULL DEFAULT 0,
	PRCATID DECIMAL(5, 0) NOT NULL DEFAULT 0,
	PRIMARY KEY( PRID )
)
```

### Running SQL statements

The best preffered and free method to run SQL on your system is by using ACS (Access Client Solutions). There is a tool named Run SQL Scripts built in which looks a little bit like this:

![](https://www.itjungle.com/fhg/fhg110116-story01-fig03.png)