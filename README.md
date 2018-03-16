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

