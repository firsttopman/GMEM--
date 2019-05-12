# 最精确有限元理论（同时为首个可自动化创建新单元的有限元理论）——广义模态单元法及体单元matlab源码示例
` ` ` ` ` ` 本文在Computational Mechanics上连载论文[1][2]的基础上，进一步讨论广义模态单元法未来的发展和应用场景。广义模态单元法为最精确的有限元理论，我曾经计划在论文中描述这一特征，但导师孙秦教授(非常Nice的教授)觉得这样写不能够体现学者的谦逊，所以后来就没有加进去。但在两篇相关学术论文及博士学位论文的审稿过程中，我发现其实并没有人能够完全读懂我论文的思想，也就是说并没有人能够真正理解这一套理论潜在的价值。<br>
` ` ` ` ` ` **在当前开源的潮流下，我期望能够以开源的方式，使该理论通过不同领域学者的共同努力，最终在工程数值仿真各领域生根发芽，发挥出其应有的价值。为了方便广义模态单元法的理解、验证以及未来的进一步演进，本repository同时给出[1]中八节点体单元matlab源码示例。**
## 1． 广义模态单元法为何为最精确的有限元方法？
` ` ` ` ` ` 通俗的解释，广义模态单元法为一种广义的有限元方法，它可以汲取任意现有单元技术的优点，因此其为最精确的有限元方法。<br>
` ` ` ` ` ` 有限元方法是通过线性方程组将物理世界的两个属性进行联结，如结构力学中的结构变形和载荷，热力学中的能量和温度，且每个单元可以描述与单元阶数相等数目的物理属性间关系。广义模态单元法对于单元中的任意一个属性间关系（论文中称为模态），能够在已有方法中选择最精确的描述方法进行描述，从而是最精确的有限元方法。当然在实施过程中一个人是无法把所有现有的有限元方法实现一遍，但只要你有更精确的，广义模态单元法就能把该方法用进来。<br>

## 2． 论文中的单元是不是精度最高的？
` ` ` ` ` ` 不是，论文中的单元只是融合Analytical method、Assumed displacement method、龙驭球院士和Nastran的两种壳单元，因此其精度局限于这几种方法的精度范围。

## 3． 方法适应领域
` ` ` ` ` ` **任意需要通过求解微分方程组进行数值模拟领域（如结构力学，流体力学，热力学，电磁学，声学，核物理等）。**
    
## 4． 各领域新单元的基本实现
` ` ` ` ` ` 对于任意工程应用领域，我们可以通过物理微分方程组求解的方式，对于一些特定约束条件可以得到理论解。但**现有有限元方法尴尬的地方是，得到的单元在许多时候反而无法得到理论解，这也就出现了许多时候我们需要使用理论解去校验一个单元精度的情况。**广义模态单元法中的Analytical method则可以很轻松解决这个问题，想要它模拟什么理论解，它就可以模拟想要的理论解。同时，由于许多变形无法或者较难得到理论解，我们还需要进一步借助[1]中的Assumed displacement method来完成特殊模态的构造，这样才能够实现任意类型单元的构造。<br>
` ` ` ` ` ` 各个领域的基本实现可以采取三个基本步骤：<br>
` ` ` ` ` ` 第一步是找到该领域的“应力权值函数”，并且需要证明该函数满足相关领域的基本约束，结构力学中为满足力和力矩平衡约束，参考[1]中证明，流体力学中则可能为质量、动量和能量守恒方程。依据结构力学的经验，“应力权值函数”通常为最基本的线性插值函数，形式比较简单。<br>
` ` ` ` ` ` 第二步是给出想要模拟的理论解，建议从零阶、一阶一直到高阶选取，参考[1]。<br>
` ` ` ` ` ` 第三步对于较难得到理论解的模态，可以使用Assumed displacement method进行构造，见[1]中的非物理模态。
` ` ` ` ` ` 第四步检查单元的奇异性(许多模态之间是线性相关的)，通常第三步和第四步是一起反复进行的。我们可能可以找到许多理论解，比如可以将更多方向的弯曲引入八节点体单元，但会发现其与已有模态是线性相关的，这也是体单元出现非物理模态的原因。
## 5．各领域新单元的自动化实现
` ` ` ` ` ` 这里要讲的内容，是[1]和[2]中未覆盖的场景。**现有的有限元技术都是需要顶级教授甚至院士级别的大牛才能够完成新的创造，这里我们则要讲一个硕士生甚至本科生，如何在计算机庞大计算能力的帮助下，实现各领域顶级单元的自动化构造。** 该自动化方法是通过优化理论，由计算机自行求解得到想要领域的高精度单元。实现步骤如下：<br>
` ` ` ` ` ` 第一步，建立大量的用例，如现有有限元论文中经常用到的标准测试用例，对于无法使用理论解求解的算例则可以使用有限元方法通过超密网格进行数值求解。对于每个用例，给出误差的判断方法，作为优化过程中的目标函数。如有些算例不允许出现误差，可以设置权值高一些（如弯曲和拉伸等基本算例）。<br>
` ` ` ` ` ` 第二步，建立单元的位移场和应变场（注意两者是独立的），尽管从物理的角度讲应变场是位移场的导数，但这种强相关反而会使得单元对于一些复杂变形精度降低（对于复杂变形，如果相邻两个单元变形后边界相差太大反而会导致精度降低，因此通过位移场和应变场独立的方法，可以选择最佳的组合，参考[1]中的非物理模态）。<br>
` ` ` ` ` ` 第三步，对于位移场，我们首先通过理论解构造，通过若干多项式或者方程的线性组合，通过求解第一步中的定义的目标函数，对于各个多项式或者方程求解导数，寻找最佳梯度方向，并通过迭代找到最优的单元构造方法。位移场选择优先级如下所示，通常从低阶至高阶选择<br>
` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` 1<br>
` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` x` ` ` ` ` ` ` ` ` ` ` ` y` ` ` ` ` ` ` ` ` ` ` ` z<br>
` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` ` xx` ` ` ` ` ` xy` ` ` ` ` ` xz` ` ` ` ` ` yy` ` ` ` ` ` yz` ` ` ` ` ` zz` ` ` ` ` ` xyz<br>
` ` ` `  xxx` ` ` `  xxy` ` ` `   xxz` ` ` `  xyy` ` ` `  xzz` ` ` `  yyy` ` ` `  yyz` ` ` `  yzz` ` ` `  zzz` ` ` `  xxyz` ` ` `  xyyz` ` ` `  xyzz<br>
……<br>
<br>
u=a1 +b1x +c1y +d1z + e1xx +f1xy +g1xz+h1yy+i1yz+j1zz +k1xyz … (共n个变量)<br>
v=a2 +b2x +c2y +d2z + e2xx + …. (共n个变量)<br>
w=…<br>
<br>
` ` ` ` ` ` 第四步，对于应变场，如果相应位移场为理论解，则对于位移场进行求导即可得到，如果位移场不为理论解，则同样可以采用若干多项式或者方程的线性组合，但需要保证应变场所得的应力场满足应力平衡方程，参考[1]中式8。<br>
` ` ` ` ` ` 这里举例二十节点体单元自动化的构造。对于二十节点体单元其包含60个模态，为了减小计算量，对于前面若干个变形可以直接使用理论解，比如刚体位移，拉伸，弯曲，剪切，扭转，同时剪力梁的基本变形也是需要包含的。对于其它变形模态，则可以构造三阶甚至四阶位移场，初始时可以使用现有二十节点体单元的位移场函数和应变场函数作为初始解，并基于此进行进一步优化迭代，得到最终包含三阶甚至四阶单元的表达式。<br>
` ` ` ` ` ` 更为复杂的，比如体壳单元，我们可以使用球坐标中的理论解，转换至笛卡尔坐标即可。


## 该理论在结构力学中的一些新型应用场景
1.	使用自动化方法构造八节点面单元（二十节点体单元）。可以肯定的是，**现有的八节点面单元（二十节点体单元）计算精度还处于原始社会**，其甚至还不能精确描述剪力梁的变形。根据三角形到四边形单元计算精度带来的提升看，八节点面单元（二十节点体单元）可能会带来一次新的革命。<br>
2.	使用自动化方法构造二十节点体壳单元，其与体单元构造方法唯一不同的地方是使用范围不同，体壳单元使用在上下表面都为自由表面，因此我们在构造算例时对于上下表面自由和非自由的需要分别使用体壳和体单元，然后由计算机完成两套单元的生成。目前体壳单元在厚度方向只有两个节点，因此其只能描述厚度方向常应变变形，对于厚度方向的应力变化则无能为力。使用进阶方法可以构造这类体壳单元，且随着计算机精度的提高，已有有限元方法在厚度方向不能模拟变化应力的缺陷也必然是需要改变的。<br>
3.	使用自动化方法对[1]中八节点的非物理模态进行优化，同时可以使用自然坐标（[1]中使用的是笛卡尔坐标），但个人感觉使S-MEM8S中将笛卡尔坐标系替换为自然坐标系会得到与Nastran完全一致的单元。<br>
4.  与其它单元的组合补充，参考[2]。<br><br>
 ` ` ` ` ` ` **广义模态单元法为非对称有限元理论，因此严谨的说，广义模态单元法为最精确的非对称有限元理论，但通过[1]、[2]中的对称化以及上边所述的自动化方法，我们相信所所得的对称单元也是最精确的单元技术之一，因为其是通过最优化理论得出来的。**<br>
` ` ` ` ` `由于工作原因，未能进一步发展该理论。导师曾建议我出国读个博士后再留校，但我觉得评论一个人的学术造诣是通过是否出国来衡量是非常不公平的。如果只是为了出国而出国，那是很无聊的。对于我来讲，在哪里不重要，重要的是能够做一个思想者，挑战一种不一样的人生，因此最终我并没有选择这条道路。
## 期待该理论能够在其它领域中有新的贡献，后续有时间补充英文版。

## 参考文献
[1] He P Q, Sun Q, Liang K. Generalized modal element method: part-I—theory and its application to eight-node asymmetric and symmetric solid elements in linear analysis. Computational Mechanics.<br>
[2] He, P.Q., Sun, Q. & Liang, K. Generalized modal element method: part II—application to eight-node asymmetric and symmetric solid-shell elements in linear analysis. Computational Mechanics. 
