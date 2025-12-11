### Uniswap V2 课程内容总结：现货价格与切线斜率的关系

#### 知识点要点：

1. **现货价格（Spot Price）定义**：

   - 现货价格是代币之间的即时交换比率，在 Uniswap V2 中也称为 **中间价格（Mid Price）**。

   - 公式：

     $text{Spot Price} = -\frac{y_0}{x_0}$

     其中：

     - y0y_0: AMM 中代币 YY 的储备量。
     - x0x_0: AMM 中代币 XX 的储备量。

2. **现货价格的计算**：

   - $ \text{Token X in terms of Token Y} = \frac{y_0}{x_0}。$
   - $ \text{Token Y in terms of Token X} = \frac{x_0}{y_0}。$
   - 示例：
     - XX: ETH, 储备量 x0=3,000x_0 = 3,000 ETH。
     - YY: DAI, 储备量 y0=6,000,000y_0 = 6,000,000 DAI。
     - 现货价格为： $6,000,0003,000=2,000 DAI/ETH\frac{6,000,000}{3,000} = 2,000 \, \text{DAI/ETH}$

3. **现货价格与切线斜率的关系**：

   - 橙线是恒定乘积曲线的切线，其斜率反映了现货价格：$ \text{Slope of Orange Line} = -\frac{y_0}{x_0}$
   - 负号的来源是代币交换中「放入」和「取出」的方向性。

4. **交易价格与现货价格的关系**：

   - 小额交易（

     Δx\Delta x

      非零）对应的交换比率称为 

     执行价格（Execution Price）

     ：

     $Execution Price=−ΔyΔx\text{Execution Price} = -\frac{\Delta y}{\Delta x}$

     - Δy\Delta y: 取出的代币 Y 数量。
     - Δx\Delta x: 放入的代币 X 数量。

   - 当 

     $Δx→0\Delta x \to 0$

      时：

     - 执行价格 $\frac{\Delta y}{\Delta x} 逼近现货价格 \frac{y_0}{x_0}。$

5. **公式推导**：

   - 从交换公式推导： $Δy=Δx⋅y0x0+Δx\Delta y = \Delta x \cdot \frac{y_0}{x_0 + \Delta x}$
   - 取比率 $ΔyΔx\frac{\Delta y}{\Delta x}： ΔyΔx=y0x0+Δx\frac{\Delta y}{\Delta x} = \frac{y_0}{x_0 + \Delta x}$
   - 当$ Δx→0\Delta x \to 0 时： ΔyΔx→y0x0\frac{\Delta y}{\Delta x} \to \frac{y_0}{x_0}$

6. **现货价格的近似计算**：

   - 小额交易可以用于近似现货价格：
     - $\Delta x 越小，\frac{\Delta y}{\Delta x} 越接近 \frac{y_0}{x_0}。$
   - 这一方法可以扩展到更复杂的 AMM，例如 Curve。

7. **总结**：

   - 现货价格是 AMM 中最重要的价格指标，由储备量的比率 y0x0\frac{y_0}{x_0} 表示。
   - 切线斜率与现货价格直接相关，小额交易是计算和验证现货价格的有效方法。
   - 理解现货价格的推导过程是深入掌握 AMM 工作原理的基础。





