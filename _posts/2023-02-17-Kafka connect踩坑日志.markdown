kafka connect生产环境使用时发现的问题列表：
1. jdbc大表抽取需要开启游标模式useCursorFetch=true；
2. jdbc connector bulk模式会因为rebalance导致restart task；
