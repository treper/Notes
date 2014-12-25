####ParallelFPGrowthMapper对transaction分组

map

实际feature itemId到fMap(frequency map) id(int)的映射，因为f-list是按频次倒序排的，所以itemId从高频到低频，高频的itemId对应的int id越小，低频的越大

对每一个Transaction,都加载G-List的分组HashMap
HashMap 
0 1 2 3 4 ->0 
5 6 7 8 9 ->1

以下举例对3个transaction进行分组：

0 2 4:
4 Hash-> 0 -> delete 0 group -> out 0 [0 2 4]
0 Hash-> null

0 2 4 6:
6 Hash-> 1 -> delete 1 group -> out 1 [0 2 4 6]
4 Hash-> 0 -> delete 0 group -> out 0 [0 2 4]
2 Hash-> null
0 Hash-> null

1 3 5 7 9: 
9 Hash-> 1 -> delete 1 group -> out 1 [1 3 5 7 9]
7 Hash-> null
5 Hash-> null
3 Hash-> 0 -> -> delete 0 group -> out 0 [1 3]
1 Hash-> null 




