报错:No dictionary found/dict.....invalid cube state...

解决:
是某个segment有问题,底层hbase表已经不存在了,但是元数据没有被删除,这边是手动disable这个cube,这个问题不常见,应该是DP的问题
一般的cube构建的时候会产生元数据不同步的情况,把构建和查询分开来,避免这种不一致问题,同时也解决了构建时查询慢的问题
