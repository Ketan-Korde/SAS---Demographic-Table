# Demographic Table (SAS)
As a part of online training on Clinical Trial Analysis &amp; Reporting I have completed the final project - Generating Demographic Table.

# Problem Statement
Derive the necessary dataset from RAW DATA and create the Demographic Table as shown in Mock Demographic Table.

# Mock Demographic Table

![App Screenshot](https://github.com/Ketan-Korde/SAS---Demographic-Table/blob/eff7c8cf5615c9c0e677dab945dc2c8ef02fd6ca/Mock%20Table.PNG)

# SAS Code
Importing RAW DATA in SAS

    /* Importing Excel File in SAS */

    FILENAME REFFILE '/home/u61989325/P/M4_T1_V2_-_project_-_demog.xlsx';

    PROC IMPORT DATAFILE=REFFILE
	  DBMS=XLSX
	  OUT=WORK.DEMOGP REPLACE;
	  GETNAMES=YES;
    RUN;

    PROC CONTENTS DATA=WORK.DEMOGP; 
    RUN;
  
Section 01 : Deriving Age Variable
  
    /* SECTION 01 : Deriving Age Variable */

    DATA demogp1;
        SET demogp;
        FORMAT dob1 DATE9.;
    
       /*Creating date of birth variable by concatinating month day year variables*/
          dob=CATX("/",month,day,year);
    
       /*converting data type char to numeric*/
          dob1=INPUT(dob,mmddyy10.);
    
      /*Calculating age*/
         AGE=INTCK("Year",dob1,diagdt);
    
       OUTPUT;
    
       trt=2;
       OUTPUT;
       
    RUN;
    
  Obtaining Summery Statistics for Age Variable
  
    PROC SORT DATA=demogp1;
    BY trt;
    RUN;

    PROC MEANS DATA=demogp1 NOPRINT;
    OUTPUT OUT=agestat;
    VAR age;
    BY trt;
    RUN;
    
    
    /*creating age statistic data*/
    DATA agestats;
       SET agestat;
       LENGTH value $10.;
   
        ord=1;
   
        IF _stat_="N" THEN DO;
        subord=1;
        VALUE=PUT(age,8.);
        END;
   
        IF _stat_="MEAN" THEN DO;
        subord=2;
        VALUE=PUT(age,8.1);
        END;
   
        IF _stat_="STD" THEN DO;
        subord=3;
        VALUE=PUT(age,8.2);
        END;
   
        IF _stat_="MIN" THEN DO;
        subord=4;
        VALUE=PUT(age,8.1);
        END;
   
        IF _stat_="MAX" THEN DO;
        subord=5;
        VALUE=PUT(age,8.1);
        END;
   
       RENAME _stat_=stat;
       DROP _type_ _freq_ age;
      RUN;
      
 Section 02 : Deriving Age Groups varibale
      
      PROC FORMAT;
        VALUE agegrp
          LOW-18="<=18 years"
          18-65="18 to 65 years"
          65-High=">65 years"
          ;
       RUN; 
       
       /* applying format */
       DATA demogp2;
          SET demogp1;
  
          AGE_GROUP=PUT(age,agegrp.);
    
       RUN;
       
       /* Obtaining Summery Statistics for age groups */


       PROC FREQ DATA=demogp2 NOPRINT;
         TABLE trt*age_group / OUTPCT OUT=agegrp_stat;
       RUN;
       
       /*creating age group statistic data*/
        DATA agegrp_stats;
           SET agegrp_stat;
  
               ord=2;
  
          IF age_group="<=18 years" THEN subord=1;
          IF age_group="18 to 65 years" THEN subord=2;
          IF age_group=">65 years" THEN subord=3;
  
          VALUE=CAT(count, ' (', STRIP(PUT(ROUND(pct_row,0.1),8.1)), '%)' );
  
          RENAME age_group=stat;
          DROP count percent pct_row pct_col;
  
          RUN; 
          
Section 03 : Deriving Sex Variable  

       /* Section 03 : Deriveing Sex Variable  */

       PROC FORMAT;
          VALUE genfmt
          1="Male"
          2="Female"
          ;
       RUN;
      
      /*creating sex variable and applying format created*/
       DATA demogp3;
          SET demogp1;
   
          sex=PUT(gender,genfmt.);
   
       RUN;
       
       /* Obtaining Summery Statistics for gender */

       PROC FREQ DATA=demogp3 NOPRINT;
         TABLE trt*sex / OUTPCT OUT=genderstat;
       RUN;
       
       /*creating gender statistic data*/
        DATA genderstats;
          SET genderstat;
  
           ord=3;
  
         IF sex="Male" THEN subord=1;
         IF sex="Female" THEN subord=2;
  
          VALUE=CAT(count, ' (', STRIP(PUT(ROUND(pct_row,0.1),8.1)), '%)' );
  
         RENAME sex=stat;
         DROP count percent pct_row pct_col; 
        RUN;  
        
        
Section 04 : Deriving Race Variable
        
      PROC FORMAT;
         VALUE racefmt
          1="White"
          2="African American"
          3="Hispanic"
          4="Asian"
          5="Other"
          ;
       RUN;
       
      DATA demogp4;
        SET demogp1;
   
        race_c=PUT(race,racefmt.);
   
      RUN;
      
      /* Obtaining summery statistics for race    */
      PROC FREQ DATA=demogp4 NOPRINT;
        TABLE trt*race_c / OUTPCT OUT=racestat;
      RUN;
     
     /*creating race statistic data*/
      DATA racestats;
        SET racestat;
   
         ord=4;
   
        IF race_c="Asian" THEN subord=1;
        IF race_c= "African American" THEN subord=2;
        IF race_c="Hispanic" THEN subord=3;
        IF race_c="White" THEN subord=4;
        IF race_c="Other" THEN subord=5;
   
        VALUE=CAT(count,' (', STRIP(PUT(ROUND(pct_row,0.1),8.1)),'%)');
   
        RENAME race_c=stat;
        DROP count percent pct_row pct_col;
       RUN;
       
 Merging above all four dataset
       
      /* creating allstats  */

       DATA allstat;
        LENGTH stat $20.;
        SET agestats agegrp_stats genderstats racestats;
 
       RUN; 
       
       /*Transposing Data*/

      PROC SORT DATA=allstat;
        BY ord subord stat;
      RUN;

Transposing Data

      PROC TRANSPOSE DATA=allstat OUT=t_allstat PREFIX=_;
       ID trt;
       VAR value;
       BY ord subord stat;
      RUN;
      
Final Modifiactions Dataset

      /* Final Modification */
      PROC SORT DATA=t_allstat;
        BY ord subord;
      RUN;

     DATA stats;
      LENGTH stat $20.;
      SET t_allstat;
      BY ord subord;
     OUTPUT;
  
      IF FIRST.ord THEN DO;
      IF ord=1 THEN stat="Age (years)";
      IF ord=2 THEN stat="Age Groups";
      IF ord=3 THEN stat="Gender";
      IF ord=4 THEN stat="Race";
      subord=0;
      _0='';
      _1='';
      _2='';
  
      OUTPUT;
      END;
  
    RUN;   

Calculating Value of N as shown in mock table

    /*calculating N for each treatment using SQL and Macro*/
 
    PROC SQL NOPRINT;
     SELECT COUNT(*) INTO : Placebo
     FROM demogp1
      WHERE trt=0;
 
     SELECT COUNT(*) INTO : Active
     FROM demogp1
      WHERE trt=1;
 
     SELECT COUNT(*) INTO : Total
     FROM demogp1
      WHERE trt=2;
 
    QUIT;

Creating Final Report

    /* Generating Final Report */
 
    TITLE1 "Table 1.1";
    TITLE2 "Demographic and Baseline Chracteristics by Tratment Group";
    TITLE3 "Randomized Population";

    FOOTNOTE "Note: Percentages are based on the numeric of non-missing values in each treatment group.";

    PROC REPORT DATA=stats OUT=final SPLIT="|";
     COLUMNS ord subord stat _0 _1 _2;
       DEFINE ord / NOPRINT ORDER;
       DEFINE subord / NOPRINT ORDER;
       DEFINE stat / DISPLAY WIDTH=50 "";
       DEFINE _0 / "Placebo | (N = &Placebo)";
       DEFINE _1 / "Active Treatment | (N = &Active)";
       DEFINE _2 / "All Patients | (N = &Total)";
     RUN;
     
 # Result    
 
 ![App Screenshot](https://github.com/Ketan-Korde/SAS---Demographic-Table/blob/eff7c8cf5615c9c0e677dab945dc2c8ef02fd6ca/Result.PNG)
