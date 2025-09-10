# utl-create-empty-table-with-longest-column-attributes-when-setting-mutiple-tables
Create empty table with longest column attributes when setting mutiple tables
    %let pgm=utl-create-empty-table-with-longest-column-attributes-when-setting-mutiple-tables;

    %stop_submission;

    Create empty table with longest column attributes when setting mutiple tables

    PROBLEM

       When settng mutiple tables like

       data apnd;
          set a b c;
       run;quit;

       SAS uses the attributes of table 'a' to set column lengths, which can cause truncation.

       This macro create a empty template table with the longest lengths

       data spnd;
         set
             %utl_attrTemplate(a b c)  /*-- resolves to empty table work._template_ with longest lengths ---*/
             a
             b
             c;
       run;quit;

    github
    https://tinyurl.com/yhj3nazm
    https://github.com/rogerjdeangelis/utl-create-empty-table-with-longest-column-attributes-when-setting-mutiple-tables


    /**************************************************************************************************************************/
    /* INPUT           |  PROCESS                             | OUTPUT                                                        */
    /* =====           |  =======                             | ======                                                        */
    /*                 |                                      |                                                               */
    /* WORK.SHORT      |  data want;                          | Name Type Len                                                 */
    /*                 |    set                               |                                                               */
    /* Name Type  Len  |     %utl_attrTemplate(long short)    | NAME Char 16  from second dataset                             */
    /*                 |     short                            | SEX  Char  3  from first dataset                              */
    /* NAME Char    7  |     long;                            | AGE  Num   8  from second dataset                             */
    /* SEX  Char    3  |  run;quit;                           |                                                               */
    /* AGE  Num     3  |                                      |                                                               */
    /*                 |  MACRO                               |                                                               */
    /*                 |                                      |                                                               */
    /* WORK.LONG       |  %macro utl_attrTemplate(dsns);      |                                                               */
    /*                 |                                      |                                                               */
    /* Name Type  Len  |  %dosubl(%nrstr(                     |                                                               */
    /*                 |                                      |                                                               */
    /* NAME Char   16  |    %array(_ds,values=&dsns);         |                                                               */
    /* SEX  Char    1  |                                      |                                                               */
    /* AGE  Num     8  |    proc sql;                         |                                                               */
    /*                 |      create                          |                                                               */
    /* data short;     |        table _template_ as           |                                                               */
    /*  length name $7 |      %do_over(_ds                    |                                                               */
    /*  sex $3         |       ,phrase=%str(                  |                                                               */
    /*  age 3;         |      select                          |                                                               */
    /*  input          |        *                             |                                                               */
    /*    name$        |      from                            |                                                               */
    /*    sex$ age;    |        ?(obs=0))                     |                                                               */
    /* cards4;         |     ,between=union all)              |                                                               */
    /* Alfred  M 14    |    ;quit;                            |                                                               */
    /* Alice   F 13    |    ))                                |                                                               */
    /* Barbara F 13    |    _template_                        |                                                               */
    /* Carol   F 14    |                                      |                                                               */
    /* Henry   M 14    |  %mend utl_attrTemplate;             |                                                               */
    /* James   M 12    |                                      |                                                               */
    /* ;;;;            |                                      |                                                               */
    /* run;quit;       |                                      |                                                               */
    /*                 |                                      |                                                               */
    /* data long;      |                                      |                                                               */
    /*  length name $16|                                      |                                                               */
    /*   sex $1 age 8; |                                      |                                                               */
    /*   input         |                                      |                                                               */
    /*     name$       |                                      |                                                               */
    /*     sex$ age;   |                                      |                                                               */
    /* cards4;         |                                      |                                                               */
    /* Alfred  M 14    |                                      |                                                               */
    /* Alice   F 13    |                                      |                                                               */
    /* Barbara F 13    |                                      |                                                               */
    /* Carol   F 14    |                                      |                                                               */
    /* Henry   M 14    |                                      |                                                               */
    /* ;;;;            |                                      |                                                               */
    /* run;quit;       |                                      |                                                               */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    data short;
     length name $7
     sex $3
     age 3;
     input
       name$
       sex$ age;
    cards4;
    Alfred  M 14
    Alice   F 13
    Barbara F 13
    Carol   F 14
    Henry   M 14
    James   M 12
    ;;;;
    run;quit;

    data long;
     length name $16
      sex $1 age 8;
      input
        name$
        sex$ age;
    cards4;
    Alfred  M 14
    Alice   F 13
    Barbara F 13
    Carol   F 14
    Henry   M 14
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*| WORK.SHORT                         |  WORK.LONG                                                                       */
    /*|   NAME   SEX AGE   Name Type  Len  |    NAME   SEX AGE  Name Type  Len                                                */
    /*|  Alfred   M   14   NAME Char    7  |   Alfred   M   14  NAME Char   16                                                */
    /*|  Alice    F   13   SEX  Char    3  |   Alice    F   13  SEX  Char    1                                                */
    /*|  Barbara  F   13   AGE  Num     3  |   Barbara  F   13  AGE  Num     8                                                */
    /*|  Carol    F   14                   |   Carol    F   14                                                                */
    /*|  Henry    M   14                   |   Henry    M   14                                                                */
    /*|  James    M   12                   |                                                                                  */
    /**************************************************************************************************************************/

    /*--- for testing ---*/
    proc datasets lib=work;
      delete _template_ want;
    run;quit;

    %deletesasmacn;
    %arraydelete(_ds);

    data want;
      set
       %utl_attrTemplate(long short)
       short
       long;
    run;quit;

    /*--- if you want to cleanup ---*/
    %arraydelete(_ds);

    /**************************************************************************************************************************/
    /*     NAME      SEX    AGE    Name Type  Len                                                                             */
    /*   Alfred      M      14    NAME Char  16                                                                               */
    /*   Alice       F      13    SEX  Char   3                                                                               */
    /*   Barbara     F      13    AGE  Num    8                                                                               */
    /*   Carol       F      14                                                                                                */
    /*   Henry       M      14                                                                                                */
    /*   James       M      12                                                                                                */
    /*   Alfred      M      14                                                                                                */
    /*   Alice       F      13                                                                                                */
    /*   Barbara     F      13                                                                                                */
    /*   Carol       F      14                                                                                                */
    /*   Henry       M      14                                                                                                */
    /*                                                                                                                        */
    /* LOG                                                                                                                    */
    /*                                                                                                                        */
    /* NOTE: There were 0 observations read from the data set WORK._TEMPLATE_.                                                */
    /* NOTE: There were 6 observations read from the data set WORK.SHORT.                                                     */
    /* NOTE: There were 5 observations read from the data set WORK.LONG.                                                      */
    /* NOTE: The data set WORK.WANT has 11 observations and 3 variables.                                                      */
    /**************************************************************************************************************************/

    /*
     _ __ ___   __ _  ___ _ __ ___
    | `_ ` _ \ / _` |/ __| `__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    */

    filename ft15f001 "c:/oto/utl_attrTemplate.sas";
    parmcards4;
    %macro utl_attrTemplate(dsns);

    %dosubl(%nrstr(

      %array(_ds,values=&dsns);

      proc sql;
        create
          table _template_ as
        %do_over(_ds
         ,phrase=%str(
        select
          *
        from
          ?(obs=0))
       ,between=union all)
      ;quit;
      ))
      _template_

    %mend utl_attrTemplate;
    ;;;;
    run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
