---
layout: post
title: Data comparison between csv files
---

#### Data comparison between csv files
Here’s the problem - you have two csv files that came from two different accounting systems. Ideally they would both have the same data but sometimes, they differ. And when they differ, thats a problem that the company’s accounting department cares greatly about and needs to fix. So once someone figures out that we have a QA team that could possibly handle some of this, this sort of thing becomes a regular task you get. What do you do? You script it the very first time. And keep adding to the script every time you see a variation of the request. That way you have a great tool, you’ve sharpened your programming skills and everyone is happy :) Read on.

I’ve used pandas which may be overkill for this kind of project but then you never know. This is the starting point, who knows how complicated these requests will get one day? So pandas it is.

The algorithm is - read the two files, load them into pandas dataframes, do an inner join on the key column (say a field named ‘reference’) and that gives you the rows that are present in both files. You save that file, thats one of the common asks. You may later need to add verification around whether this field matches in both files etc for rows that exist in both rows. Once you have the inner join results, its easy to extend that to whatever else you need. Write these results into a csv file.

Next, do an outer join with a little indicator field which tells you if each individual row is in the left dataframe only, or the right dataframe only or in both. Awesome. Write this result into a csv file as well. For now you could just stop here, provide both these csv files you just generated to the accounting team and ask them to do the rest in excel. Or you can do the rest in python. I’ll stop here though. 

Here’s the actual code,

```
import pandas as pd
import numpy as np
import sys

def usage():
   print ‘USAGE: python data_compare.py recurlyfilename.csv intuitfilename.csv’

if len(sys.argv) == 3:
   rec_file = sys.argv[1]
   int_file = sys.argv[2]
else:
   print 'Expected 2 arguments. Please try again.’
   usage()
   exit(-1)

print rec_file
print int_file

rec_df = pd.read_csv(rec_file)
int_df = pd.read_csv(int_file,
                    names=[ 'reference’,'Date’,'CardholderName’,'Card’,'CardNo’,'Credit/Debit’,'Type’,'BatchID’, 'WithholdingStatus’,'Status’,'Amount’,'Fee’])

innerjoined_df = pd.merge(int_df, rec_df, on='reference’, how='inner’)
innerjoined_df.to_csv('found_in_both.csv’)

outerjoined_df = pd.merge(int_df, rec_df, on='reference’, how='outer’, indicator=True)

outerjoined_df.to_csv('difference.csv’)
```
