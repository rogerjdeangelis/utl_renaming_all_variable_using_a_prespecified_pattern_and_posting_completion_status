# utl_renaming_all_variable_using_a_prespecified_pattern_and_posting_completion_status
Renaming_all_variable_using_a_prespecified_pattern_and_posting_completion_status. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Renaming all variable using a prespecified pattern and posting completion status

    github
    https://tinyurl.com/ya3sk6wk
    https://github.com/rogerjdeangelis/utl_renaming_all_variable_using_a_prespecified_pattern_and_posting_completion_status

    SAS Forum
    https://tinyurl.com/ybc6mrlk
    https://communities.sas.com/t5/Base-SAS-Programming/How-to-update-dynamically-created-variable-names/m-p/446198

    see Soren's varlist macro on the end
      I made some minor changes (see doc on end)


    INPUT
    =====

      RULE Generate and apply this rename to work.have

         Rename = (WINDOWS_123 = WINDOWS   TEST_456 = TEST)


      WORK.HAVE total obs=2

             WINDOWS_
      Obs       123      TEST_456

       1         1           2
       2         3           4


    PROCESS  (all the code)
    =======================

      * if you rerun do not forget to rerun make data;
      data log;

          retain rc status;
          length renLst $1024 vars $32;
          retain renLst;

          do vars=%varlist(have,qstyle=DOUBLE,od=%str(,));
             renLst=catx(' ',renLst,vars,'=',scan(vars,1,"_"));
          end;

          call symputx('renLst',renLst);

          rc=dosubl('
             proc datasets lib=work ;
               modify have;
               rename
                  &renLst.;
          ');


          if rc ne 0 then status="Rename Failed     ";
          else status="Rename Completed" ;

          drop vars;
      run;quit;


    OUTPUT
    ======

      WORK.LOG total obs=1

       RC         STATUS                    RENLST

        0    Rename Completed    WINDOWS_123 = WINDOWS TEST_456 = TEST


      WORK.HAVE total obs=2

        WINDOWS    TEST

           1         2
           3         4

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    data have;
        input windows_123 test_456;
        cards4;
    1 2
    3 4
    ;;;;
    run;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    data log;

        retain rc status;
        length renLst $1024 vars $32;
        retain renLst;

        do vars=%varlist(have,qstyle=DOUBLE,od=%str(,));
           renLst=catx(' ',renLst,vars,'=',scan(vars,1,"_"));
        end;

        call symputx('renLst',renLst);

        rc=dosubl('
           proc datasets lib=work ;
             modify have;
             rename
                &renLst.;
        ');


        if rc ne 0 then status="Rename Failed     ";
        else status="Rename Completed" ;

        drop vars;
    run;quit;


    *                _ _     _
    __   ____ _ _ __| (_)___| |_
    \ \ / / _` | '__| | / __| __|
     \ V / (_| | |  | | \__ \ |_
      \_/ \__,_|_|  |_|_|___/\__|

    ;

    changed

    &od2.%qsysfunc(quote(&w))%end;
    to
    %unquote(&od2.%qsysfunc(quote(&w)))%end;

    Also changed ORACLE to DOUBLE


    /************************************************************
             Name: VarList.sas
             Type: Macro function
      Description: Returns variable list from table
       Parameters:
               Data       - Name of table <libname.>memname
               Keep=      - variables to keep
               Drop=      - variables to drop
               Qstyle=    - Quote style:
                              DOUBLE is like "Name" "Sex" "Weight"...
                              SAS is like 'Name'n 'Sex'n 'Weight'n...
                              Anything else is like Name Sex Weight...
               Od=%str( ) - Output delimiter
               prx=       - PRX expression
           Notes: An error provokes %ABORT CANCEL
        Examples:
               %put %varlist(sashelp.class,keep=_numeric_);
               %put %varlist(sashelp.class,qstyle=sas);
               %put %varlist(sashelp.class,qstyle=DOUBLE,od=%str(,));
               %put %varlist(sashelp.class,prx=/ei/i);
               %put %varlist(sashelp.class,prx=funky error); %* provokes error;

       Author: Søren Lassen, s.lassen@post.tele.dk
     **************************************************************/
     %macro varlist(data,keep=,drop=,qstyle=,od=%str( ),prx=);
       %local dsid1 dsid2 i w rc error prxid prxmatch od2;

      %let qstyle=%upcase(&qstyle);

      %let dsid1=%sysfunc(open(&data));
       %if &dsid1=0 %then %do;
         %let error=1;
         %goto done;
         %end;

      %let dsid2=%sysfunc(open(&data(keep=&keep drop=&drop)));
       %if &dsid2=0 %then %do;
         %let error=1;
         %goto done;
         %end;

      %if %length(&prx) %then %do;
         %let prxid=%sysfunc(prxparse(&prx));
         %if &prxid=. %then %do;
           %let prxid=;
           %let error=1;
           %goto done;
           %end;
         %end;
       %else %let prxmatch=1;

      %do i=1 %to %sysfunc(attrn(&dsid1,NVARS));
         %let w=%qsysfunc(varname(&dsid1,&i));
         %if %sysfunc(varnum(&dsid2,&w)) %then %do;
           %if 0&prxid %then
             %let prxmatch=%sysfunc(prxmatch(&prxid,&w));
           %if &prxmatch %then %do;
             %if SAS=&qstyle %then
               %do;&od2.%str(%')%qsysfunc(tranwrd(&w,%str(%'),''))%str(%')n%end;
             %else %if DOUBLE=&qstyle %then
               %do;%unquote(&od2.%qsysfunc(quote(&w)))%end;
             %else
               %do;&od2.&w%end;
             %let od2=&od;
             %end;
           %end;
         %end;

    %done:
       %if 0&dsid1 %then
         %let rc=%sysfunc(close(&dsid1));
       %if 0&dsid2 %then
         %let rc=%sysfunc(close(&dsid2));
       %if 0&prxid %then
         %syscall prxfree(prxid);
       %if 0&error %then %do;
         %put %sysfunc(sysmsg());
         %abort cancel;
         %end;

    %mend varlist;


