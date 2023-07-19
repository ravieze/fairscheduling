# fairscheduling
On a kafka topic assume a FAT producer dumps 1M msgs, thin producer dumps 10 msgs in next second. Now kafka consumer has to drain 1M messages before he consumes the 10 msgs from the thin producer. We try to solve this issue.
