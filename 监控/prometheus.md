https://www.cnblogs.com/chanshuyi/p/01_head_first_of_prometheus.html#%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86



https://cloud.tencent.com/document/product/14	16/56033





[P95、P99.9百分位数值——服务器响应时间的重要衡量指标](https://www.cnblogs.com/hunternet/p/14354983.html#1220964152)

avg：平均数

P50：即中位数，100个请求，按照响应时间大小排序，第50个的请求耗时，即为P50的值，如果P50的值为200ms，意味着一般的用户接口响应时间在200ms以上

P95：响应耗时从小到大排列，顺序处于95%位置的值即为P95值

P99.9：就是99.9%用户耗时作为指标，1000个用户里面，999个用户的耗时上限，通过测量与优化该值，就可以保证绝大多数用户的体验。至于P99.99值，优化成本过高，而且服务响应由于网络波动、系统抖动等不能解决之情况，因此大多数时候都不考虑该指标。

[如何计算百分位数值](https://www.cnblogs.com/hunternet/p/14354983.html#1220964152)