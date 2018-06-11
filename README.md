# utl_how_to_reuse_hash_table_without_reloading_sort_of
How to reuse hash table. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    How to reuse hash table

    This is an incomplete example because I lost the input dataset.
    If I have time I will recraete the input and

    see
    https://stackoverflow.com/questions/50748774/how-to-reuse-hash-table-in-sas

    /* T606930 Persistent reusable HASH

    The main weakness of hash objects is that
    they cannot be saved and reused.
    You have to reload each time you do the lookup.

    The hash table is loaded once and two
    separate lookups and reports are created
    without reloadng the hash table.

    Below is a technique that supports a persistent hash.

    Timings ( Produce two reports using different lookups into a moderate MDDB-Cube)

    * 86 seconds (index lookup after indexed dataset loaded into memory)
    * 64 seconds (proc format)
    * 11 seconds (Persistent hash)

    The lookup consists of unique pairs

       String

       Question
       Florida Family Physicians in 1959    Answer

       DOC FL FAMLEE 1959                   3,567    * Total Florida Family Physicians in 1959
       DOC NY INTMED 1967                   5,663

    Here is how the hash is done

       1. load the hash table
       2. do the global lookups and place the results in an array
       3. pass the array to a datastep and proc print in a dosubl routine
       4. do another lookup and place the results in an array
       5. pass the array to a datastep and proc print in a dosubl routine

    Here is the code

    * PERSISTENT HASH;
    data _null_;
        retain idx 0;
        declare hash lookUp();
        rc=lookUp.defineKey('question');
        rc=lookUp.defineData('answer');
        rc=lookUp.defineDone();
        do until(eof1);
          set iaf.&pgm._hsh002 end=eof1;
          rc=lookUp.add();
        end;
        start=time();
        array que[12] $15 _temporary_;
        array ans[12] _temporary_;
         do rep=1 to 1000000;
          do job="DOC","DEN","NUR";
           do State="FL";
             do specialty="FAMLEE","INTMED";
               do year="2000","2005";
                 idx=idx+1;
                 call missing(answer);
                 question=cats(job,state,specialty,year);
                 que[idx]=question;
                 rc=lookUp.find();
                 ans[idx]=answer;
               end;
             end;
           end;
         end;
         idx=0;
        end;

        * FIRST REPORT;

        GetQue=put(addr(que[1]),16.);
        GetAns=put(addr(ans[1]),16.);
        call symputx('AdrAns',GetAns);
        call symputx('AdrQue',GetQue);
        _error_=dosubl(
           'data quean;
              do idx=1 to 12;
                 ans=peek (&adrans + (idx-1)*8,8 );
                 que=peekc(&adrque + (idx-1)*15,15);
                 output;
             end;
            run;
            proc print data=quean;run;quit;
        ');

        * SECOND REPORT;

        GetQue=put(addr(que[1]),16.);
        GetAns=put(addr(ans[1]),16.);
        call symputx('AdrAns',GetAns);
        call symputx('AdrQue',GetQue);

        array que2[8] $15 _temporary_;
        array ans2[8] _temporary_;
         do rep=1 to 1000000;
          do job="PHM","POD";
           do State="FL";
             do specialty="AAC300","AAC200";
               do year="1995","2000";
                 idx=idx+1;
                 call missing(answer);
                 question=cats(job,state,specialty,year);
                 que2[idx]=question;
                 rc=lookUp.find();
                 ans2[idx]=answer;
                 keep question answer;
                 output;
               end;
             end;
           end;
         end;
         idx=0;
        end;
        _error_=dosubl(
           'data quean2;
              do idx=1 to 8;
                 ans=peek (&adrans + (idx-1)*8,8 );
                 que=peekc(&adrque + (idx-1)*15,15);
                 output;
             end;
            run;
            proc print data=quean2;run;quit;
        ');
        lapse=time()-start;put lapse=;
        stop;
    run;


    Sampe output Report

    Obs    IDX    ANS          QUE

      1      1    146    DOCFLINTMED2000
      2      2    186    DOCFLINTMED2005
      3      3     20    DOCFLFAMLEE2000
      4      4     52    DOCFLFAMLEE2005
      5      5     63    DENFLINTMED2000
      6      6     47    DENFLINTMED2005
      7      7      6    DENFLFAMLEE2000
      8      8     13    DENFLFAMLEE2005
      9      9    291    NURFLINTMED2000
     10     10    470    NURFLINTMED2005
     11     11      3    NURFLFAMLEE2000
     12     12     30    NURFLFAMLEE2005

