# 最精确有限元理论——广义模态单元法及体单元应用源代码示例
   本文介绍广义模态单元法未来发展的若干可能。在当前主流开源的潮流下，我期望能够以开源和讨论的形式，使该理论通过学者们的共同努力最终在工程数值仿真的各个领域生根发芽，并为社会的进步发挥其力量。我曾经在论文中描述该方法为最精确的有限元方法，但我的博士导师孙秦教授建议我写论文要谦虚，不要自己说自己好，但我也后来反而觉得这样描述是一种不严谨的学术态度。当然有时也会稍稍遗憾自己未能进一步发展这一套理论。导师一直建议我出国读个博士后再留校，但出于两个方面的原因导致我没有选择这条道路，一是家庭原因，二是我已经提出了世界上最好的有限元理论，我不期望在学术界评论一个人的学术水平是通过是否出国或者发表多少篇论文这种唯一标准来衡量，我觉得做学术不同于闯社会，学术就是一个美丽纯洁的艺术品，重要的是如何去打造这个艺术品，如何在思想的海洋遨游，而不是在体制下苟且的生活。

## 1． 为何为世间最精确的有限元方法？
   有限元方法是通过线性方程组（对于非线性问题则通过局部线性化）将物理世界的两个属性进行联结（如力学的变形和力，热学的能量和温度），每个单元可以描述与单元阶数相等数目的物理属性间的关系。该理论为广义有限元理论，对于单元中的任意一个属性间关系，其能够在所有已有方法中选择最精确的那一组描述，因此其是最精确的有限元方法。
    如果出现了一个新的方法，比之间模态单元法构造的单元要精确，那么把这种新的方法使用模态单元法进行融合，就会得到精度更高或者相等的单元，因此模态单元法永远是精度最高的有限元方法。

## 2． 论文中的单元是不是精度最高的？
   不是，论文中的单元只是融合了已知的两到三种方法，因此其精度局限于这两到三种方法的精度。

## 3． 方法适应领域：
   任意需要通过求解微分方程组进行数值模拟领域（如结构力学，流体力学，热学，电磁学，声学，核物理等）。
    
## 4． 各领域的基本实现
   对于各个领域，我们通过微分方程组求解的方式，对于一些特定约束是可以得到一些理论解的。但现有有限元尴尬的地方是得到的单元许多时候反而无法得到理论解。广义有限元方法则可以很好解决这个问题。
    广义有限元方法由于可以包含任意方法，因此只要你能够给出理论解，那么得到的单元也就可以给你精确得出理论解。
各个领域的基本实现可以采取三个基本步骤：
    第一步是找到该领域的“应力权值函数”，从结构力学看，其是非常好找的，为最基本的线性插值函数。
    第二步是给出想要模拟的理论解，建议从零阶、一阶一直到高阶选取。
    第三步检查单元的奇异性(许多模态之间是线性相关的)，对于相关的模态进行更换。可以选择假设应变法，参考非物理模态。

## 5．各领域的高阶实现
   高阶实现相比于基本实现，要更为灵活，且计算精度可以达到更高。高阶实现是通过优化理论，由计算机自行求解得到想要领域的高精度单元。这与我们通常创建一种高精度单元需要院士级别大牛来完成是完成不同的，即使对于一个小白，也可以创建出顶级的单元。高阶实现包含如下三步：
    第一步，建立大量的算例，对于无法使用理论解求解的算例则可以使用现有有限元方法通过超密网格进行求解。同时对于这些算例，给出判别准则SCORE，如有些算例不允许出现误差，可以设置权值高一些（如弯曲和拉伸等基本算例），对于复杂算例则设置权值低一些。
    第二步，建立单元的位移场和应变场（注意两者是独立的），尽管从物理的角度讲应变场是位移场的导数，但这种强相关反而会使得单元对于一些复杂变形精度降低（对于复杂变形，单元边界也会形状复杂，如果相邻两个单元的边界变形后相差太大反而会导致精度降低，因此通过独立的方法会自动选择相邻边界变形误差与精度之间的最佳组合）。
    第三步，对于位移场和应变场，起都是通过若干多项式或者方程的线性组合，通过求解第一步中的SCORE对于各个多项式或者方程的导数，寻找最佳的线性组合。至此，通过迭代既可以得到最优的单元构造方法。位移场选择优先级如下所示，从低阶至高阶选择，选择的多则可以提高精度
                                1
                         x      y      z
                xx    xy     xz     yy     yz     zz  xyz
           xxx   xxy  xxz  xyy  xzz  yyy   yyz  yzz  zzz   xxyz  xyyz  xyzz …
u=a1 +b1x +c1y +d1z + e1xx +f1xy +g1xz+h1yy+i1yz+j1zz +k1xyz … (共n个变量)
v=a2 +b2x +c2y +d2z + e2xx + …. (共n个变量)
w=…
    对于八节点体单元24个模态，由于其只有三组非物理模态是明确需要替代的，每个模态又包含u、v、w，因此需要对3*n*3个变量进行优化。
对于二十节点体单元60个模态，为了减小计算量，对于前面若干个变形可以直接使用理论解，比如刚体位移，拉伸，弯曲，剪切，扭转等，对于其它变形，也可以使用现有二十节点体单元的位移场函数作为初始解，并进行进一步优化迭代，得到最终单元。
这里我们假设需要对US-MEM8S(或S-MEM8S)的非物理模态进行优化，优化起始点可以设置a1至j1为0，k1为1，同时v和w都为0.

## 该理论在力学中的一些新型应用场景。
1.	使用进阶方法构造二十节点体单元。可以肯定的是，现有的二十节点体单元计算精度还处于原始社会，但根据三角形到四边形单元计算精度带来的提升看，八节点面单元（二十节点体单元）可能会带来一次新的革命。
2.	使用进阶方法构造二十节点体壳单元，其与体单元构造方法唯一不同的地方是使用范围不同，体壳单元使用在上下表面都自由场景，因此我们在构造算例时对于上下表面自由和非自由的需要使用两套单元，然后由计算机完成两套单元的生成。目前体壳单元在厚度方向只有两个节点，因此其只能描述厚度方向常应变变形，对于厚度方向的应力变化则无能为力。使用进阶方法可以构造这类体壳单元，且随着计算机精度的提高，已有有限元方法在厚度方向不能模拟变化应力的缺陷也必然是需要改变的。
3.	使用高阶方法对已有八节点和八节点体壳单进行优化，可以使用自然坐标或者笛卡尔坐标。体壳单元的位移场必须包含三阶场。
4.	任意类型过渡单元。
