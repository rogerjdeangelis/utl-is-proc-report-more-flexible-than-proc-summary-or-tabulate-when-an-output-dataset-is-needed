# utl-is-proc-report-more-flexible-than-proc-summary-or-tabulate-when-an-output-dataset-is-needed
Is proc report more flexible than proc summary or tabulate for small tables when an output dataset is required?

    Is proc report more flexible than proc summary or tabulate for small tables when an output dataset is required?

    Good question!!

    Problem:

       Given detail data, create a report with grand total and subtotals before each grouping.
       This is important because means and percent can easily be computed in a datastep
       because all the totals preceed the associated detail data.

     Creating a specific output dataset with totals and subtotals before detail data.

      1. Proc report (best solution) by K Sharpe. As a side bonus you get a report.
      2. Proc summary - Close but less usefull
      3. Proc tabulate - Least flexible

      An output dataset is more important than the static repport?

      Note 'proc report can sort, transpose, summarize and group arbitrary statistics in almost any order'

    github
    https://tinyurl.com/yarpox56
    https://github.com/rogerjdeangelis/utl-is-proc-report-more-flexible-than-proc-summary-or-tabulate-when-an-output-dataset-is-needed

    sas forum
    https://tinyurl.com/ya2zhsqs
    https://communities.sas.com/t5/SAS-Programming/Subtotal-and-Grand-Total-in-dataset/m-p/527236

    K Sharpe Profile
    https://communities.sas.com/t5/user/viewprofilepage/user-id/18408


    INPUT
    =====

    data have;
    input Group :$20. Member :$20. Hours  Employees;
    cards4;
    Grp_a Mem_a_1 6 0
    Grp_a Mem_a_2 4 1
    Grp_b Mem_b_1 6 3
    Grp_b Mem_b_2 3 1
    ;;;;
    run;quit;


    WORK.HAVE total obs=4                        RULES (total and subtotals for Hours and employees)
                                              |
       GROUP    MEMBER     EMPLOYEES   HOURS  |   HOURS
                                              |
                                              |    19  = (6+4+6+3)
                                              |    10  = (6+4)
                                              |
       Grp_a    Mem_a_1        0         6    |     6
       Grp_a    Mem_a_2        1         4    |     4
                                              |
                                              |     9  = (6+3)
                                              |
       Grp_b    Mem_b_1        3         6    |     6
       Grp_b    Mem_b_2        1         3    |     3
                                              |
                                              |
    EXAMPLE OUTPUT
    --------------
    WORK.WANT total obs=7

     GROUP    MEMBER         EMPLLOYEES  HOURS   TOTALS

              Grand_Total         5        19    _RBREAK_
     Grp_a    Grp_a               1        10    GROUP
     Grp_a    Mem_a_1             0         6
     Grp_a    Mem_a_2             1         4
     Grp_b    Grp_b               4         9    GROUP
     Grp_b    Mem_b_1             3         6
     Grp_b    Mem_b_2             1         3


    SOLUTIONS
    =========

    --------------------------------------------
    1. Proc report (best solution) by K Sharpe
    --------------------------------------------

    proc report data=have out=want(rename=_break_=Totals) nowd;
       column  Group  Member  Hours  Employees;
       define group/group;
           compute before;
           Member='Grand_Total';
       endcomp;
       compute before group;
           Member=group;
       endcomp;
       break before group/summarize;
       rbreak before/summarize;
    run;quit;

    /*
    WORK.WANT total obs=7

     GROUP    MEMBER         EMPLLOYEES  HOURS   TOTALS

              Grand_Total         5        19    _RBREAK_
     Grp_a    Grp_a               1        10    GROUP
     Grp_a    Mem_a_1             0         6
     Grp_a    Mem_a_2             1         4
     Grp_b    Grp_b               4         9    GROUP
     Grp_b    Mem_b_1             3         6
     Grp_b    Mem_b_2             1         3
    */


    ----------------------------------------
    2. Proc summary - Close but less usefull
    ----------------------------------------

    proc summary data=have;
      class group member;
      types () group group*member;
      var hours employees;
      output out=wantSum(drop=_:) sum= / autoname;
    run;quit;

    /*
    Not as useful but is more efficient;

    WORK.WANTSUM total obs=7

                               HOURS_    EMPLOYEES_
    Obs    GROUP    MEMBER       SUM         SUM

     1                           19           5
     2     Grp_a                 10           1
     3     Grp_b                  9           4
     4     Grp_a    Mem_a_1       6           0
     5     Grp_a    Mem_a_2       4           1
     6     Grp_b    Mem_b_1       6           3
     7     Grp_b    Mem_b_2       3           1
    */


    ----------------------------------
    3. Proc tabulate - Least flexible
    ----------------------------------

    proc tabulate data=have out=wantTab(where=(_type_ < '11') drop=_page_ _table_);
    var hours employees;
    class group member;
    tables (all group)*(all member), (hours*(sum) employees*(sum));
    run;quit;

    /*
    Up to 40 obs from WANTTAB total obs=7

                                         HOURS_    EMPLOYEES_
    Obs    GROUP    MEMBER     _TYPE_      SUM         SUM

     1                           00        19           5
     2              Mem_a_1      01         6           0
     3              Mem_a_2      01         4           1
     4              Mem_b_1      01         6           3
     5              Mem_b_2      01         3           1
     6     Grp_a                 10        10           1
     7     Grp_b                 10         9           4
    */


