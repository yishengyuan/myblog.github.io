### Uniswap V2 时间加权平均价格（TWAP）课程大纲

1. **关于TWAP的常见误解**
   - 有人认为，如果知道了Token X的TWAP，就可以通过对Token X的TWAP取倒数来得到Token Y的TWAP。这是错误的，接下来将解释为什么这种推理不成立。
2. **现货价格与时间加权平均价格的区别**
   - 在Uniswap V2中，Token X和Token Y之间的现货价格可以表示为Y/X，反过来就是X/Y，这代表了Token Y和Token X之间的现货价格。
   - 如果知道了Token X的现货价格P（P = Y/X），那么取其倒数（1/P）可以得到Token Y的现货价格。但这并不意味着Token Y的TWAP等于Token X的TWAP的倒数。
3. **时间加权平均价格（TWAP）公式**
   - Token X的TWAP可以通过以下公式计算：
     - TWAP(X) = 累积价格变化 / 时间区间
   - Token Y的TWAP公式几乎与此相同，唯一的区别在于需要使用Token Y与Token X的现货价格倒数（1/P）。
   - 由于公式的不同结构，不能简单地将Token X的TWAP取倒数来得到Token Y的TWAP。
4. **倒数关系不能直接应用于TWAP**
   - 虽然现货价格的倒数可以用来转换Token X和Token Y之间的价格关系，但这并不意味着Token Y的TWAP等于Token X的TWAP的倒数。
   - 可以通过数学推导验证，取倒数的做法不会得出正确的结果。
5. **总结**
   - 在实际应用中，虽然现货价格的倒数能够帮助我们从一个代币的价格推算另一个代币的价格，但对于TWAP的计算，这一推理并不成立。