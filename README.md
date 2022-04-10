# Dognition_Case_MySQLscript

Data Infrastructure Course Real Life Project

In this case, I analyzed the subscriber trends and classified characteristics, correlated subscription types, account numbers, and activity, and Identified reasons for Dognition’s retention challenges via data mining.

I helped Dognition (https://www.dognition.com) to examine their data, identify potential issues, and provide some general insights regarding their product and users. 

The Dognition team is aware that their database is somewhat “messy” but they are unclear about the extent of the problems.  

There are six tables in the database and the team has provided you with some information about the fields in each table. 

None of the six tables have a properly-defined primary key.  

Given all of this information, I have been assigned three main tasks.



## Table of Tasks
* [Broad Data Cleaning and Relational Schema](#Broad_Data_Cleaning_and_Relational_Schema)
* [Searching for Potential Issues](#Searching_for_Potential_Issues)
* [Summarize the Data](#Summarize_the_Data)

## Broad_Data_Cleaning_and_Relational_Schema

* For complete_tests:
After some trials we found that all the user_guid is NULL in this table, dog_guid could be selected with created_at to give a trial. During testing(using group by) whether those two could be qualified as the primary keys, we found there were still some rows unable to be identified uniquely(the result set is not blank), unless combining with test_name. Therefore, the combination of dog_guid, created_at and test_name would be the primary key.

* For dogs:
As the table is about dogs, it is preferable if dog_guid can be used as the ONLY primary key. It has then been verified.

* For exam_answers:
This result shows that the combination of script_detail_id, start_time, end_time, loop_number, and dog_guid can uniquely identify each row in the table.

* For reviews:
The intuition here for this table is that every user with their dog(s) should have a unique review for one particular test. So user_guid, dog_guid and test_name were used to give a trial as the combined primary key. However, some rows were unable to be identified uniquely(after using group by, the result set is not blank). And by combining with an extra column create_at, they were successfully identified.

* For site_activities:
This result shows that the combination of description, created_at, and user_guid fields can uniquely identify each row in the table.

* For users:
We can see that there're 13 records that have multiple values for one user_guid. What are they?
From the column "uid_cnt" and "utc_correction" it's obvious that the duplicate records are due to the N/A result in utc_correction. Next we'll drop those records so that the user_guid field can uniquely identify each row in the table.
This count is the same as COUNT(distinct(user_guid)) from cleaned_users_v0, which means we have successfully eliminated records that have multiple values for user_guid.

Then we can generate the relational schema for this database.

## Searching_for_Potential_Issues

————Summary about the potential issues in each table————<br>

For #complete_tests#:<br>
1.The user_guid is all NULL in the table, indicating that this information has not been successfully stored. <br>
There was data missing for some of the dog_guid as well. It is assumed that the dog_guid could be used to track the user_guid by joining with another table in the future analysis and hense the user_guid could be deleted in this table.<br>
2.Note for created_at and updated_at: as indicated by the client, those two could be the same if the information has never been updated.

For #dogs#:<br>
1.2974 records have a zero or negative weight, which doesn't make sense.<br>
2.26 records show that the dogs were born prior to 1990, which seems not quite possible, because it's almost impossible for dogs to live for more than 23 years.<br>
This kind of problem may be the mistake made from the dog owner. Therefore, Dognition should double check those information with the users again.<br>
3.There're 333 records in which update timestamp is earlier than create timestamp. That doesn't make sense. And this might be caused by mis-recorded of the system/website.

For #exam_answers#:<br>
When looking through the data, some of the end times were earlier than the start times for the record, which shouldn't be possible as you cannot end/submit a question before you start it. There were 49639 instances of this, or approximately 2% of the table, but it is still an issue to look into.

For #reviews#:<br>
1.updated_at and created_at has exactly the same values which means that one of these columns is unnecessary and can be easily eliminated.<br>
2.We can see '0' as rating given 10483 times which is not possible since the options do not have '0' as an option.If those '0' mean 'NULL', they need to change.

For #site_activities#:<br>
1.All membership_id and category_id were NULL value, while some user_guid or dog_guid were missing. Those two columns (membership_id and category_id) would be ignored and never used in future analysis. Those missing records inuser_guid or dog_guid would be excluded if necessary. <br>
2.Some dog_guid was recorded as ‘Membership’, which does not make sense. It is assumed that those erroneous records could be ignored when conducting analysis on this column if necessary.<br>
3.Though the script_id did not have to be unique, but by common sense, as it indicated the website page number, it should be continuous. However, the script_id here missed the value of ‘2’ and ‘10’. This is assumed that the users stayed in page 2 were counted into page 1 or page 3, as well as page 10 in the same way.

For #users#:<br>
First, the membership_id column is all integers from 1 to 18, which does not match the Data Dictionary description of the Unique ID.<br>
Second, the number of records for the updated_at and last_active_at columns does not match before and after the last update. This shows that there is inconsistency in these two columns and needs to be corrected. The last_active_at column of the user table has many missing values that need to be filled. Unless the act of signing in is not considered an activity, then this column is problematic.<br>
Third, according to the nature of UTC_correction, its record should be either positive, negative, or Null. However, some cells of this column is now logged as “#NA” in users table. We think the statistics here are questionable.<br>
Fourth, there were 5 NULL membership_type in users table, which are assumed to be excluded for further analysis.



As there were many NUll values in the table, the previous codes had a check with the number of imformative user_guid and dog_guid then compared them with the number of rows in the cleaned table. It was found that the user_guid is all NULL in the table, indicating that this information has not been successfully stored. There was data missing for some of the dog_guid as well. It is assumed that the dog_guid could be used to track the user_guid by joining with another table in the future analysis and hense the user_guid could be deleted in this table.

Note for created_at and updated_at: as indicated by the client, those two could be the same if the information has never been updated.

 • Examine cleaned_dogs table

We first start with a glance at the whole cleaned table.

After looking at the result set there're several things we need to check for potential problems:

1. Check the columns with binary datatype: gender, dog_fixed, dna_tested, exclude. Find out whether there's unreasonable value.
2. Examine the columns with common sense, for example, "birthday" should not be any year after 2015; "weight" and "total_tests_completed" should not be negative; "updated_at" should come after "created_at"...
3. Examine some inconsistencies according to the assessment.

Now we fist have a look at gender, dog_fixed, dna_tested, exclude column to check the binary datatype.

It seems that these columns are all well-recorded. Then we move on to other columns.

Ignoring the None result(which means there's no error in birthday or weight) we can see that 26 records report a birthday error, and around 3000 records that have a zero or negative weight. This kind of problem may be the mistake made from the dog owner. Therefore, Dognition should double check those information with the users again.

There're 333 records in which update timestamp is earlier than create timestamp. That doesn't make sense. And this might be caused by mis-recorded of the system/website.

Next we will check the potential inconsistencies in iti days/minutes and the number of total tests.

Perfect. The count is 0, then we can conclude that the table has made all the calculations right.

BUT there's one more thing...The datatype of some fields is not suitable. For example, "total_tests_completed" should be int and "mean_iti_days","mean_iti_minutes"...should be float/double. Dognition may need to change their datatypes so that analysts could use these fields properly.

 • Examine cleaned_exam_answers table

Noticed end time for some is earlier than start time - should not happen because you cannot end a question before you start it - run columns individually and confirmed with timediff.

COUNT WHERE TIMEDIFF < 0

This is a large number, so it is reasonable to say that this is a problem. to double check, calculate a percentage.

2% isn't a lot in the long run, but can still be problematic and should still be looked into.

 • Examine cleaned_reviews table

In the given data, updated_at and created_at has exactly the same values which means that one of these columns is unnecessary and can be easily eliminated.

According to the data disctionary, rating can only be given between 1 and 9. Another value 'None' appears when no rating has been given. In the provided data, we can see '0' as rating given 10483 times which is not possible since the options do not have '0' as an option.

 • Examine cleaned_site_activities table
 
First, the previous codes checked the NULL values in the ID columns of the cleaned site_activities table, it was found that all membership_id and category_id were NULL value, while some user_guid or dog_guid were missing. Those two columns (membership_id and category_id) would be ignored and never used in future analysis. Those missing records inuser_guid or dog_guid would be excluded if necessary. <br>

Besides, it was found that some dog_guid was recorded as 'Membership', which does not make sense. It is assumed that those erroneous records could be ignored when conducting analysis on this column if necessary.

Finally, when checking the script_id and script_detail_id. Though the script_id did not have to be unique, but by common sense, as it indicated the website page number, it should be continuous. However, the script_id here missed the value of '2' and '10'. This is assumed that the users stayed in page 2  were counted into page 1 or page 3, as well as page 10 in the same way.

 • Examine cleaned_users table
 
From the output we can initially find some potential issues. 

First, the membership_id column is always composed of individual digits, which does not match the Data Dictionary description of the Unique ID. If we make a comparison with the membership_type column, we can find that the data of both columns are the same. Therefore, we preliminatively think that all the membership_id data in this table is missing, and the statistician populates this column with the membership_type data. 

Second, we find that the data in the last_active_at column is problematic. We can see that there are a lot of Null cells in this column. And the definition of this column is, time-stamp of user's last activity in his/her Dognition account. By comparing the two columns sign_in_count and last_active_at, we can find that if the user never sign in dognition, the value of sign_in_count's cell is 0, and the value of last_active_at's cell is NULL. This is a very normal situation. Then, by comparing the updated_at column with the last_active_at column, we can see that the system automatically updates the statistics at 2015-01-28 20:51:49. If the user was last active before this time, then its updated_at cell will be this time. But if the user is active after this time, the time of their updated_at will be the same as the time that their last_active_at shows. However, we found that many users had a sign_in_count record greater than or equal to 1, but their last_active_at record was null. We assume that the data in these two columns are not consistent. There may be some problems in last_active_at column. 

Third, according to the nature of UTC_correction, its record should be either positive, negative, or Null. However, some cells of this column is now logged as "#NA" in users table. We think the statistics here are questionable.

Next, we use SQL query to verify first two potential issues.

First, let's examine the membership_id column. Let's first look at the maximum and minimum values of this column and the number of membership_type values that it does not equal and equal.

Well, the maximum value in the membership_id column is 18, and the minimum value is 1, and all are integers. This is enough to say that it cannot be used as a Unique ID because there is too much same data. But it could be grouping users in some way. Let's continue to look at the membership_id and membership_type records.

Oops, it seems that the number of different rows of membership_id and membership_type is quite large, indicating that our previous guess that the records of these two columns are duplicate is wrong. Let's take a look at the result of GROUP BY for membership_id and the number of groups.

So it seems that the record of membership_id ranges from 0 to 18 integers, and its distribution is not very obvious. Such records cannot be used as Unique ID. Next, let's examine the second problem we found. Let's first verify that all rows in the last_Active_at column that are not null with updated_at. If there are inconsistencies here, that would also be a potential problem.

Now we can see that the number of records for the updated_at and last_active_at columns does not match before and after the last update. This shows that there is inconsistency in these two columns and needs to be corrected.

The last_active_at column of the user table has many missing values that need to be filled. Unless the act of signing in is not considered an activity, then this column is problematic.


## Summarize_the_Data

### a) Use the data to analyze "user sign-ups".

The results indicated that there were 5 NULL membership_type in users table, which are assumed to be excluded for further analysis.

The result showed no membership_type having the value bigger than 5 or smaller than 1, which means there is no erroneous records in this column.

Those two results indicated that the creat_at column ranges from 2012-12-07 to 2015-10-12 with no 'unbelivable' value.

Up to here, the columns for user sign-ups analysis have been checked.

Irregularities and possible explainations<br>
#1 From the results, it could be found that there was almost no sign_up_count until 2013 Feb. It was because the company was founded at the end of Year 2012 and the website had not been available until 2013 March (reference:Google).

#2 Percentage_5 had been 0 until 2015 May because membership type 5 was a recent test.

#3 From 2013-7 to 2013-10, 2014-10, and from 2015-5 to 2015-9, the sign_up_count growed rapidly. By searching on Google for those specific timings, it was found that the articles about dognition was published more frequently, than the rest time. Our hypothesis here to explain this was that the company implemented some marketing techniques(e.g. those articles we found) and advertisement, to attract more new users.

#4 Looking in a larger scope, very few customers were retained after those sign-up booms, and were no longer converted to be a subscriber. Our hypothesis is that those customers were not satisfied with the products. More evidence need to be provided to justify this hypothesis.

### b) Use results from the “user sign-ups” analysis to investigate more 

By analyzing the trends of time of joining (new sign-up users) to the total tests_completed they took, some interesting facts were found out. 

For type 1 (Dognition Assessment of initial 20 games), for most time this kind of subscribers took the most tests and had a big boom in 2014 Oct. It was assumed to be a simultaneous boom in new sign-up users but there was not. Therefore, it could be explained by a promotion on this kind of particular subscription, during that time.<br>

For type 2 (Annual), for this kind of subscribes, those who signed up early took the most tests. Then it ranked the second for most time and also had a big boom like the type 1.<br>

For type 3 (Monthly), they took very few tests, which was consistent with the fewer number in subscription.<br>

For type 4 (Free), for users signed up in the second half year in 2013 or 2015, they conducted somehow more tests but generally much fewer than type 1. By checking with the number of new sign-up users along the timeline, free users had two booms from 2013-7 to 2013-10, 2014-10, and from 2015-5 to 2015-9. This might indicated that free users were attracted to the website and signed up, but for some reason (found the assessment too tough), they did not get the free 4 games done.

### c) Investigate hypothesis

#### 1. Is the assessment too complicated so that many users get to a certain point, become frustrated, and quit?

To test this conjecture, let's first look at user activity on the webpage. Here, we extract the description and COUNT(*) columns from the cleaned_site_activities table and rank them in descending order of COUNT(*) to see what happens most often when the users use the webpage.

It looks like a lot of users have made it to some specific numbers of page, and we happened to find the activity_type and test_name in the cleaned_site_activities table. We can then filter out the activities in the table and analyze this kind of activity together with their activity type, test_name, and the number of times they occur.

As we can see from the above results, all of the above user activity occurred on the test page because it contained very specific game names. Tens of thousands of users have reached page 103, 106, 109, 127, 130, 133, 153, 156, and 159. But after that, fewer and fewer people get to the later pages. That means fewer and fewer people are completing later tests. Most users may encounter difficulties at tests like Foot Pointing. And there is a very small percentage of the people who finished all the tests. We might reasonably guess that it was because Dognition Assessment was too complicated that people reached a point where they started to feel frustrated and gave up the test early. This can be one of the reasons for low retention rate of Dognition.

#### 2. Is there any issue with the Dognition website ?

One hypothesis is that there may be issues with the Dognition website, where certain webpages are prone to issues, resulting in user confusion. From the results above, it could be found that value '2' and '10' were missing in the script_id. As script_id represented the type of Dognition activity users are engaged in, this might mean some webpages are not available to get access. By lacking such kind of activities on certain webpages, users could be confused easily. It needs to be confirmed with the website developer whether those activities were removed in purpose or not. To sum up, this hypothesis could not be rejected because truly there was some issues within the webpages.

#### 3. The assessment itself is simply better suited to certain “types” of owners and/or dogs ?

We examine the total number of dogs, sum of tests completed by the dogs, the average number of tests completed and the average days of mean of intervals between two tests in separate groups of gender, breed_type, breed_group and dimension.

It turns out that there's not so much difference with different groups in terms of average mean_iti_days. The differences are mostly within 1 day.<br>
For example, when we look at breed_type, pure breed dogs tend to have higer average of mean_iti_days and less tests completed. Can we say that pure breed dogs are not suited for this assessment? But if we look at breed_group,herding dogs have both higher average of mean_iti_days and more tests completed. How to explain that?<br>
Therefore, we suggest that it can not easily jump into the conclusion that the assessment itself is better suited to certain “types” of dogs.

