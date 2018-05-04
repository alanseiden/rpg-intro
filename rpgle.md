# rpg-intro

Introduction to RPG (Report Program Generator). See [this Wikipedia](https://en.wikipedia.org/wiki/IBM_RPG) page for an introduction to RPG.

## Todo

* Intro to RLA
* Creating service programs out of modules
* Creating commands for programs

## Contents

1. IBM i file system and the IFS
2. IBM i commands
3. Creating your first RPG program.
4. RPG Syntax
5. RPG data-types
6. RPG Procedures
7. RPG subroutines
8. RPG data-structures
9. RPG prototypes
10. RPG modules and programs
11. Embedded SQL basics

## IBM i file system and the IFS 

The two file systems we’re going to look at are 

* The 'library file system'
* The IFS (Integrated-File-System) 

The reason we’re going to look at these is because you can develop RPG in either, but their differences must be explained. Like most other operating system, you can create a simple stream file and write your code inside of this file and the compiler will do its work – this is still a normal concept on IBM i, but most industries do not do this. 

You’ll find that most IBM i shops will develop in something called ‘source members’. A ‘source member’ is a DB2 concept. A table within DB2 can contain multiple members which share the same columns, but there are special tables called ‘source physical files’ (SPF for short). ‘Physical file’ (PF for short) is a legacy name for tables, but the name will continue to live on while people develop in source members. A physical file is a type of object and all objects live within a library.  

Our future programs, modules, commands, and tables will all live within a library and each object name is a maximum of 10 characters long. Libraries cannot contain other libraries (except for QSYS, which is where every library lives). A library is just another type of object. 
Each source physical file has a record length. When you create a source physical file (CRTSRCPF) you can specify a record length. A normal length for an SPF is 112. Whatever length you supply to the command, minus 12 – this result is the max length of each line when editing source code. 

The reason we minus 12 is because the first 12 bytes are used for the line sequence number and source date (when that line was last updated). Now note, this is only for source physical files – you don’t have to worry about line length with stream files.

## IBM i commands 
Commands on IBM i are the simplest form of commands ever invented. Commands are a maximum of 10 characters long (because they’re an object) and are made up of word abbreviations. For example:

![](https://raw.githubusercontent.com/WorksOfBarry/rpg-intro/master/assets/table1.PNG)

Concatenating the abbreviations can make up commands, which are also pronounceable:

![](https://raw.githubusercontent.com/WorksOfBarry/rpg-intro/master/assets/table2.PNG)

Some of the commands we’ll be using throughout the lectures are the following:

![](https://raw.githubusercontent.com/WorksOfBarry/rpg-intro/master/assets/table3.PNG)

When using commands on the IBM i, if you are unsure of the parameters you are able to prompt the command. You can prompt after you have entered the command and then pressed F4. This will bring up a list of parameters available for a command. You may also have the option to use F10, which will show more available parameters. 

If you’re searching for a command, you can type the start of the command followed by an asterisk. For example:

* `CRT*` will show you all commands starting with `CRT`
* `WRK*` will show you all commands starting with `WRK`

## Creating your first RPG program.

First, we will need to do the following steps:

1. Create a library (`CRTLIB`)
2. Create a source physical file (`CRTSRCPF`)
3. Create a source member (`ADDPFM`)

You may also do this within Rational Developer for i, but it’s very useful to memorise these commands for when you don’t have the IDE.

It is possible to write all your RPG code in stream files on the IFS. The easiest way to create a stream file is with Rational Developer for i.

```
**FREE  

//Declare lText standalone field/variable 
Dcl-S lText Char(25);  

//Assign a value to field 
lText = 'Hello world.';  

//Display the lText value. 
Dsply lText;  

//Set Indicator LR to *On (true) and return 
*InLR = *On; 
Return;  
```
**Note**: You must be running OS version 7.2 at TR 11 or 7.3 at TR 3. You can look up your system's TR level using the approach described in [this post](http://www.seidengroup.com/2014/08/16/find-technology-refresh-tr-level-ibm-i).
 
Once we have written our code, we are going to use CRTBNDRPG to create our program object. What makes CRTBNDPGM useful is that it just creates a program object – it’s automating the CRTRPGMOD step. Another way we could have created our program object is by using CRTRPGMOD and then CRTPGM over that module. 

CRTBNDRPG does allow you to compile both stream files and source members.

* To compile a source member: `CRTBNDRPG PGM(MYLIB/XMPLE1) SRCFILE(MYLIB/QRPGLESRC) SRCMBR(XMPLE1) TEXT('My RPG IV Program')`
* To compile a stream file: `CRTBNDRPG PGM(MYLIB/XMPLE1) SRCSTMF('xmple1.rpgle') TEXT('My RPG IV Program')`

Note that when you compile a stream file, the SRCSTMF path should be relative to your current directory when running the compile.

You can [read more here](http://www.ibm.com/support/knowledgecenter/ssw_ibm_i_72/cl/crtbndrpg.htm) for info on CRTBNDPGM.

You are now going to call your program using `CALL <programname>`, which should then display ‘Hello world’ on your terminal. If you only see the output flash on your screen then run `WRKMBRPDM` before calling your program.

## RPG Syntax

There are lots of variations of RPG syntax. For all the RPG we write, we will be using free format. 

Notice how in the last lecture we used `**FREE` – this is a compiler directive. It tells the compiler we’re going to write our free format code from the start of the line. If we didn’t have that directive, we’d have to start each line at the 8th index (8 spaces, then start the code). If used, `**FREE` must be on the first line and in the first 6 characters with nothing following.

`*InLR = *On` tells the runtime that we’re on the last record. We need this because some elements of legacy RPG are still supported. [Read more about indicator LR here.](https://www.mcpressonline.com/programming/rpg/practical-rpg-activation-groups-and-inlr)

> Setting `*InLR = *On` by itself never ends the program - it simply tells the program to terminate _when_ you return. - JonBoy, code400.com

In RPG, things like IF, DOW, DOU, SELECT, WHEN, DCL-S, etc are all RPG operations. You will always need a space between the operation code and the next value. For good practice, you should surround expressions with brackets.

```
//Good practice
If (2 > 1);
  Dsply 'True';
ENDIF;

//Bad practice
If 2 > 1;
  Dsply 'True';
ENDIF; 
```

Variables are case-insensitive and cannot start with numeric digits (but can contain them). Variables must be defined at the top of your program/procedure It is good practice to start variables in the global scope with a lowercase 'g' – variables in a local scope to start with a lowercase 'l'.

```
**FREE
Ctl-Opt DFTACTGRP(*No);

Dcl-S gMyVar Char(10);

MyProc();

*InLR = *On;
Return;

Dcl-Proc MyProc;
  Dcl-S lMyVar Char(10);

  gMyVar = 'Hello';
  lMyVar = 'World';
END-PROC;    
```

## RPG data-types

RPG consists of lots of data-types, in this lecture we’re going to cover a few of them. You declare variables with the ‘Dcl-S’ operation, followed by the name, type and then optional keywords.

```
Dcl-S name type [keywords]
```

Here are some of the types we’re going to use in our lectures:

![](https://raw.githubusercontent.com/WorksOfBarry/rpg-intro/master/assets/table4.PNG)

When you use character fields in expressions, it will always trim the right blanks from the variable. For example:

```
Dcl-S lMyVarA Char(20)  Inz('Hello!');
Dcl-S lMyVarB Char(100) Inz('Hello!'); 

If (lMyVarA = lMyVarB); 
  Dsply 'True';
ENDIF;
```

Is actually computed as:

```
If (%TrimR(lMyVarA) = %TrimR(lMyVarB));
  Dsply 'True';
ENDIF;  
```

RPG provides lots of keywords when declaring both variables and data-structures (more on data-structures in a future Lecture). One of those is `DIM` (dimensions), which can be used on both `Dcl-S` and `Dcl-Ds`. The `DIM` keyword allows you declare arrays, where the length is specified in the ‘DIM’ keyword. You can reference array elements the same way you would call a procedure.

```
Ctl-Opt DftActGrp(*No);

Dcl-S MyArray Char(10) Dim(10);

MyArray(1) = 'Value1';
MyArray(5) = 'Value5';
```

Arrays indexes in RPG always start at 1. Whenever you’re referencing the length of an array, never use a hardcoded constant, instead use the ‘%Elem’ built-in function (elements). In this example, we will use this built-in function to clear every element in the array.

```
Dcl-S index   Int(3);
Dcl-S MyArray Char(10) Dim(10);

MyArray(1) = 'Value1';
MyArray(5) = 'Value5';

For index = 1 to %Elem(MyArray);
  MyArray(index) = *Blank;
ENDFOR;
```

While this code is valid and working, there is an easier option to clearing fields, arrays and data-structures. Instead of using this `FOR` loop to clear the, we can use the `CLEAR` operation code. The `CLEAR` operation code works on all fields, arrays, and data-structures (and subfields) – for every type too:

```
Dcl-S index   Int(3);
Dcl-S MyArray Char(10) Dim(10);

MyArray(1) = 'Value1';
MyArray(5) = 'Value5';

Clear MyArray;
```

Another useful keyword is `INZ` (initialization, US spelling). This allows the developer to give variables an initial value when they are declared. You would have already seen this used in this Lecture and will in future lectures too. If you use ‘INZ’ with ‘DIM’, it will give every element that initial value. There is a handy extra operation code, called `RESET`. `RESET` allows you to reset fields, arrays, and data-structure subfields back to their initial value (defined with the ‘INZ’ keyword):

```
Dcl-S index   Int(3);
Dcl-S MyArray Char(10) Dim(10) Inz('Value');

MyArray(1) = 'Nothing!';
MyArray(5) = 'Nothing!';

Reset MyArray;
```

## RPG Procedures

Procedures in RPG are very comparable to functions in C. If you studied Lecture 5, you would have already seen an example of a basic procedure in RPG.

Procedures can:

* Have a return type (or void)
* Have a parameter list
* Have parameter by reference, constant or value

The syntax of a procedure can be confusing at first, but after practice it will become much simpler.

```
dcl-proc name [export];
  [dcl-pi *N [returntype] [end-pi];]
    [parmname parmtype passby;]
  [end-pi;]
end-proc;
```

`Dcl-Proc` stands for 'declare procedure' and `Dcl-Pi` stands for `declare procedure-interface`. A PI is required for any procedure or program that has parameters or a return value. It’s used so the compiler knows how to invoke it correctly.

### Procedure with no parameters or return value

```
**FREE
Ctl-Opt DFTACTGRP(*No);

Dcl-S gMyVar Char(20) Inz('Hello!');

Dsply gMyVar;
MyProc();
Dsply gMyVar;

*InLR = *On;
Return;

Dcl-Proc MyProc;
  gMyVar = 'Goodbye!';
END-PROC;
```

In this example, after `MyProc()` has been called – it will update the global variable to a different value. Notice how there is no PI because there are no parameters or return value.

### Procedure with no parameters and return value

```
**FREE
Ctl-Opt DFTACTGRP(*No);

Dcl-S gMyVar Char(20) Inz('Hello!');

Dsply gMyVar;
gMyVar = MyProc();
Dsply gMyVar;

*InLR = *On;
Return;

Dcl-Proc MyProc;
  Dcl-Pi *N Char(8) End-Pi;

  Return 'Goodbye!';
END-PROC;
```

In this example, calling `MyProc()` returns a Char(8) value and assigns it to `gMyVar`. Notice how the `Dcl-Pi` and `End-Pi` are on the same line because there are no parameters for this procedure. `*N` represents nothing/blank. You are able to replace it with the procedure name, but it is not required.

### Procedure with a parameter and return value.

```
**FREE
Ctl-Opt DFTACTGRP(*No);

Dcl-S gMyVar Char(20) Inz('Hello!');

Dsply gMyVar;
gMyVar = MyProc('Goodbye');
Dsply gMyVar;

*InLR = *On;
Return;

Dcl-Proc MyProc;
  Dcl-Pi *N Char(8);
    pValue Char(20) Const;
  END-PI;

  Return %Trim(pValue) + '!';
END-PROC;
```

In this example, we call MyProc() with a parameter and returns my parameter with an exclamation mark concatenated on the end. We use the ‘const’ option here so we can pass in a constant value (or expression). Notice that ‘End-Pi’ is after the parameter definition.

## RPG subroutines

A subroutine is a block of code that can be executed in the current scope and has access to all variables in the current and global scope. Variables cannot be defined in a subroutine and subroutines must be defined after the return point of your program/procedure. Mainlines can only access subroutines within the global scope and procedures can only access subroutines in the local scope (of that procedure).

The following operations are used for subroutines:

![](https://raw.githubusercontent.com/WorksOfBarry/rpg-intro/master/assets/table5.PNG)

```
**FREE

Ctl-Opt DftActGrp(*No);

Dcl-S gMyGlobal Int(3);

gMyGlobal = 0;
Dsply ('Program start.');
Exsr GlobalSub;
MyProc();
Dsply ('Program end: ' + %Char(gMyGlobal));

*InLR = *On;
Return;

Begsr GlobalSub;
  Dsply ('Entered subroutine.');
  gMyGlobal += 1;
  Dsply ('End of subroutine.');
ENDSR;

Dcl-Proc MyProc;
  Dcl-S lMyLocal Int(3);

  lMyLocal = 0;
  Dsply ('Entered my procedure.');
  Exsr LocalSub;
  Dsply ('End of my procedure.');

  Return;

  Begsr LocalSub;
    Dsply ('Entered local subroutine.');
    lMyLocal  += 1;
    gMyGlobal += 1;
    Dsply ('End of local subroutine.');
  ENDSR;
END-PROC;                      
```

## RPG data-structures

Simply put, data-structures (a DS) in RPG are a set of fields that are grouped together (even in memory). Data-structures contain subfields which can be of any type and they can also contain sub-data-structures (A DS, within a DS). When declaring subfields, you can use any of the keywords you’d use when declaring a regular field (e.g `INZ`, `DIM`).

```
dcl-ds name [keywords];
  subfield type [keywords];
  …
end-ds;
```

```
//Non-qualified DS
Dcl-Ds MyDS;
  SubfieldA Char(10);
  SubfieldB Int(10);
  SubfieldC Packed(11:2);
END-DS;

SubfieldA = 'Subfield';
SubfieldB = 1337;
SubfieldC = 363424.75;
```

It is important to know what a qualified DS and non-qualified DS is. Qualified simply means that when you reference it, you must append the data-structure name to the left. For example, let’s make the previous DS example qualified.

```
Dcl-Ds MyDS Qualified;
  SubfieldA Char(10);
  SubfieldB Int(10);
  SubfieldC Packed(11:2);
END-DS;

MyDS.SubfieldA = 'Subfield';
MyDS.SubfieldB = 1337;
MyDS.SubfieldC = 363424.75;
```

Data-structure templating is a useful thing to know if you like to write clean code. A DS template is a data-structure that is not defined at runtime, but is known to the compiler so you can copy it’s subfields into another data-structure.

```
Dcl-Ds MyDS_Temp Template Qualified;
  SubfieldA Char(10);
  SubfieldB Int(10);
  SubfieldC Packed(11:2);
END-DS;

Dcl-Ds MyRealDS  LikeDS(MyDS_Temp);
Dcl-Ds MyOtherDS LikeDS(MyDS_Temp);

MyRealDS.SubfieldB  = 1337;
MyOtherDS.SubfieldB = 1337;
```

Note that you do not have to use `LIKEDS` on a data-structure with the `TEMPLATE` keyword – you can use it on any data-structure.

## RPG Prototypes

In RPG, each program and procedure can have an interface. We learned about procedure interfaces in a previous chapter, but we didn't learn that programs can also have a PI (procedure interface). 

You can [read more here](http://www.ibm.com/support/knowledgecenter/ssw_ibm_i_72/rzasd/freeinterface.htm) for info on procedure interface declaration.

In general, procedure interfaces for programs can: 

* Have pass by reference and const parameters.
* The value after `Dcl-PI` should be the program name.
* Cannot have a return value.


```
**FREE

Dcl-Pi XMPLE1;
  pName Char(10)
End-Pi;

Dsply pName;

*InLR = *On;
Return;
```

### Prototyping

Next we must learn about prototypes. A prototype is used so the compiler knows how to map the parameters of other procedures and programs so we can call them.

There are two types of prototypes:

* Program prototypes, which uses keyword `EXTPGM`
  * Reference to external program objects
* Procedure prototypes, which uses keyword `EXTPROC`
  * Reference to procedures in different modules within the current program
  * Reference to procedures in service programs within the specified binding directory
  * Reference to APIs provided by the operating system (`printf` or `system` for example)

The syntax is as follows

```
dcl-pr name [returntype] extpgm/extproc[()] [end-pi];
  parmname parmtype passby;
end-pr;
```

Each parameter much match to that of the program, procedure or API you are referencing. The following example is a program with a prototype for the program shown at the beginning of this chapter:

```
**FREE

Dcl-PR XMPLE1 ExtPgm;
  Name Char(10);
End-PR;

Dcl-S myName Char(10);

myName = 'Barry';

//Call your program like a procedure
XMPLE1(myName);

*InLR = *On;
Return;
```

Note that if the external program, procedure or API name is different from your prototype name: you will have to pass a parameter to `EXTPGM`/`EXTPROC` to specify the real name of what youre trying to call. For example:

```
Dcl-PR myPgm ExtPgm('XMPLE1');
  Name Char(10);
End-PR;
```

You can [read more here](http://www.ibm.com/support/knowledgecenter/ssw_ibm_i_72/rzasd/freeprototype.htm) for info on procedure prototype declaration.

## RPG modules

In this section we will use three new commands, `CRTRPGMOD`, `CRTSQLRPGI` and `CRTPGM`:

* `CRTRPGMOD` is used to create a module object. Modules can contain procedures or can be used as an entry point for programs.
* `CRTSQLRPGI` can be used to create both RPG modules and programs. We will use it to create a module in this section.
* `CRTPGM` is used to package modules together to make a program object.

Modules should be used to make modular and reusable code. Normally, procedures in modules are created with the EXPORT keyword. The EXPORT keyword means it can be referenced by other modules within the program it is part of. If your module is just a set of 'exported' procedures, it should use the NOMAIN program-header which you will see next.

For example, this module wraps two embedded SQL statements into procedures; let's call this module MATHMOD.

```
**FREE

Ctl-Opt NoMain;

Dcl-Proc Math_Pow Export;
  Dcl-Pi *N Int(10);
    pNumA Int(10) const;
    pNumB Int(10) const;
  End-pi;

  Dcl-S lResult Int(10);
  EXEC SQL SET :lResult = POWER(:pNumA, :pNumB);
  Return lResult;
End-Proc;

Dcl-proc Math_Cos Export;
  Dcl-pi *N Packed(11:8);
    pNum Int(10) Const;
  End-pi;

  Dcl-S lResult Packed(11:8);
  EXEC SQL SET :lResult = COS(:pNum);
  Return lResult;
End-proc;
```

To compile this, we can use `CRTSQLRPGI` with option `OBJTYPE(*MODULE)` so that a module is created. `CRTSQLRPGI` is similar to `CRTBNDRPG` and `CRTRPGMOD`, but we will talk more about that later.

NOMAIN is used to indicate that this module has no mainline and that there is no entry point. The other two procedures have the EXPORT keyword to indicate that they can be referenced by other modules.

To reference them, we must declare their procedure prototypes in all the modules we want to call them from. In the next example, we will create the entry module. This module will be the mainline entry point for the program object when it is called; let's call this module MYPGM.

```
**FREE

Dcl-pr Math_Pow Int(10) ExtProc('MATH_POW');
  pNumA Int(10) const;
  pNumB Int(10) const;
End-pr;

Dcl-pr Math_Cos Packed(11:8) ExtProc('MATH_COS');
  pNum Int(10) Const;
End-Pr;

Dcl-s MyPow Int(10);
Dcl-s MyCos Packed(11:8);

MyPow = math_pow(5:6);
MyCos = math_cos(12);

*InLR = *on;
Return;
```

To compile this, we can simply use CRTRPGMOD since there is no embedded SQL. 

Now that we have both modules, we can use CRTPGM. Notice how the first module on my module list is the entry module - you are also able to manually specify the entry module. I am going to call the program MYPGM.

```
CRTPGM PGM(MYLIB/MYPGM) MODULE(MYLIB/MYPGM MYLIB/MATHMOD)
```

After you have created the program, you should be able to call the program like any other:

```
CALL MYPGM
```

This really should give a brief idea of how modules can be used. You may have an order entry program, where the entry module does all the display file handling, another module might do the order entry processing and another might send data to a web service.

Know that you can includes modules written in any ILE language when creating programs.

## RPG and Embedded SQL

### How does Embedded SQL work?

Embedded SQL takes your source member, scans it for the EXEC SQL operation(?) and replaces it with RPG program calls. I/some call this step the 'pre-compile' - it's what the pre-compiler does. The pre-compiler also declares a data-structure for your use in programs, and one of the subfields is SQLSTATE for example. I use SQLSTATE to check if there are any data errors or SQL errors. SQLSTATE is a character five field, and you can [find what the data means here](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_73/rzala/rzalaccl.htm).

![](http://i.imgur.com/OmCPkD6.png)

### How do I start using it?

The first step is to ditch `CRTBNDRPG` and `CRTRPGMOD`. They are now useless for writing code with Embedded SQL, as you can use `CRTSQLRPGI` as a replacement for both of these. Also, you can start using `SQLRPGLE` as the extention for all your Embedded SQL RPG code.

To create a regular program, you use `CRTSQLRPGI` with `OBJTYPE(*PGM)` as a parameter; for a module you use `OBJTYPE(*MODULE)`. This is just my opinion though, of course you can still use the other commands, but you can still compile regular RPG with this command.

For this, we are going to use this SQL to create a new physical file/table. I did use STRSQL to create this table. I typed create table and used F4 to prompt the rest of the data in. LIAMALLAN1 is the library I made the PF in, but that is optional of course.

```sql
CREATE TABLE LIAMALLAN1/CUSTOMERS (
	CUS_ID INT NOT NULL WITH DEFAULT, 
	CUS_BAL NUMERIC (11 , 2) NOT NULL WITH DEFAULT, 
	CUS_NAME CHAR (25) NOT NULL WITH DEFAULT, 
	CUS_EMAIL CHAR (50) NOT NULL WITH DEFAULT
)
```

You can optionally use UPDDTA against the PF with insert mode to add data - or you can use SQL INSERT. Note that these are all seperate statements.

```sql
INSERT INTO LIAMALLAN1/CUSTOMERS VALUES(
	1, 
	10.25, 
	'Liam Barry',
	'mrliamallan@live.co.uk'
)

INSERT INTO LIAMALLAN1/CUSTOMERS VALUES(
	2, 
	100.66, 
	'Eric Jooka',
	'ericjooka@person.com'
)

INSERT INTO LIAMALLAN1/CUSTOMERS VALUES(
	3, 
	1123124.12, 
	'Emily Bae',
	'emilybae@hello.com'
)
```

### How do I really start using it?

Now we have some data, we can really start using Embedded SQL. So, make sure you have a test source member/steamfile to put your Embedded SQL in. Embedded SQL allows any regular DB2 statement within your source, be it DELETE, UPDATE, INSERT or SELECT.

As SELECT may be the most important one for beginners, we'll look at that first. As good practice, for every PF I use within Embedded SQL, I like to declare a data-structure (Dcl-DS) matching the PF fields. I also make it a template, incase I want to use it in multiple places.

```
//Template data structure matching CUSTOMERS file
Dcl-DS CUSTOMER_Temp Qualified Template;
  CUS_ID    Int(10);
  CUS_BAL   Packed(11:2);
  CUS_NAME  Char(25);
  CUS_EMAIL Char(50);
End-Ds;

Dcl-DS CUSTOMER LikeDS(CUSTOMER_Temp);
```

Selecting one record from the file is a simple start, and useful if you're writing something like a maintainance screen.

```
Exec SQL SELECT CUS_BAL,
                CUS_NAME
         INTO   :Customer.CUS_BAL,
                :Customer.CUS_NAME
         FROM   CUSTOMERS
         WHERE  CUS_ID = 1;
```

![](http://i.imgur.com/f8b6nAV.png)

And as you can see, it's simple to get data - very easy. What if we update data on our maintainance screen? The next snippet of code sits below the previous SELECT statement that we created in our RPG.

```
//Imagine this is the change on our screen
Customer.CUS_NAME = 'Barry James';

Exec SQL UPDATE CUSTOMERS SET
           CUS_BAL  = :Customer.CUS_BAL,
           CUS_NAME = :Customer.CUS_NAME
         WHERE CUS_ID = 1;
```

After we have compiled and ran this program, open STRSQL and SELECT * FROM CUSTOMERS..

![](http://i.imgur.com/Setqhzh.png)

### Lots of data

It's a need to SELECT more than one row at a time, luckily you can do this with cursors. Note, when using cursors: make sure you **close** the cursor when you've finished using it - you're causing yourself problems if you don't do this. Luckily, we declared our `CUSTOMER` data-structure so we can re-use it in our do-while.

You delcare your cursor with your `SELECT` statement. The syntax is a bit like `EXEC SQL DECLARE [cursor-name] CURSOR FOR [select-statement]`.

```
Exec SQL Declare Cust_Cur Cursor FOR
  SELECT *
  FROM   CUSTOMERS;
```

Even after we've closed the cursor, we are able to re-open it again - but not while it's already open. The code is commented instead of seperating it into blocks.

```
Exec SQL Declare Cust_Cur Cursor FOR
  SELECT *
  FROM   CUSTOMERS;

//Open our cursor that we defined previously.
Exec SQL Open Cust_Cur;

//'00000' = Unqualified Successful Completion
If (SQLSTATE = '00000');

  //Attemping to get the first record from
  //CUSTOMERS into our CUSTOMER data structure.
  Exec SQL Fetch Cust_Cur Into :CUSTOMER;

  //'00000' = Unqualified Successful Completion
  Dow (SQLSTATE = '00000');
    //Print some data
    printf(%Trim(Customer.CUS_NAME) + ': ' + %Char(Customer.CUS_BAL) + x'25');

     //Fetch the next record
    Exec SQL Fetch Cust_Cur Into :CUSTOMER;
  ENDDO;

ENDIF;

//Close the cursor.. WE MUST CLOSE THE CURSOR
Exec SQL Close Cust_Cur;
```

This code will loop through each record in the CUSTOMERS file and print some relevant data out. It looks something like this..

![](http://i.imgur.com/B0PyBx0.png)

### Deleting data

There is still a lot of cursors I haven't covered.. but this is a good start. The last part I will cover is deleting data from the CUSTOMERS file. It's simple, like all over Embedded SQL statements `EXEC SQL [statement]`. There are two ways I'm going to show. The first is a hardcoded CUS_ID, the second is using Customer.CUS_ID.

```
Exec SQL DELETE FROM CUSTOMERS
         WHERE CUS_ID = 1;
```

```
Customer.CUS_ID = 2;
Exec SQL DELETE FROM CUSTOMERS
         WHERE CUS_ID = :Customer.CUS_ID;
```

The ending result would remove two records from the file.

![](http://i.imgur.com/VktmscL.png)
