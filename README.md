# CBT797
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 797 is the combined work of Dan Dalby and Sam Golob.      *   FILE 797
//*                                                                 *   FILE 797
//*           email:  sbgolob@cbttape.org                           *   FILE 797
//*                                                                 *   FILE 797
//*           email:  zOS.JES2@Gmail.com    (Dan Dalby)             *   FILE 797
//*                                                                 *   FILE 797
//*           These programs manipulate, re-create, or alter        *   FILE 797
//*           the characteristics of your TSO "auth tables"         *   FILE 797
//*           that are pointed to by the LWA (Logon Work Area)      *   FILE 797
//*           for your TSO session.  With these programs, you       *   FILE 797
//*           have an amazing and phenomenal amount of control      *   FILE 797
//*           over those tables (unique to your own TSO session).   *   FILE 797
//*           You just have to authorize one of these programs      *   FILE 797
//*           (TSUB or LLWA or LWATMGR) in your IKJEFTE2 table,     *   FILE 797
//*           and you're on your way.                               *   FILE 797
//*                                                                 *   FILE 797
//*           These tools will help if you're installing some       *   FILE 797
//*           product which requires TSO authorization, and you     *   FILE 797
//*           want the authorization JUST FOR YOU and for NOBODY    *   FILE 797
//*           ELSE.                                                 *   FILE 797
//*                                                                 *   FILE 797
//*           Also added program ASUB which manipulates the         *   FILE 797
//*           PARMLIB-generated TSO "auth tables" that are in       *   FILE 797
//*           common storage, which are valid for the entire LPAR,  *   FILE 797
//*           and which are copied to the LWA-pointed tables that   *   FILE 797
//*           are manipulated by TSUB and the other programs in     *   FILE 797
//*           this pds.  ASUB has a syntax similar to TSUB.         *   FILE 797
//*                                                                 *   FILE 797
//*           Dan Dalby has contributed the LWATMGR program, and    *   FILE 797
//*             also the LWATEDIT program to EDIT the tables and    *   FILE 797
//*             SAVE them, thereby creating updated new tables,     *   FILE 797
//*             that are effective immediately.                     *   FILE 797
//*                                                                 *   FILE 797
//*           These programs must be APF authorized.  However,      *   FILE 797
//*           once these programs have been authorized to your      *   FILE 797
//*           TSO session, you can subsequently authorize any       *   FILE 797
//*           number of other programs, so your initial tables      *   FILE 797
//*           do not have to contain much, just one of these        *   FILE 797
//*           programs, and perhaps the STEPLIB program, from       *   FILE 797
//*           Dan Dalby also (source included in this file).        *   FILE 797
//*           You need the macros from member MACLIB to assemble    *   FILE 797
//*           STEPLIB and Dan's other programs.                     *   FILE 797
//*                                                                 *   FILE 797
//*           These contributions may be used separately, or they   *   FILE 797
//*           may be combined, for even further effectiveness.      *   FILE 797
//*           They all refer to your individual TSO userid's local  *   FILE 797
//*           copies of the four TSO authorization tables:          *   FILE 797
//*                                                                 *   FILE 797
//*               AUTHCMD   - or IKJEFTE2   csect                   *   FILE 797
//*               AUTHPGM   - or IKJEFTE8   csect                   *   FILE 797
//*               AUTHTSF   - or IKJEFTAP   csect                   *   FILE 797
//*               NOTBKGND  - or IKJEFTNS   csect                   *   FILE 797
//*                                                                 *   FILE 797
//*           The copies (of these tables) that your own TSO        *   FILE 797
//*           session uses, are pointed to by the Logon Work Area   *   FILE 797
//*           (control block LWA - described by the IKJEFLWA macro  *   FILE 797
//*           in SYS1.MODGEN), and they are unique to your own      *   FILE 797
//*           TSO session.  Each TSO session has its own copy of    *   FILE 797
//*           the authorization tables, pointed to by the LWA.      *   FILE 797
//*                                                                 *   FILE 797
//*           The programs in this file manipulate or display the   *   FILE 797
//*           authorization tables that are used by your TSO        *   FILE 797
//*           session.  Contrary to popular belief, the actual      *   FILE 797
//*           IKJEFTE2, IKJEFTE8, IKJEFTAP, and IKJEFTNS tables     *   FILE 797
//*           that are used by your session are NOT in common       *   FILE 797
//*           storage.  The common storage tables (or the IKJTABLS  *   FILE 797
//*           load module CSECTs in an APF-authorized STEPLIB in    *   FILE 797
//*           your LOGON PROC) are actually copied into your own    *   FILE 797
//*           TSO address space at LOGON time.  So everybody        *   FILE 797
//*           actually has individual copies of their own tables    *   FILE 797
//*           in their own TSO address space.  That is why, when    *   FILE 797
//*           you change these tables in common storage, you have   *   FILE 797
//*           to LOGON again for them to take effect.  However, a   *   FILE 797
//*           PARMLIB UPDATE(xx) command, at least in the later     *   FILE 797
//*           z/OS releases, will go around to all the active TSO   *   FILE 797
//*           address spaces, and will update their tables.  But    *   FILE 797
//*           this is with the caveat that if the "came from        *   FILE 797
//*           STEPLIB" bits are on in the LWAPRMLB flag in the      *   FILE 797
//*           LWA, then PARMLIB UPDATE will not overlay your LWA    *   FILE 797
//*           tables.  So we make sure to turn those bits on, when  *   FILE 797
//*           we make our new tables.                               *   FILE 797
//*                                                                 *   FILE 797
//*           So when you do a PARMLIB UPDATE, the new tables       *   FILE 797
//*           are in effect right away, at least when the           *   FILE 797
//*           current tables are not marked as coming from a        *   FILE 797
//*           STEPLIB.                                              *   FILE 797
//*                                                                 *   FILE 797
//*           Therefore, when you do a PARMLIB UPDATE, these new    *   FILE 797
//*           settings apply to almost EVERYONE who's logged on     *   FILE 797
//*           to TSO.  If you want your OWN settings, and you       *   FILE 797
//*           want to keep them in force, these programs presented  *   FILE 797
//*           here, are very effective tools.                       *   FILE 797
//*                                                                 *   FILE 797
//*       Short summary of the tools -                              *   FILE 797
//*                                                                 *   FILE 797
//*           SHOWTPVT shows all fields of the IKJTPVT control      *   FILE 797
//*           block, with their values on your system.  This        *   FILE 797
//*           control block is undocumented (for the public)        *   FILE 797
//*           by IBM.  Very useful for the system programmer.       *   FILE 797
//*           Its values are controlled by the IKJTSOxx PARMLIB     *   FILE 797
//*           member.  (See CBT File 731 for more tools.)           *   FILE 797
//*                                                                 *   FILE 797
//*           LWATMGR is a self-contained table management tool,    *   FILE 797
//*           and is described below.                               *   FILE 797
//*                                                                 *   FILE 797
//*           TSUB will manipulate existing LWA-pointed auth        *   FILE 797
//*           tables for a user's TSO session.  So TSUB does        *   FILE 797
//*           not need to allocate external files.                  *   FILE 797
//*                                                                 *   FILE 797
//*           TSUB will DISPLAY, REPLACE, BLANK or "NULLIFY" any    *   FILE 797
//*           existing table entry.  Or TSUB can display any        *   FILE 797
//*           of the entire tables.  If you BLANK OUT an entry      *   FILE 797
//*           which is not the last one in the table, then that     *   FILE 797
//*           invalidates all subsequent entries in that table.     *   FILE 797
//*           A blank table entry is always treated as a delimiter. *   FILE 797
//*           So you have to know what you are doing.  TSUB's       *   FILE 797
//*           DISPLAY functions do not have to be APF-authorized.   *   FILE 797
//*                                                                 *   FILE 797
//*           Nullify will insert C'?       ' (? followed by        *   FILE 797
//*           7 blanks) into a table entry, effectively nullifying  *   FILE 797
//*           it without invalidating the table entries following.  *   FILE 797
//*           Use action "N" instead of "B" unless you know         *   FILE 797
//*           EXACTLY the effect you want using "B".                *   FILE 797
//*                                                                 *   FILE 797
//*           TSUB can now alter the characteristics of an          *   FILE 797
//*           existing auth table.  See the functions of TSUB       *   FILE 797
//*           below:                                                *   FILE 797
//*                                                                 *   FILE 797
//*           The original action codes of TSUB are:                *   FILE 797
//*                                                                 *   FILE 797
//*           D - Display table entries by slot number.             *   FILE 797
//*           R - Replace the entry in a given slot, by another     *   FILE 797
//*               program name.                                     *   FILE 797
//*           B - Blank a given table entry, by slot number.        *   FILE 797
//*           N - Nullify a given table entry, by slot number.      *   FILE 797
//*                                                                 *   FILE 797
//*           All action codes in TSUB, except D, require           *   FILE 797
//*           APF-authorization.                                    *   FILE 797
//*                                                                 *   FILE 797
//*           The new action codes of TSUB are:                     *   FILE 797
//*                                                                 *   FILE 797
//*           H - Add PARMLIB-like table header, in 2nd 8-bytes.    *   FILE 797
//*           L - Supply length of table to first blank, in LWA.    *   FILE 797
//*           Z - Zero the table length marked in the LWA.          *   FILE 797
//*           S - Mark that the table came from STEPLIB, so         *   FILE 797
//*                someone (else's) PARMLIB UPDATE(xx) won't        *   FILE 797
//*                overlay it.  You'll need this if you modified    *   FILE 797
//*                your existing table (if originally copied from   *   FILE 797
//*                PARMLIB) and you don't want a PARMLIB UPDATE(xx) *   FILE 797
//*                to replace your changes.                         *   FILE 797
//*           P - Turn off STEPLIB bit in LWA and mark that it      *   FILE 797
//*                came from PARMLIB.  Now PARMLIB UPDATE(xx)       *   FILE 797
//*                will overlay the table.                          *   FILE 797
//*           X - (undocumented) Gives LPA storage range display.   *   FILE 797
//*                                                                 *   FILE 797
//*           LLWA will replace all, or some, of these tables       *   FILE 797
//*           completely, from various sources.  LLWA can           *   FILE 797
//*           provide the new tables from an assembled load         *   FILE 797
//*           module (similar to IKJTABLS), or from a list of       *   FILE 797
//*           program names in LRECL=8 format, or from a            *   FILE 797
//*           card-image file, that is in PARMLIB's IKJTSOxx        *   FILE 797
//*           format.                                               *   FILE 797
//*                                                                 *   FILE 797
//*           You can control the inputs and outputs to the LLWA    *   FILE 797
//*           program by manually allocating the input datasets     *   FILE 797
//*           and maybe the one possible output dataset, using      *   FILE 797
//*           the TSO ALLOCATE and FREE commands.                   *   FILE 797
//*                                                                 *   FILE 797
//*           You should Linkedit LLWA as NORENT and NOREUS.        *   FILE 797
//*           LLWA will operate more efficiently then, and will     *   FILE 797
//*           not use up so much storage in your address space      *   FILE 797
//*           when you run it.  It is also better (but not really   *   FILE 797
//*           necessary) to run LLWA in TSO READY mode, rather      *   FILE 797
//*           than under ISPF.  Since ISPF takes up the TSO user's  *   FILE 797
//*           "below 16M" storage, and LLWA needs to GETMAIN more   *   FILE 797
//*           storage for the new auth tables, it is best to run    *   FILE 797
//*           LLWA in READY mode, when more storage is available.   *   FILE 797
//*                                                                 *   FILE 797
//*           LWATMGR uses dynamic allocation throughout.           *   FILE 797
//*                                                                 *   FILE 797
//*           The functionality of LWATMGR is pretty much covered   *   FILE 797
//*           by the two programs LLWA and TSUB.  And vice-versa.   *   FILE 797
//*           But each one does a few things that the others will   *   FILE 797
//*           not do.  So it pays to be familiar with all three     *   FILE 797
//*           programs.                                             *   FILE 797
//*                                                                 *   FILE 797
//*       Description of LWATMGR -                                  *   FILE 797
//*                                                                 *   FILE 797
//*           This program represents a breakthrough in systems     *   FILE 797
//*           programmer capability, in that it allows the sysprog  *   FILE 797
//*           with access to an APF-authorized library to replace   *   FILE 797
//*           and manipulate his own TSO session's authorization    *   FILE 797
//*           tables.  It also allows the sysprog to test a new     *   FILE 797
//*           IKJTSOxx PARMLIB member on one TSO userid at a time,  *   FILE 797
//*           before letting it loose on every userid in the LPAR.  *   FILE 797
//*                                                                 *   FILE 797
//*           Let me explain.  Everybody knows that you can change  *   FILE 797
//*           the TSO authorization tables globally in your LPAR,   *   FILE 797
//*           by editing the IKJTSOxx member in PARMLIB and either  *   FILE 797
//*           using a PARMLIB UPDATE(xx) TSO command, or a SET      *   FILE 797
//*           IKJTSO=xx operator console command, to make this      *   FILE 797
//*           member active.  The PARMLIB UPDATE TSO command and    *   FILE 797
//*           the SET IKJTSO=xx operator command do essentially     *   FILE 797
//*           the same thing.  What is not generally known, is why  *   FILE 797
//*           and how these commands will affect every currently    *   FILE 797
//*           active TSO userid on the system immediately.  You     *   FILE 797
//*           don't want to do that, if you are merely testing      *   FILE 797
//*           the effect of a new IKJTSOxx member.  Running a       *   FILE 797
//*           PARMLIB UPDATE(xx) command on it, would be like       *   FILE 797
//*           putting it into production immediately.  You would    *   FILE 797
//*           like to run the new tables in some kind of "test      *   FILE 797
//*           mode" first, on one userid at a time.                 *   FILE 797
//*                                                                 *   FILE 797
//*           Enter LWATMGR.  Let me tell you an unknown fact       *   FILE 797
//*           first.  The PARMLIB-created TSO auth tables are       *   FILE 797
//*           NOT the ones that are used by your TSO session to     *   FILE 797
//*           make decisions about authorized TSO commands.         *   FILE 797
//*           During LOGON time, the public tables are COPIED       *   FILE 797
//*           into SP-252 Key 0 storage in your own TSO address     *   FILE 797
//*           space.  They are then pointed to, by fields in the    *   FILE 797
//*           Logon Work Area (the LWA), and THESE copies of the    *   FILE 797
//*           tables (IKJEFTE2, IKJEFTE8, IKJEFTAP, IKJEFTNS)       *   FILE 797
//*           are the ones your session actually uses.  LWATMGR     *   FILE 797
//*           does something that nothing else (to our              *   FILE 797
//*           knowledge) does.  It can manipulate those             *   FILE 797
//*           "LWA-pointed-to" copies of the TSO authorization      *   FILE 797
//*           tables, and its scope immediately affects ONLY the    *   FILE 797
//*           TSO session which invoked the LWATMGR program.  So    *   FILE 797
//*           when you use LWATMGR to change your TSO session's     *   FILE 797
//*           auth tables, you only affect your own session, and    *   FILE 797
//*           you do not touch the sessions belonging to the        *   FILE 797
//*           rest of your TSO world (the other sessions on your    *   FILE 797
//*           LPAR, or MVS instance).                               *   FILE 797
//*                                                                 *   FILE 797
//*           LWATMGR has several modes of operation.  LWATMGR      *   FILE 797
//*           can replace ALL of the Auth tables from an IKJTSOxx-  *   FILE 797
//*           like source member.  This is called the UPDATE        *   FILE 797
//*           function.  Input to the UPDATE function of LWATMGR    *   FILE 797
//*           is an IKJTSOxx PARMLIB member, or another FB-80       *   FILE 797
//*           file or pds member which looks like it.  You can      *   FILE 797
//*           also choose to replace only one table from the        *   FILE 797
//*           IKJTSOxx-format source member.                        *   FILE 797
//*                                                                 *   FILE 797
//*           Another thing LWATMGR can do, is to either replace    *   FILE 797
//*           one (program name) entry in a particular table, by    *   FILE 797
//*           a different program name.  This is the REPLACE        *   FILE 797
//*           option.                                               *   FILE 797
//*                                                                 *   FILE 797
//*           A third thing LWATMGR can do, is to eliminate one     *   FILE 797
//*           table entry from a particular table.  This is the     *   FILE 797
//*           DELETE option.                                        *   FILE 797
//*                                                                 *   FILE 797
//*           A fourth thing LWATMGR can do, is to add one new      *   FILE 797
//*           program name to any of the tables.  This is the ADD   *   FILE 797
//*           option of LWATMGR.                                    *   FILE 797
//*                                                                 *   FILE 797
//*           A fifth option of LWATMGR is to DISPLAY all the       *   FILE 797
//*           existing "LWA-pointed-to" auth tables, or to DISPLAY  *   FILE 797
//*           any of the tables, one at a time.  The default        *   FILE 797
//*           action of LWATMGR, entered without parameters, is to  *   FILE 797
//*           display all four tables as they actually exist for    *   FILE 797
//*           your TSO session.                                     *   FILE 797
//*                                                                 *   FILE 797
//*           LWATMGR has a sixth and a seventh option too.         *   FILE 797
//*                                                                 *   FILE 797
//*           The sixth option is called RELOAD.  RELOAD replaces   *   FILE 797
//*           the "LWA-pointed-to" tables for your session, from    *   FILE 797
//*           an IKJTABLS-like load module, which contains CSECTs   *   FILE 797
//*           IKJEFTE2, IKJEFTE8, IKJEFTAP, and IKJEFTNS.  You can  *   FILE 797
//*           assemble and linkedit your own tables of this type.   *   FILE 797
//*           (See CBT Tape File 185 to learn how to set this up.)  *   FILE 797
//*           The RELOAD option of LWATMGR is invoked AFTER the     *   FILE 797
//*           LOGON time.  But although it resembles getting your   *   FILE 797
//*           tables (at LOGON time) from an APF-authorized         *   FILE 797
//*           STEPLIB in your LOGON PROC (again see CBT File 185    *   FILE 797
//*           for more details), THIS NEW process is done AFTER     *   FILE 797
//*           the LOGON time.  And furthermore, LWATMGR's process   *   FILE 797
//*           makes the tables look almost as though they came      *   FILE 797
//*           from PARMLIB, and not as if they came from an         *   FILE 797
//*           APF-authorized STEPLIB.  (There are significant       *   FILE 797
//*           internal difference between (LWA) tables that were    *   FILE 797
//*           created from those two separate sources.              *   FILE 797
//*                                                                 *   FILE 797
//*           You can use LWATMGR RELOAD *after* you've used Dan    *   FILE 797
//*           Dalby's STEPLIB program on an APF-authorized load     *   FILE 797
//*           library to create the effect of having an             *   FILE 797
//*           authorized STEPLIB in your LOGON PROC, containing     *   FILE 797
//*           an IKJTABLS load module.  But this can be done to     *   FILE 797
//*           ANY TSO session (to which STEPLIB and LWATMGR are     *   FILE 797
//*           in the AUTHCMD table), without a STEPLIB in the       *   FILE 797
//*           LOGON PROC, and AFTER LOGON time.                     *   FILE 797
//*                                                                 *   FILE 797
//*           A seventh option of LWATMGR is the BUILD option,      *   FILE 797
//*           where you can reload some or all of the tables, not   *   FILE 797
//*           from a load module, or from an IKJTSOxx member,       *   FILE 797
//*           which may be relatively hard to keep and maintain,    *   FILE 797
//*           but from a mere list of program names, kept in a      *   FILE 797
//*           file that has LRECL = 8.  This can be either one      *   FILE 797
//*           flat file, or many members of a pds.  This kind of    *   FILE 797
//*           "authorized program list" is very compact and very    *   FILE 797
//*           easy to store.  You can keep many different "special  *   FILE 797
//*           purpose" authorization lists in a very small space,   *   FILE 797
//*           such as members of a pds.                             *   FILE 797
//*                                                                 *   FILE 797
//*           Input to the BUILD option of LWATMGR is just a list   *   FILE 797
//*           of program names, starting with headers ---E2---,     *   FILE 797
//*           ---E8---, ---NS---, or ---AP---.  These headers tell  *   FILE 797
//*           LWATMGR which tables to construct.  The record length *   FILE 797
//*           of each record is just 8 bytes, making the table      *   FILE 797
//*           extremely compact, and easy to edit.  You can keep    *   FILE 797
//*           a whole bunch of different auth tables available for  *   FILE 797
//*           different purposes, very compactly stored in a pds,   *   FILE 797
//*           and you can shuffle them "into production" in your    *   FILE 797
//*           own TSO session, using the BUILD option of LWATMGR,   *   FILE 797
//*           as needed.  Again, they are not global.  They affect  *   FILE 797
//*           only your own TSO session.                            *   FILE 797
//*                                                                 *   FILE 797
//*           Please see CBT File 452 (for the STEPLIB command),    *   FILE 797
//*           and File 185, for more information and tools to       *   FILE 797
//*           manipulate the TSO authorization tables.              *   FILE 797
//*                                                                 *   FILE 797
//*           Included in this file, is a sample LIST pds of        *   FILE 797
//*           LRECL = 8, to input into the BUILD option, a load     *   FILE 797
//*           module containing IKJTABLS with the requisite         *   FILE 797
//*           CSECTs, to input into the RELOAD option, and a        *   FILE 797
//*           sample IKJTSOxx member, which can be used as input    *   FILE 797
//*           to the UPDATE option.                                 *   FILE 797
//*                                                                 *   FILE 797
//*           The LLWA program can convert input to the tables,     *   FILE 797
//*           from IKJTSOxx "PARMLIB-format" or IKJTABLS load       *   FILE 797
//*           module format, into the more compact LRECL=8 program  *   FILE 797
//*           list format, for easier storage and manipulation.     *   FILE 797
//*                                                                 *   FILE 797
```
