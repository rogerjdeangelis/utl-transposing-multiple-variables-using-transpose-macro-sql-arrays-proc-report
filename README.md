# utl-transposing-mutiple-variables-using-transpose-macro-sql-arrays-proc-report
Transposing mutiple variables using transpose macro sql arrays proc report.

    Transposing mutiple variables using transpose macro sql arrays proc report                                              
                                                                                                                            
      Two Solutions                                                                                                         
                                                                                                                            
        1. SQL arrays                                                                                                       
        2. proc report                                                                                                      
        3. utl_transpose macro (preferred)                                                                                  
                                                                                                                            
    Output datasets are much more useful than static listings                                                               
                                                                                                                            
    utl_Transpose macro by                                                                                                  
    AUTHORS: Arthur Tabachneck, Xia Ke Shan, Robert Virgile and Joe Whitehurst                                              
                                                                                                                            
    Macros                                                                                                                  
    https://tinyurl.com/y9nfugth                                                                                            
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories                              
                                                                                                                            
    SAS Forum                                                                                                               
    https://tinyurl.com/y6vkvza7                                                                                            
    https://communities.sas.com/t5/New-SAS-User/Proc-SQL-Combine-Multiple-Row-Values-into-One-Column-Row/m-p/504076         
                                                                                                                            
                                                                                                                            
    INPUT                                                                                                                   
    =====                                                                                                                   
                                                                                                                            
    WORK.HAVE total obs=6                                                                                                   
                                                                                                                            
      ID    TYPE     AMOUNT    COUNT                                                                                        
                                                                                                                            
     111    TYPE1    100000      4                                                                                          
     111    TYPE2    403000      7                                                                                          
     111    TYPE3    230000      3                                                                                          
     222    TYPE1     40000      2                                                                                          
     222    TYPE2     90600      1                                                                                          
     222    TYPE3     20000      2                                                                                          
                                                                                                                            
                                                                                                                            
    EXAMPLE OUTPUT                                                                                                          
    ==============                                                                                                          
                                                                                                                            
     WORK.WANT total obs=2                                                                                                  
                                                                                                                            
     AMOUNT1    AMOUNT2    AMOUNT3    COUNT1    COUNT2    COUNT3                                                            
                                                                                                                            
      100000     403000     230000       4         7         3                                                              
       40000      90600      20000       2         1         2                                                              
                                                                                                                            
                                                                                                                            
    PROCESS                                                                                                                 
    =======                                                                                                                 
                                                                                                                            
      1. SQL arrays                                                                                                         
      -------------                                                                                                         
         %array(typs,values=1-3);                                                                                           
                                                                                                                            
         proc sql;                                                                                                          
            create                                                                                                          
               table want as                                                                                                
            select                                                                                                          
               %do_over(typs, phrase=%str(                                                                                  
                   sum((TYPE="TYPE?")*amount) as amount?),between=comma)                                                    
              ,%do_over(typs, phrase=%str(                                                                                  
                   sum((TYPE="TYPE?")*count) as count?),between=comma)                                                      
            from                                                                                                            
                have                                                                                                        
            group                                                                                                           
                by id                                                                                                       
         ;quit;                                                                                                             
                                                                                                                            
       2.  /* SAS should honor ODS output? */                                                                               
       --------------------------------------                                                                               
           proc report data=have out=havxpo                                                                                 
            ( rename=(  /* SAS should honor ODS output so we do not need this */                                            
                _C2_ = amount1                                                                                              
                _C3_ = count1                                                                                               
                _C4_ = amount2                                                                                              
                _C5_ = count2                                                                                               
                _C6_ = amount3                                                                                              
                _C7_ = count3                                                                                               
            )) nowd missing;                                                                                                
          cols id type, ( amount count);                                                                                    
          define id / group;                                                                                                
          define count / analysis sum;                                                                                      
          define amount /analysis sum;                                                                                      
          define type / across;                                                                                             
          run;quit;                                                                                                         
                                                                                                                            
          /*                                                                                                                
                         TYPE1                 TYPE2                 TYPE3                                                  
            ID     AMOUNT      COUNT     AMOUNT      COUNT     AMOUNT      COUNT                                            
           111     100000          4     403000          7     230000          3                                            
           222      40000          2      90600          1      20000          2                                            
                                                                                                                            
          */                                                                                                                
                                                                                                                            
       3. utl_transpose macro (preferred)                                                                                   
       ----------------------------------                                                                                   
                                                                                                                            
          %utl_transpose(data=have, out=want, by=id, id=type, guessingrows=1000,                                            
             delimiter=_, var=amount count, var_first=no);                                                                  
                                                                                                                            
    *                _              _       _                                                                               
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _                                                                        
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |                                                                       
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |                                                                       
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|                                                                       
                                                                                                                            
    ;                                                                                                                       
                                                                                                                            
    data have;                                                                                                              
      informat name $5.;                                                                                                    
      format name $5.;                                                                                                      
      input year name height weight;                                                                                        
      cards4;                                                                                                               
    2013 Dick 6.1 185                                                                                                       
    2013 Tom  5.8 163                                                                                                       
    2013 Harry 6.0 175                                                                                                      
    2014 Dick 6.1 180                                                                                                       
    2014 Tom  5.8 160                                                                                                       
    2014 Harry 6.0 195                                                                                                      
    ;;;;                                                                                                                    
    run;quit;                                                                                                               
                                                                                                                            
                                                                                                                            
