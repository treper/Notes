#####Storm和Spark Streaming的比较

|Properites|Storm|Spark-Streaming|
|:--------:|:---:|:-------------:|
|Processing Model|one at a time|batches up events within a short time|
|Latency|sub-second|serveral seconds|
|Fault Tolerance|each record will be processed at least once, but allows duplicates to appear during recovery from a fault|each mini-batch will be processed exactly once, even if a fault such as a node failure occurs. [Actually, Storm's Trident library also provides exactly once processing. But, it relies on transactions to update state, which is slower and often has to be implemented by the user.]|