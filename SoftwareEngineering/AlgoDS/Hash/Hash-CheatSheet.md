[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Hash CheatSheet

# SuRF

SuRF 是一种查询效率高、压缩比高的字典树（Trie）数据结构，其功能十分简单，提供过滤器的功能。众所周知，BloomFilter 被广泛用于单点过滤——用于判断单个 key 是否存在于集合中。而 SuRF 的不仅拥有单点过滤功能，还能支持范围查询这个大杀器，即其能根据给出的 key 范围来判断其是否在集合中。SuRF 是一种 Trie 树结构，Trie 树的结构使其能支持范围查询，但传统 Trie 树所占空间太大，效率低，在数据库里实际运用很少。SuRF 的突破创新点是其拥有极限小的空间，其 trie 树的每个节点平均只需要占 10bit 空间，而且同时保持了很高的查询性能，构造性能。SuRF 团队将 SuRF 运用到了目前十分受欢迎的 NoSQL 系统 RocksDB 之中，在范围查询之中提升了 1.5 倍~5 倍的效率。
