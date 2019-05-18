第10章 在策略控制近似方法
============================

在本章中，我们回到控制问题，现在使用动作-价值函数 :math:`\hat{q}(s,a,\mathbf{w})\approx q_{*}(s, a)` 的参数近似，
其中 :math:`\mathbf{w} \in \mathbb{R}^{d}` 是有限维权重向量。我们继续只关注在策略情况，将离策略方法留给第11章。
本章以半梯度Sarsa算法为特征，将半梯度TD(0)（最后一章）自然延伸到动作价值和策略控制。
在回合案例中，扩展是直截了当的，但在持续的情况下，我们必须向后退几步并重新审视我们如何使用折扣来定义最优策略。
令人惊讶的是，一旦我们有真正的函数近似，我们就必须放弃折扣并转而使用新的“差异”价值函数来控制问题的新“平均回报”表达式。

首先从回合案例开始，我们将上一章中提出的函数近似思想从状态值扩展到动作价值。
然后我们扩展它们以控制遵循策略GPI的一般模式，使用 :math:`\varepsilon` 贪婪进行动作选择。
我们在陡坡汽车问题上显示n步线性Sarsa的结果。然后我们转向持续的情况并对于具有不同价值的平均奖励案例重复开发这些想法。


10.1 回合半梯度控制
-----------------------

将第9章的半梯度预测方法扩展到动作价值是很简单的。在这种情况下，它是近似动作价值函数 :math:`\hat{q} \approx q_{\pi}`，
其表示为具有权重向量 :math:`\mathbf{w}` 的参数化函数形式。
而在我们考虑 :math:`S_{t} \mapsto U_{t}` 形式的随机训练示例之前，
现在我们考虑 :math:`S_{t}, A_{t} \mapsto U_{t}` 形式的例子。
更新目标 :math:`U_{t}` 可以是 :math:`q_{\pi}\left(S_{t}, A_{t}\right)` 的任何近似值，包括通常的备份值，
例如完整的蒙特卡罗回报（:math:`G_{t}`）或任何n步Sarsa回报（7.4）。动作价值预测的一般梯度下降更新是

.. math::

    \mathbf{w}_{t+1} \doteq \mathbf{w}_{t}+\alpha\left[U_{t}-\hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t}\right)\right] \nabla \hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t}\right)
    \tag{10.1}

例如，一步Sarsa方法的更新是

.. math::

    \mathbf{w}_{t+1} \doteq \mathbf{w}_{t}+\alpha\left[R_{t+1}+\gamma \hat{q}\left(S_{t+1}, A_{t+1}, \mathbf{w}_{t}\right)-\hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t}\right)\right] \nabla \hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t}\right)
    \tag{10.2}

我们将这种方法称为 *回合半梯度一步Sarsa*。对于常量策略，此方法以与TD(0)相同的方式收敛，
具有相同类型的误差边界（9.14）。

为了形成控制方法，我们需要将这些动作价值预测方法与策略改进和行动选择技术相结合。
适用于连续动作或来自大型离散集的动作的适当技术是正在进行的研究主题，但尚未明确解决。
另一方面，如果动作集是离散的并且不是太大，那么我们可以使用前面章节中已经开发的技术。
也就是说，对于当前状态 :math:`S_t` 中可用的每个可能动作，
我们可以计算 :math:`\hat{q}\left(S_{t}, a, \mathbf{w}_{t}\right)`，
然后找到贪婪动作 :math:`A_t^*=\arg\max _{a} \hat{q}(S_t,a,\mathbf{w}_{t-1})`。
然后通过将估计策略更改为贪婪策略（例如 :math:`\varepsilon` -贪婪策略）的软近似来完成策略改进
（在本章中处理的在策略案例中）。根据相同的策略选择动作。完整算法的伪代码于下框中给出。

.. admonition:: 回合半梯度Sarsa估计 :math:`\hat{q} \approx q_*`
    :class: important

    输入：可微分的动作价值函数参数化 :math:`\hat{q} : \mathcal{S} \times \mathcal{A} \times \mathbb{R}^{d} \rightarrow \mathbb{R}`

    算法参数：步长 :math:`\alpha>0`，小 :math:`\varepsilon>0`

    任意初始化价值函数权重 :math:`\mathbf{w} \in \mathbb{R}^{d}` （比如 :math:`\mathbf{w}=\mathbf{0}`）

    对每个回合循环：

        :math:`S, A \leftarrow` 初始回合状态和动作（比如 :math:`\varepsilon` -贪婪）

        对回合每一步循环：

            采取动作 :math:`A`，观察 :math:`R, S^{\prime}`

            如果 :math:`S^{\prime}` 不是终点：

                :math:`\mathbf{w} \leftarrow \mathbf{w}+\alpha[R-\hat{q}(S, A, \mathbf{w})] \nabla \hat{q}(S, A, \mathbf{w})`

                进入下一个回合

            选择 :math:`A^{\prime}` 作为 :math:`\hat{q}\left(S^{\prime}, \cdot, \mathbf{w}\right)` 的函数（比如 :math:`\varepsilon` -贪婪）

            :math:`\mathbf{w} \leftarrow \mathbf{w}+\alpha\left[R+\gamma \hat{q}\left(S^{\prime}, A^{\prime}, \mathbf{w}\right)-\hat{q}(S, A, \mathbf{w})\right] \nabla \hat{q}(S, A, \mathbf{w})`

            :math:`S \leftarrow S^{\prime}`

            :math:`A \leftarrow A^{\prime}`

**例10.1：陡坡汽车任务** 考虑如图10.1左上图所示，在陡峭的山路上驾驶动力不足的汽车的任务。
困难在于重力比汽车的发动机强，即使在全油门时，汽车也无法陡坡加速。
唯一的解决方案是首先远离目标并向左边相反的斜坡上移动。
然后，通过应用全油门，汽车可以建立足够的惯性，以便将其带到陡坡上，即使它在整个过程中减速。
这是一个连续控制任务的简单示例，在某种意义上，事物必须变得更糟（离目标更远）才能变得更好。
除非得到人类设计师的明确帮助，否则许多控制方法对于此类任务都有很大的困难。

.. figure:: images/figure-10.1.png

    **图10.1：** 陡坡汽车任务（左上图）和一次运行中学习的成本函数 :math:`-\max _{a} \hat{q}(s, a, \mathbf{w})`。

这个问题的奖励在所有时步都是 :math:`-1`，直到汽车越过山顶的目标位置，结束了这一回合。
有三种可能的动作：全油门前进（:math:`+1`），全油门倒车（:math:`-1`）和零油门（:math:`0`）。
汽车根据简化的物理学运动。 它的位置 :math:`x_t` 和速度 :math:`\dot{x}_{t}`，由更新

.. math::

    \begin{array}{l}
    {x_{t+1} \doteq \text{bound}\left[x_{t}+\dot{x}_{t+1}\right]} \\
    {\dot{x}_{t+1} \doteq \text{bound}\left[\dot{x}_{t}+0.001 A_{t}-0.0025 \cos \left(3 x_{t}\right)\right]}
    \end{array}

其中 :math:`\text{bound}` 操作强制 :math:`-1.2 \leq x_{t+1} \leq 0.5`
和 :math:`-0.07 \leq \dot{x}_{t+1} \leq 0.07`。
另外，当 :math:`\dot{x}_{t+1}` 到达左边界时，:math:`\dot{x}_{t+1}` 被重置为零。
当它到达右边界时，达到了目标并且回合终止了。每回合从随机位置 :math:`x_{t} \in[-0.6,-0.4)` 和零速度开始。
为了将两个连续状态变量转换为二进制特征，我们使用了网格平铺，如图9.9所示。
我们使用8个铺片，每个铺片覆盖每个维度中有界距离的1/8，并且如第9.5.4节所述的非对称偏移 [1]_。
然后通过铺片编码创建的特征向量 :math:`\mathbf{x}(s, a)` 与线性组合，用于近似动作价值函数的参数向量：

.. math::

    \hat{q}(s, a, \mathbf{w}) \doteq \mathbf{w}^{\top} \mathbf{x}(s, a)=\sum_{i=1}^{d} w_{i} \cdot x_{i}(s, a)
    \tag{10.3}

对于每对状态 :math:`s` 和动作 :math:`a`。

图10.1显示了在学习使用这种形式的函数近似来解决此任务时通常会发生的情况 [2]_。
显示的是在单次运行中学习的价值函数（*成本（cost- to-go）* 函数）的负数。
初始动作值均为零，这是乐观的（在此任务中所有真值均为负值），即使探测参数 :math:`\varepsilon` 为0，也会导致进行大量探索。
这可以在中间的顶部面板中看到，图中标有“Step 428”。此时甚至没有一个回合完成，但是汽车在山谷中来回摆动，沿着状态空间的圆形轨迹。
所有经常访问的状态都比未开发状态更糟糕，因为实际的奖励比（不切实际的）预期更糟糕。
这会不断推动个体远离任何地方，探索新的状态，直到找到解决方案。

图10.2显示了此问题的半梯度Sarsa的几条学习曲线，具有不同的步长。

.. figure:: images/figure-10.2.png

    **图10.2：** 具有铺片编码函数近似和 :math:`\varepsilon` -贪婪动作选择的半梯度Sarsa方法的陡坡汽车任务学习曲线。


10.2 半梯度n步Sarsa
------------------------

我们可以通过在半梯度Sarsa更新方程（10.1）中使用n步回报作为更新目标来获得回合半梯度Sarsa的n步版本。
n步回报立即从其表格形式（7.4）推广到函数近似形式：

.. math::

    G_{t : t+n} \doteq R_{t+1}+\gamma R_{t+2}+\cdots+\gamma^{n-1} R_{t+n}+\gamma^{n} \hat{q}\left(S_{t+n}, A_{t+n}, \mathbf{w}_{t+n-1}\right), \quad t+n<T
    \tag{10.4}

其中 :math:`G_{t:t+n}=G_{t}` 如果 :math:`t+n \geq T`，像往常一样。 n步更新方程是

.. math::

    \mathbf{w}_{t+n} \doteq \mathbf{w}_{t+n-1}+\alpha\left[G_{t : t+n}-\hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t+n-1}\right)\right] \nabla \hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t+n-1}\right), \quad 0 \leq t<T
    \tag{10.5}

完整的伪代码在下面的框中给出：

.. admonition:: 回合半梯度n步Sarsa估计 :math:`\hat{q} \approx q_*` 或 :math:`q_\pi`
    :class: important

    输入：可微分的动作价值函数参数化 :math:`\hat{q} : \mathcal{S} \times \mathcal{A} \times \mathbb{R}^{d} \rightarrow \mathbb{R}`

    输入：一个策略 :math:`\pi` （如果估计 :math:`q_\pi`）

    算法参数：步长 :math:`\alpha>0`，小 :math:`\varepsilon>0`，一个正整数 :math:`n`

    任意初始化价值函数权重 :math:`\mathbf{w} \in \mathbb{R}^{d}` （比如 :math:`\mathbf{w}=\mathbf{0}`）

    所有存储和访问操作（:math:`S_t`，:math:`A_t` 和 :math:`R_t`）都可以使用它们的索引 :math:`mod n+1`

    对每个回合循环：

        初始化并存储 :math:`S_0 \ne` 终点

        选择并存储动作 :math:`A_0 \sim \pi(\cdot | S_0)` 或者关于 :math:`\hat{q}(S_{0}, \cdot, \mathbf{w})` :math:`\varepsilon` -贪婪

        :math:`S, A \leftarrow` 初始回合状态和动作（比如 :math:`\varepsilon` -贪婪）

        :math:`T \leftarrow \infty`

        对 :math:`t=0,1,2,cdots` 循环：

            如果 :math:`t<T`，则：

                采取动作 :math:`A_t`

                观察和存储下一个奖励为 :math:`R_{t+1}` 和下一个状态为 :math:`S_{t+1}`

                如果 :math:`S_{t+1}` 是终点，则：

                    :math:`T=t+1`

                否则：

                    选择并存储动作 :math:`A_{t+1} \sim \pi(\cdot | S_{t+1})` 或者关于 :math:`\hat{q}(S_{t+1}, \cdot, \mathbf{w})` :math:`\varepsilon` -贪婪

            :math:`\tau \leftarrow t-n+1` （:math:`\tau` 是其估算值正在更新的时间）

            如果 :math:`\tau \geq 0`：

                :math:`G \leftarrow \sum_{i=\tau+1}^{\min (\tau+n, T)} \gamma^{i-\tau-1} R_{i}`

                如果 :math:`\tau+n<T` 则 :math:`G\leftarrow G+\gamma^n \hat{q}\left(S_{\tau+n}, A_{\tau+n}, \mathbf{w}\right) \quad` （:math:`G_{\tau:\tau+n}`）

                :math:`\mathbf{w} \doteq \mathbf{w}+\alpha\left[G-\hat{q}\left(S_{t}, A_{t}, \mathbf{w}\right)\right] \nabla \hat{q}\left(S_{\tau}, A_{\tau}, \mathbf{w}\right)`

        直到 :math:`\tau=T-1`

正如我们之前看到的，如果使用中等级别的自举，性能最佳，对应于大于1的n。
图10.3显示了该算法陡坡汽车任务中在 :math:`n=8` 时比在 :math:`n=\infty` 处更快地学习并获得更好的渐近性能。
图10.4显示了参数 :math:`\alpha` 和 :math:`n` 对该任务学习率的影响的更详细研究结果。

.. figure:: images/figure-10.3.png

    **图10.3：** 陡坡汽车任务中一步与八步半梯度Sarsa的表现。使用了良好的步长：
    :math:`n=1` 时 :math:`\alpha=0.5/8` 且 :math:`n=8` 时 :math:`\alpha=0.3/8`。

.. figure:: images/figure-10.4.png

    **图10.4：** :math:`\alpha` 和 :math:`n` 对陡坡汽车任务的n步半梯度Sarsa和铺片编码函数近似的早期性能的影响。
    像往常一样，中等的自举（:math:`n=4`）表现最佳。这些结果用于选定的 :math:`\alpha` 值，以对数刻度，然后通过直线连接。
    对于 :math:`n=16`，标准误差范围从 :math:`n=1` 时0.5（小于线宽）到大约 :math:`n=16` 时4，因此主要影响都是统计上显着的。

*练习10.1* 在本章中，我们没有明确考虑或给出任何蒙特卡罗方法的伪代码。他们会是什么样的？
为什么不为它们提供伪代码是合理的？他们将如何在陡坡汽车任务上表现？

*练习10.2* 给出关于控制的半梯度一步 *预期* Sarsa提供伪代码。

*练习10.3* 为什么图10.4中显示的结果在大 :math:`n` 处比在小 :math:`n` 处具有更高的标准误差？


10.3 平均奖励：持续任务的新问题设置
-------------------------------------

我们现在引入第三个经典设置──与回合和折扣设置一起，用于制定马尔可夫决策问题（MDP）中的目标。
与折扣设置一样，*平均奖励* 设置适用于持续存在的问题，即个体与环境之间的交互在没有终止或启动状态的情况下持续进行的问题。
然而，与那种情况不同的是，没有折扣──个体对延迟奖励的关注与对即时奖励的关注一样多。
平均奖励设置是经典动态规划理论中常用的主要设置之一，在强化学习中较少见。
正如我们在下一节中讨论的那样，折扣设置在功能近似方面存在问题，因此需要平均奖励设置来替换它。

在平均奖励设置中，策略 :math:`\pi` 的质量被定义为平均奖励率，或简称为 *平均奖励*，
同时遵循该策略，我们将其表示为 :math:`r(\pi)`：

.. math::

    \begin{aligned}
    r(\pi) & \doteq \lim _{h \rightarrow \infty} \frac{1}{h} \sum_{t=1}^{h} \mathbb{E}\left[R_{t} | S_{0}, A_{0 : t-1} \sim \pi\right] && \text{(10.6)}\\
    &=\lim _{t \rightarrow \infty} \mathbb{E}\left[R_{t} | S_{0}, A_{0 : t-1} \sim \pi\right] && \text{(10.7)}\\
    &=\sum_{s} \mu_{\pi}(s) \sum_{a} \pi(a | s) \sum_{s^{\prime}, r} p\left(s^{\prime}, r | s, a\right) r
    \end{aligned}

期望取决于初始状态 :math:`S_0`，以及按照 :math:`\pi` 的
后续动作 :math:`A_{0}, A_{1}, \dots, A_{t-1}`，:math:`\mu_\pi` 是稳态分布，
:math:`\mu_{\pi}(s) \doteq \lim_{t \rightarrow \infty} Pr\left\{S_{t}=s | A_{0:t-1} \sim \pi\right\}`，
假设存在于任何 :math:`\pi` 并且独立于 :math:`S_0`。关于MDP的这种假设被称为 *遍历性（ergodicity）*。
这意味着MDP启动或个体做出的任何早期决策只会产生暂时影响；从长远来看，处于一个状态的期望仅取决于策略和MDP转移概率。
遍历性足以保证上述等式中存在极限。

在未折现的持续情况中，可以在不同类型的最优性之间进行微妙的区分。
然而，对于大多数实际目的而言，简单地根据每个时间步的平均奖励来订购策略可能就足够了，换句话说，根据他们的 :math:`r(\pi)`。
如（10.7）所示，该数量基本上是 :math:`\pi` 下的平均回报。
特别是，我们认为所有达到 :math:`r(\pi)` 最大值的策略都是最优的。

请注意，稳态分布是特殊分布，在该分布下，如果根据 :math:`\pi` 选择动作，则保留在同一分布中。也就是说，为此

.. math::

    \sum_{s} \mu_{\pi}(s) \sum_{a} \pi(a | s) p\left(s^{\prime} | s, a\right)=\mu_{\pi}\left(s^{\prime}\right)
    \tag{10.8}

在平均奖励设置中，回报是根据奖励与平均奖励之间的差来定义的：

.. math::

    G_{t} \doteq R_{t+1}-r(\pi)+R_{t+2}-r(\pi)+R_{t+3}-r(\pi)+\cdots
    \tag{10.9}

这称为 *差分* 回报，相应的值函数称为 *差分* 价值函数。
它们以相同的方式定义，我们将一如既往地使用相同的符号：
:math:`v_{\pi}(s) \doteq \mathbb{E}_{\pi}\left[G_{t} | S_{t}=s\right]` 和
:math:`q_{\pi}(s, a) \doteq \mathbb{E}_{\pi}\left[G_t|S_{t}=s, A_{t}=a\right]`
（类似于 :math:`v_*` 和 :math:`q_*`）。 差分价值函数也有Bellman方程，与我们之前看到的略有不同。
我们只是删除所有 :math:`\gamma s` 并通过奖励和真实平均奖励之间的差替换所有奖励

.. math::

    \begin{array}{l}
    {v_{\pi}(s)=\sum_{a} \pi(a | s) \sum_{r, s^{\prime}} p\left(s^{\prime}, r | s, a\right)\left[r-r(\pi)+v_{\pi}\left(s^{\prime}\right)\right]} \\
    {q_{\pi}(s, a)=\sum_{r, s^{\prime}} p\left(s^{\prime}, r | s, a\right)\left[r-r(\pi)+\sum_{a^{\prime}} \pi\left(a^{\prime} | s^{\prime}\right) q_{\pi}\left(s^{\prime}, a^{\prime}\right)\right]} \\
    {v_{*}(s)=\max _{a} \sum_{r, s^{\prime}} p\left(s^{\prime}, r | s, a\right)\left[r-\max _{\pi} r(\pi)+v_{*}\left(s^{\prime}\right)\right], \text { 以及 }} \\
    {q_{*}(s, a)=\sum_{r, s^{\prime}} p\left(s^{\prime}, r | s, a\right)\left[r-\max _{\pi} r(\pi)+\max _{a^{\prime}} q_{*}\left(s^{\prime}, a^{\prime}\right)\right]}
    \end{array}

（参见（3.14），练习3.17，（3.19）和（3.20））。

还存在两种TD误差的差分形式：

.. math::

    \delta_{t} \doteq R_{t+1}-\overline{R}_{t}+\hat{v}\left(S_{t+1}, \mathbf{w}_{t}\right)-\hat{v}\left(S_{t}, \mathbf{w}_{t}\right)
    \tag{10.10}

以及

.. math::

    \delta_{t} \doteq R_{t+1}-\overline{R}_{t}+\hat{q}\left(S_{t+1}, A_{t+1}, \mathbf{w}_{t}\right)-\hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t}\right)
    \tag{10.11}

其中 :math:`\overline{R}_{t}` 是平均奖励 :math:`r(\pi)` 在时间 :math:`t` 的估计值。
通过这些替代定义，我们的大多数算法和许多理论结果都可以在不改变的情况下进行平均奖励设置。

例如，半梯度Sarsa的平均奖励版本定义如（10.2），除了TD误差版本的不同。也就是说

.. math::

    \mathbf{w}_{t+1} \doteq \mathbf{w}_{t}+\alpha \delta_{t} \nabla \hat{q}\left(S_{t}, A_{t}, \mathbf{w}_{t}\right)
    \tag{10.12}

由（10.11）给出的 :math:`t`。完整算法的伪代码在下框中给出。

.. admonition:: 差分半梯度Sarsa估计 :math:`\hat{q} \approx q_*`
    :class: important

    输入：可微分的动作价值函数参数化 :math:`\hat{q} : \mathcal{S} \times \mathcal{A} \times \mathbb{R}^{d} \rightarrow \mathbb{R}`

    算法参数：步长 :math:`\alpha,\beta>0`

    任意初始化价值函数权重 :math:`\mathbf{w} \in \mathbb{R}^{d}` （比如 :math:`\mathbf{w}=\mathbf{0}`）

    任意初始化平均奖励估计 :math:`\overline{R} \in \mathbb{R}` （比如 :math:`\overline{R}=0`）

    初始化动作 :math:`S`，状态 :math:`A`

    对每个回合循环：

        采取动作 :math:`A`，观察 :math:`R, S^{\prime}`

        选择 :math:`A^{\prime}` 作为 :math:`\hat{q}\left(S^{\prime}, \cdot, \mathbf{w}\right)` 的函数（比如 :math:`\varepsilon` -贪婪）

        :math:`\delta \leftarrow R-\overline{R}+\hat{q}\left(S^{\prime}, A^{\prime}, \mathbf{w}\right)-\hat{q}(S, A, \mathbf{w})`

        :math:`\overline{R} \leftarrow \overline{R}+\beta \delta`

        :math:`\overline{R} \leftarrow \overline{R}+\beta \delta`

        :math:`\mathbf{w} \leftarrow \mathbf{w}+\alpha \delta \nabla \hat{q}(S, A, \mathbf{w})`

        :math:`S \leftarrow S^{\prime}`

        :math:`A \leftarrow A^{\prime}`

*练习10.4* 为半梯度Q-learning的差分版本提供伪代码。

*练习10.5* 需要哪些方程式（除了10.10）来指定TD(0)的差分版本？

*练习10.6* 假设有一个MDP在任何策略下产生确定性的奖励序列 :math:`+1,0,+1,0,+1,0, \dots` 一直继续下去。
从技术上讲，这是不允许的，因为它违反了遍历性；没有静态极限分布 :math:`\mu_\pi`，并且不存在极限（10.7）。
然而，平均奖励（10.6）是明确的；它是什么？现在考虑这个MDP中的两个状态。
从 **A** 开始，奖励序列完全如上所述，从 :math:`+1` 开始，
而从 **B** 开始，奖励序列以 :math:`0` 开始，然后继续 :math:`+1,0,+1,0, \ldots`。
对于这种情况，差分回报（10.9）没有很好地定义，因为不存在限制。为了修复这个问题，可以交替地将状态值定义为

.. math::

    v_{\pi}(s) \doteq \lim _{\gamma \rightarrow 1} \lim _{h \rightarrow \infty} \sum_{t=0}^{h} \gamma^{t}\left(\mathbb{E}_{\pi}\left[R_{t+1} | S_{0}=s\right]-r(\pi)\right)
    \tag{10.13}

根据这个定义，状态 **A** 和 **B** 的值是多少？

*练习10.7* 考虑马尔可夫奖励过程，该过程由三个状态 **A**，**B** 和 **C** 组成，
状态转换确定性地围绕环。到达 **A** 时会收到 :math:`+1` 的奖励，否则奖励为 :math:`0`。
使用（10.13）的三个状态的差分值是多少？

*练习10.8* 本节的方框中的伪代码使用 :math:`\delta_t` 作为误差而
不是简单的 :math:`R_{t+1}-\overline{R}_{t}` 更新 :math:`\overline{R}_{t}`。
这两个误差都有效，但使用 :math:`\delta_t` 更好。要了解原因，请考虑练习10.7中三个状态的环形MRP。
平均奖励的估计应倾向于其真实值 :math:`\frac{1}{3}`。 假设它已经存在并被卡在那里。
:math:`R_{t+1}-\overline{R}_{t}` 误差的序列是什么？
:math:`\delta_{t}` 误差的序列是什么（使用（10.10））？
如果允许估计价值因误差而改变，哪个误差序列会产生更稳定的平均回报估计值？ 为什么？

**例10.2：访问控制排队任务** 这是一个涉及对一组10台服务器的访问控制的决策任务。
有四个不同优先级的客户到达一个队列。如果允许访问服务器，则客户向服务器支付1,2,4或8的奖励，
具体取决于他们的优先级，优先级较高的客户支付更多。
在每个时步中，队列头部的客户被接受（分配给其中一个服务器）或被拒绝（从队列中移除，奖励为零）。
在任何一种情况下，在下一个时步中，将考虑队列中的下一个客户。队列永远不会清空，队列中客户的优先级随机分布。
当然，如果没有免费服务器，则无法提供服务；在这种情况下，客户总是被拒绝。
每个繁忙的服务器在每个时步都变为空闲，概率 :math:`p=0.06`。
虽然我们刚刚将它们描述为明确，但让我们假设到达和离开的统计数据是未知的。
任务是根据优先级和免费服务器的数量来决定是接受还是拒绝下一个客户，以便最大化长期奖励而不折扣。

在这个例子中，我们考虑这个问题的表格解决方案。虽然状态之间没有泛化，
但我们仍然可以在通用函数近似设置中考虑它，因为此设置泛化了表格设置。
因此，我们对每对状态（空闲服务器的数量和队列头部的客户的优先级）和操作（接受或拒绝）进行了不同的动作-价值估计。
图10.5显示了差分半梯度Sarsa求解的解，
参数 :math:`\alpha=0.01`，:math:`\beta=0.01`，:math:`\varepsilon=0.1`。
初始动作价值和 :math:`\overline{R}` 为零。

.. figure:: images/figure-10.5.png

    图10.5：差分半梯度一步法Sarsa在200万步后的访问控制排队任务中找到的策略和价值函数。
    图表右侧的下降可能是由于数据不完善；许多这些状态从未经历过。:math:`\overline{R}` 学习的价值约为2.31。


10.4 弃用折扣设置
--------------------


10.5 微分半梯度n步Sarsa
-----------------------------


10.6 总结
------------


书目和历史评论
---------------


.. [1]
    特别地，我们使用了铺片编码软件，可从http://incompleteideas.net/tiles/tiles3.html获得，
    其中 :math:`\text{int}=\operatorname{IHT}(4096)` 和
    :math:`\text{tiles}(\text{iht}, 8,[8*x/(0.5+1.2), 8*xdot/(0.07+0.07)], A)`
    得到状态 :math:`(x，xdot)` 和动作 :math:`A` 的特征向量中的索引。

.. [2]
    这个数据实际上来自“半梯度Sarsa(:math:`\lambda`)”算法，直到第12章我们都不会遇到，但是半梯度Sarsa的行为类似。
