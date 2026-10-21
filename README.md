# jnr-jenner-analytics-part-VI-proc-R-macro-wrapper-and-resolving-macro-triggers-within-R
enner Analytics part VI proc R macro wrapper and resolving macro triggers within R
    /*---
    c:/utl/jnr-jenner-analytics-part-VI-proc-R-macro-wrapper-and-resolving-macro-triggers-within-R.sas
    ---*/
    
    Jenner Analytics part VI proc R macro wrapper and resolving macro triggers within R
    
    Too long to post see
    https://github.com/rogerjdeangelis/jnr-jenner-analytics-part-VI-proc-R-macro-wrapper-and-resolving-macro-triggers-within-R
    
    SOAPBOX ON
    SAS and the SLC do not resolve macro triggers in Python and R code; Jenner does.
    This is a big deal, especially using Jenner macros to generate Python and R code.
    Jenner allows calling R within a Jenner datastep.
    SOAPBOX OFF
    
    CONTENTS
    
      1 Create input parquet file
      2 Proc R vanilla interface
      3 Proc R resolving macro variable in the R script
      4 Proc R macro wrapper
      5 Calling R within a Jenner datastep
    
    STATUS
    https://github.com/rogerjdeangelis/jnr-jenner-analytics-part-I-inline-drop-down-to-duckdb-sql
    https://github.com/rogerjdeangelis/jnr-jenner-analytics-part-II-drop-down-to-open-source-spss-run-regression
    https://github.com/rogerjdeangelis/jnr-jenner-analytics-part-III-inline-drop-down-to-open-source-matlab-summarize-data
    https://github.com/rogerjdeangelis/jnr-jenner-analytics-part-IV-inline-drop-down-to-poweshell-add-column-to-excell-sheet
    https://github.com/rogerjdeangelis/nr-jenner-analytics-part-V-proc-R-macro-wrapper-and-resolving-macro-triggers-within-R
    https://github.com/rogerjdeangelis/jnr-jenner-analytics-part-VI-proc-R-macro-wrapper-and-resolving-macro-triggers-within-R
    
    /******************************************************************************************************************/
    /* 1  create input parquet file                                                                                   */
    /******************************************************************************************************************/
    
    %utlfkil(d:/parquet/class.parquet);
    
    libname prq parquet "d:/parquet";
    
    options validvarname=v7;
    
    data prq.class;
      informat
         name $8.
         sex $1.
         age 8.
         height 8.
         weight 8.
    ;
    input name sex age height weight;
    cards4;
    Alfred M 14 69 112.5
    Alice F 13 56.5 84
    Barbara F 13 65.3 98
    Carol F 14 62.8 102.5
    Henry M 14 63.5 102.5
    ;;;;
    run;
    
    /******************************************************************************************************************/
    /* 2  proc R vanilla interface                                                                               */
    /*    code also outputs a dill pickel on the pthon workspace dill.dump_module("d:/pkl/workspace.pkl")             */
    /*                                                                                                                */
    /*  Why use your own installed R                                                                             */
    /*    1 you can add packages beyond the default collection of packages                                            */
    /*    2 run other versions of R minconda, anacondat, and wsl ubuntu R                                   */
    /******************************************************************************************************************/
    
    %utlfkil(d:/parquet/want.parquet);
    
    /*--- YOU NEED THIS TO USE YOUR OWN R ----*/
    options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
    proc r;
    submit;
    library(arrow)
    library(sqldf)
    cat(R.version.string, "\n")
    have <- arrow::read_parquet("D:/parquet/class.parquet")
    
    want <- sqldf('
      select
           sex
          ,avg(age)    as avgAge
          ,avg(height) as avgHgt
          ,avg(weight) as avgWgt
      from
          have
      group
          by sex
      order
          by sex;
      ');
    
    print(want)
    
    arrow::write_parquet(
      want,
      sink = "D:/parquet/want.parquet",
      compression = "snappy"
    )
    endsubmit;
    run;
    
    libname prq parquet "d:/parquet";
    
    options nocenter;
    proc print data=prq.want;
    format _numeric_ 6.2;
    title "Back to jenner";
    run;
    
    
    FROM LOG: 10:44:22
    ==================
    
    R version 4.5.2 (2025-10-31 ucrt)
    
      sex   avgAge   avgHgt    avgWgt
    1   F 13.33333 61.53333  94.83333
    2   M 14.00000 66.25000 107.50000
    
    LIST: 10:44:22
    ==============
    
    Back to jenner
    
    Obs  sex  avgAge  avgHgt  avgWgt
    
      1  F     13.33   61.53   94.83
      2  M     14.00   66.25  107.50
    
    FULL LOG:  10:59:21
    ===================
    
    NOTE: Copyright (c) 2026 Jenner Analytics Ltd., London, England.
    NOTE: Jenner v1.5.102 (build v1.5.102+67ce60c837.20261020T035203Z)
          Licensed to Roger DeAngelis, Serial 3BF7B3E6.
    NOTE: DATA _null_
    
    autexec started.
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Library WORKX assigned path=d:\wpswrkx.
    NOTE: DATA workx.class
    
    NOTE: Processing inline DATALINES (19 lines)
    
    NOTE: Read 19 rows from DATALINES.
    NOTE: The data set WORKX.CLASS has 19 observations and 5 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_
    
    LOG:  10:59:21
    NOTE: DATA _null_ completed. Output written to FILE PRINT
    NOTE: Option SASAUTOS changed to c:/otojnr.
    NOTE: DATA _null_
    
    NOTE: Reading from fileref c:/jnr/runsas_selection.sas (c:/jnr/runsas_selection.sas)
        1 %utlfkil(d:/parquet/want.parquet);
        2
        3 /*--- YOU NEED THIS TO USE YOUR OWN R ----*/
        4 options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
        5 proc r;
        6 submit;
        7 library(arrow)
        8 library(sqldf)
        9 cat(R.version.string, "\n")
       10 have <- arrow::read_parquet("D:/parquet/class.parquet")
       11
       12 want <- sqldf('
       13   select
       14        sex
       15       ,avg(age)    as avgAge
       16       ,avg(height) as avgHgt
       17       ,avg(weight) as avgWgt
       18   from
       19       have
       20   group
       21       by sex
       22   order
       23       by sex;
       24   ');
       25
       26 print(want)
       27
       28 arrow::write_parquet(
       29   want,
       30   sink = "D:/parquet/want.parquet",
       31   compression = "snappy"
       32 )
       33 endsubmit;
       34 run;
       35
       36 libname prq parquet "d:/parquet";
       37
       38 options nocenter;
       39 proc print data=prq.want;
       40 format _numeric_ 6.2;
       41 title "Back to jenner";
       42 run;
    
    NOTE: Read 42 rows from c:/jnr/runsas_selection.sas.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_
    
    autoexec completed  10:59:21
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    The file d:/parquet/want.parquet does not exist
    NOTE: Option SET changed to RHOME C:\Progra~1\R\R-4.5.2\bin\r.
    NOTE: R initialized.
    Warning message:
    package 'arrow' was built under R version 4.5.3
    
    Attaching package: 'arrow'
    
    Loading required package: gsubfn
    Loading required package: proto
    Loading required package: RSQLite
    Warning message:
    package 'RSQLite' was built under R version 4.5.3
    R version 4.5.2 (2025-10-31 ucrt)
      sex   avgAge   avgHgt    avgWgt
    1   F 13.33333 61.53333  94.83333
    2   M 14.00000 66.25000 107.50000
    NOTE: Library PRQ assigned path=d:/parquet.
    NOTE: PROC PRINT data=prq.want
    
    NOTE: PROC PRINT completed: 2 observations printed, 4 variables
    
    /******************************************************************************************************************/
    /* 3  resolve macro triggers in your R code. Big deal to create addition R code?                        */
    /******************************************************************************************************************/
    
    %utlfkil(d:/parquet/want.parquet);
    
    %let inp=d:/parquet/class.parquet;
    %let out=d:/parquet/want.parquet;
    
    /*--- R EXECUTES THE SAME EXACT CODE THAT PROC SQL WOULD EXECUTE ---*/
    %macro procsql;
          select
           sex
          ,avg(age)    as avgAge
          ,avg(height) as avgHgt
          ,avg(weight) as avgWgt
      from
          have
      group
          by sex
      order
          by sex;
    %mend procsql;
    
    /*--- you need to set this path if you want to:
        1. use packages not available with builtin collection of packages
        2. for miniconda, anaconda or wsl ubuntu R ---*/
    
    options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
    proc R;
    submit;
    library(sqldf)
    library(arrow)
    
    have <- arrow::read_parquet("D:/parquet/class.parquet")
    want <- sqldf(" %procsql ")
    want
    
    arrow::write_parquet(
      want,
      sink = "D:/parquet/want.parquet",
      compression = "snappy"
    )
    
    endsubmit;
    run;
    
    libname prq parquet "d:/parquet";
    
    options nocenter;
    proc print data=prq.want;
    format _numeric_ 6.2;
    title "Back to jenner";
    run;
    
    FROM LOG:  11:16:31
    ===================
    R
      sex   avgAge   avgHgt    avgWgt
    1   F 13.33333 61.53333  94.83333
    2   M 14.00000 66.25000 107.50000
    
    
    LIST: 11:16:31
    ==============
    
    Back to jenner
    
      Obs  sex  avgAge  avgHgt  avgWgt
    
        1  F     13.33   61.53   94.83
        2  M     14.00   66.25  107.50
    
    LOG:  11:16:31
    ==============
    
    NOTE: Copyright (c) 2026 Jenner Analytics Ltd., London, England.
    NOTE: Jenner v1.5.102 (build v1.5.102+67ce60c837.20261020T035203Z)
          Licensed to Roger DeAngelis, Serial 3BF7B3E6.
    NOTE: DATA _null_
    
    autexec started.
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Library WORKX assigned path=d:\wpswrkx.
    NOTE: DATA workx.class
    
    NOTE: Processing inline DATALINES (19 lines)
    
    NOTE: Read 19 rows from DATALINES.
    NOTE: The data set WORKX.CLASS has 19 observations and 5 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.02 seconds
    NOTE: DATA _null_
    
    LOG:  11:16:31
    NOTE: DATA _null_ completed. Output written to FILE PRINT
    NOTE: Option SASAUTOS changed to c:/otojnr.
    NOTE: DATA _null_
    
    NOTE: Reading from fileref c:/jnr/runsas_selection.sas (c:/jnr/runsas_selection.sas)
        1 %utlfkil(d:/parquet/want.parquet);
        2
        3 %let inp=d:/parquet/class.parquet;
        4 %let out=d:/parquet/want.parquet;
        5
        6 /*--- R EXECUTES THE SAME EXACT CODE THAT PROC SQL WOULD EXECUTE ---*/
        7 %macro procsql;
        8       select
        9        sex
       10       ,avg(age)    as avgAge
       11       ,avg(height) as avgHgt
       12       ,avg(weight) as avgWgt
       13   from
       14       have
       15   group
       16       by sex
       17   order
       18       by sex;
       19 %mend procsql;
       20
       21 /*--- you need to set this path if you want to:
       22     1. use packages not available with builtin collection of packages
       23     2. for miniconda, anaconda or wsl ubuntu R ---*/
       24
       25 options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
       26 proc R;
       27 submit;
       28 library(sqldf)
       29 library(arrow)
       30
       31 have <- arrow::read_parquet("D:/parquet/class.parquet")
       32 want <- sqldf(" %procsql ")
       33 want
       34
       35 arrow::write_parquet(
       36   want,
       37   sink = "D:/parquet/want.parquet",
       38   compression = "snappy"
       39 )
       40
       41 endsubmit;
       42 run;
       43
       44 libname prq parquet "d:/parquet";
       45
       46 options nocenter;
       47 proc print data=prq.want;
       48 format _numeric_ 6.2;
       49 title "Back to jenner";
       50 run;
    
    NOTE: Read 50 rows from c:/jnr/runsas_selection.sas.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_
    
    autoexec completed  11:16:31
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Option SET changed to RHOME C:\Progra~1\R\R-4.5.2\bin\r.
    NOTE: R initialized.
    Loading required package: gsubfn
    Loading required package: proto
    Loading required package: RSQLite
    Warning message:
    package 'RSQLite' was built under R version 4.5.3
    Warning message:
    package 'arrow' was built under R version 4.5.3
    
    Attaching package: 'arrow'
    
      sex   avgAge   avgHgt    avgWgt
    1   F 13.33333 61.53333  94.83333
    2   M 14.00000 66.25000 107.50000
    NOTE: Library PRQ assigned path=d:/parquet.
    NOTE: PROC PRINT data=prq.want
    
    NOTE: PROC PRINT completed: 2 observations printed, 4 variables
    
    
    /******************************************************************************************************************/
    /* 4  macro wrapper for R code                                                                               */
    /******************************************************************************************************************/
    
    /*--- EXAMPLE AVERAGES FOR FEMALES ONLY ---*/
    
    %utlfkil(d:/parquet/want.parquet);
    
    %macro runr(
      gender=F
      ,inp=d:/parquet/class.parquet
      ,out=d:/parquet/want.parquet
      )/des="Example of wrapping R is a Kenner macro";
    
        options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
        proc r;
        submit;
        library(arrow)
        library(sqldf)
        cat(R.version.string, "\n")
        have <- arrow::read_parquet("&inp")
    
        want <- sqldf('
          select
               sex
              ,avg(age)    as avgAge
              ,avg(height) as avgHgt
              ,avg(weight) as avgWgt
          from
              have
          where
              sex ="F"
          ');
    
        print(want)
    
        arrow::write_parquet(
          want,
          sink = "&out",
          compression = "snappy"
        )
        endsubmit;
    run;
    
    libname prq parquet "d:/parquet";
    
    options nocenter;
    proc print data=prq.want;
    format _numeric_ 6.2;
    title "Back to jenner";
    run;
    
    %mend runr;
    
    %runr;
    
    LIST: 11:28:45
    ==============
    
    R version 4.5.2 (2025-10-31 ucrt)
      sex   avgAge   avgHgt   avgWgt
    1   F 13.33333 61.53333 94.83333
    
    Back to jenner
    
      Obs  sex  avgAge  avgHgt  avgWgt
    
        1  F     13.33   61.53   94.83
    
    LOG:  11:28:45
    ==============
    
    NOTE: Copyright (c) 2026 Jenner Analytics Ltd., London, England.
    NOTE: Jenner v1.5.102 (build v1.5.102+67ce60c837.20261020T035203Z)
          Licensed to Roger DeAngelis, Serial 3BF7B3E6.
    NOTE: DATA _null_
    
    autexec started.
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Library WORKX assigned path=d:\wpswrkx.
    NOTE: DATA workx.class
    
    NOTE: Processing inline DATALINES (19 lines)
    
    NOTE: Read 19 rows from DATALINES.
    NOTE: The data set WORKX.CLASS has 19 observations and 5 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_
    
    LOG:  11:28:45
    NOTE: DATA _null_ completed. Output written to FILE PRINT
    NOTE: Option SASAUTOS changed to c:/otojnr.
    NOTE: DATA _null_
    
    NOTE: Reading from fileref c:/jnr/runsas_selection.sas (c:/jnr/runsas_selection.sas)
        1 %utlfkil(d:/parquet/want.parquet);
        2
        3 %macro runr(
        4   gender=F
        5   ,inp=d:/parquet/class.parquet
        6   ,out=d:/parquet/want.parquet
        7   )/des="Example of wrapping R is a Kenner macro";
        8
        9     options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
       10     proc r;
       11     submit;
       12     library(arrow)
       13     library(sqldf)
       14     cat(R.version.string, "\n")
       15     have <- arrow::read_parquet("&inp")
       16
       17     want <- sqldf('
       18       select
       19            sex
       20           ,avg(age)    as avgAge
       21           ,avg(height) as avgHgt
       22           ,avg(weight) as avgWgt
       23       from
       24           have
       25       where
       26           sex ="F"
       27       ');
       28
       29     print(want)
       30
       31     arrow::write_parquet(
       32       want,
       33       sink = "&out",
       34       compression = "snappy"
       35     )
       36     endsubmit;
       37 run;
       38
       39 libname prq parquet "d:/parquet";
       40
       41 options nocenter;
       42 proc print data=prq.want;
       43 format _numeric_ 6.2;
       44 title "Back to jenner";
       45 run;
       46
       47 %mend runr;
       48
       49 %runr;
    
    NOTE: Read 49 rows from c:/jnr/runsas_selection.sas.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.02 seconds
    NOTE: DATA _null_
    
    autoexec completed  11:28:45
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Option SET changed to RHOME C:\Progra~1\R\R-4.5.2\bin\r.
    NOTE: R initialized.
    Warning message:
    package 'arrow' was built under R version 4.5.3
    
    Attaching package: 'arrow'
    
    Loading required package: gsubfn
    Loading required package: proto
    Loading required package: RSQLite
    Warning message:
    package 'RSQLite' was built under R version 4.5.3
    R version 4.5.2 (2025-10-31 ucrt)
      sex   avgAge   avgHgt   avgWgt
    1   F 13.33333 61.53333 94.83333
    NOTE: Library PRQ assigned path=d:/parquet.
    NOTE: PROC PRINT data=prq.want
    
    NOTE: PROC PRINT completed: 1 observations printed, 4 variables
    
    /******************************************************************************************************************/
    /* 5  calling R within a Jenner datastep                                                                          */
    /******************************************************************************************************************/
    
    %macro runr2(arg);
    options set=PROC_PYPATH "D:\py314\R.exe";
    proc R;
    submit;
    area <- pi * &arg * &arg;
    cat("radius =", &arg, "area =", area, "\n")
    endsubmit;
    run;
    %mend runr2;
    
    %symdel radii / nowarn;
    
    data _null_;
      do radii=1 to 4;
        call symputx('radii',radii,'G');
        rc=dosubl('%runr2(&radii)');
      end;
    run;
    
    
    FROM LOG:  12:09:01
    ===================
    
    radius = 1 area = 3.141
    radius = 2 area = 12.57
    radius = 3 area = 28.27
    radius = 4 area = 50.27
    
    
    LOG:  15:23:11
    ==============
    
    NOTE: Copyright (c) 2026 Jenner Analytics Ltd., London, England.
    NOTE: Jenner v1.5.102 (build v1.5.102+67ce60c837.20261020T035203Z)
          Licensed to Roger DeAngelis, Serial 3BF7B3E6.
    NOTE: DATA _null_
    
    autexec started.
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Library WORKX assigned path=d:\wpswrkx.
    NOTE: DATA workx.class
    
    NOTE: Processing inline DATALINES (19 lines)
    
    NOTE: Read 19 rows from DATALINES.
    NOTE: The data set WORKX.CLASS has 19 observations and 5 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.02 seconds
    NOTE: DATA _null_
    
    LOG:  12:09:01
    NOTE: DATA _null_ completed. Output written to FILE PRINT
    NOTE: Option SASAUTOS changed to c:/otojnr.
    NOTE: DATA _null_
    
    NOTE: Reading from fileref c:/jnr/runsas_selection.sas (c:/jnr/runsas_selection.sas)
        1 %macro runr2(arg);
        2 options set=PROC_PYPATH "D:\py314\R.exe";
        3 proc R;
        4 submit;
        5 area <- pi * &arg * &arg;
        6 cat("radius =", &arg, "area =", area, "\n")
        7 endsubmit;
        8 run;
        9 %mend runr2;
       10
       11 %symdel radii / nowarn;
       12
       13 data _null_;
       14   do radii=1 to 4;
       15     call symputx('radii',radii,'G');
       16     rc=dosubl('%runr2(&radii)');
       17   end;
       18 run;
    
    NOTE: Read 18 rows from c:/jnr/runsas_selection.sas.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_
    
    autoexec completed  12:09:01
    
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_
    
    NOTE: Option SET changed to PROC_PYPATH D:\py314\R.exe.
    NOTE: R initialized.
    radius = 1 area = 3.141593
    NOTE: Option SET changed to PROC_PYPATH D:\py314\R.exe.
    NOTE: R initialized.
    radius = 2 area = 12.56637
    NOTE: Option SET changed to PROC_PYPATH D:\py314\R.exe.
    NOTE: R initialized.
    radius = 3 area = 28.27433
    NOTE: Option SET changed to PROC_PYPATH D:\py314\R.exe.
    NOTE: R initialized.
    radius = 4 area = 50.26548
    
    NOTE: DATA elapsed:
      wall  2.79 seconds
      cpu   0.38 seconds
    
    
    /*--- END ---*/
