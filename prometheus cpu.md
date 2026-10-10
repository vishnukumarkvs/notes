prometheus cpu
==============

https://github.com/prometheus/prometheus/issues/16331


https://www.youtube.com/watch?v=IWrd-pSojqg


Check if compaction is wroking? with 12hr retention, it shouldnt, becvausw 10% limit

Moving to 1hr block instead of 2 hours -> slightly less cpu, query CPU increases, disk after flush increases

WAL size is equal to min block duration, WAL has uncompressed data

suppose I have an app which runs queries every 2 minutes, It sums data on last 1 hour. Now, if I migrate from 2h to 1 hr blocks, will it have an impact?


