# fair-scheduling
On a kafka topic assume a FAT producer dumps 1M msgs, thin producer dumps 10 msgs in next second. Now kafka consumer has to drain 1M messages before he consumes the 10 msgs from the thin producer. We try to solve this issue.


# Working

Lets assume we have 3 producers
- p1 --> produced 30 msgs at 9.00 am
- p2 --> produced 1Million msgs at 8.59 am
- p3 --> produced 20 msgs at 9.01 am

<img width="1255" alt="image" src="https://github.com/ravieze/fairscheduling/assets/5900517/ae2c7636-4fce-4e4d-b74b-273ea355197d">

## (i) initialization
1. (step 14) check target high watermark and max offset from target kafka topic. If the diffarence between them is > 100 then go to next step

## (ii) Writing the data 
1. (step 2) From the source kafka topic we drain N1 msgs (5Million say) it into the local store (Assuem file store/ rockdb)
2. (step 3) Write them into disk. Have folders for every producer (p1, p2, p3). Each folder has pages. Each page has R-rows (say 10 rows)  in them.

## (iii) reading data and fill target kafka topic
1. (step 11, 12, 13) From the source data (disk folders and files) we will read 1 page per producer (p1, p2, p3) and delete the page from the disk.
2. (step 14) we concatinate the rows from the pages and dump them into the target kafka queue.
3. -- Given that we take 1page from every producer folder, when data is dumped into the target kafka topic we get R rows(10 rows) from each producer page. Hence even if P2 duped 1M messages, in the taret kafka topic we see only 10 msgs from p2, 10 msgs from p1, p3. This solves the "noisy neighbor" issue by fairly scheduling between the producers. 
4. sleep 10 seconds and go to (i)

meaning first we read all green page of all producers first, followed by red page, followed by yellow pages. Concatinate the rows from each of these pages and dump the result into the target kafka topic. So this way you will see that a fat produer/ noisy producer will not affect the thin producer.

## (iv) backup and restore
1. every 5min we sync the data to s3.
2. every time the application startsup we first take data from s3 into the local folder. 

--> in the above if local SSD disk is replaced with rockdb. things get more fast and simple.

# inspirations
Netflix timestone

Any questions, please let me know by raising tickets. I have a working implementation to the above. But wanted to know how others are handling this issue of noisy producers. I have seen Amazon.in implementation on youtube and they say they use 3 kafka topics. But, in my view this doesnt really solve the issue of noisy neighbors. 
