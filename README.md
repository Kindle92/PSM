**随机对照试验（RCT）可以保证各组研究对象的基线特征（例如年龄、性别、疾病严重程度、伴随疾病等能够影响疾病预后的因素）得以一致。** 

**然而，在真实世界的前瞻性研究中，治疗方案的选择往往受到这些基线特征的影响，使得很多足以影响结局的因素在组间的分布并不均衡，产生一定的偏倚，最终部分低估或高估某种治疗方案的作用。回顾性的研究更是这样。** 

**举个栗子，如图，不知道是什么正经或不正经的理由，处理组的年轻人就是多（80%），最终观测的死亡率偏小，如果不加分层，还以为是治疗影响的结局。** 

![](https://raw.githubusercontent.com/QuanWangCN/PSM/master/Fig.0.png) 

**常用的平衡偏倚统计学方法包括分层(Stratified)、回归（Regression）、倾向性分析(Propensity)等。分层相当常用，最大的局限是层数不能太多、样本不能太少；回归最为常用，局限是收集的因素需尽可能全面、样本不能太少。** 

**今天介绍的是倾向性分析里最常用的倾向性评分匹配（Propensity Score Matching），操作起来分两步：第一步计算每个样本的倾向性评分，第二步按评分匹配对照组和实验组的样本。** 

**具体而言，倾向性评分是指在一定协变量（潜在偏倚）条件下，一个观察对象接受某种暴露/处理因素的可能性。用logistic回归模型估计倾向性评分，结果也易于理解。倾向性评分越接近于1，说明患者接受某种暴露/处理因素的可能性更高，越接近于0，说明患者不接受任何暴露/处理因素的可能性更大。解释成人话，评分相同，意味着个体同质（至少在分组选择上），用来组间比较比较好。** 

以下是用R进行模拟数据产生和倾向性得分匹配。做两个测试表df.patients（病人群体，主要是女性）和df.population（对照群体，主要是男性），而女性的distress评分会偏高，若不把性别偏倚排除掉，则会以为两个群体的distress评分有显著差异，会犯I类错误。 

\```r 

\#这段代码用于制表 

library(wakefield) 

df.patients <- r_data_frame(n = 100,age(x = 30:78,name = 'Age'), sex(x = c("Male", "Female"),prob = c(0.70, 0.30),name = "Sex")) 

df.patients$Sample <- as.factor('Patients') 

summary(df.patients) 

set.seed(1234) 

df.population <- r_data_frame(n = 500,age(x = 18:80,name = 'Age'), sex(x = c("Male", "Female"), prob = c(0.50, 0.50),name = "Sex")) 

df.population$Sample <- as.factor('Population') 

summary(df.population) 

mydata <- rbind(df.patients, df.population) 

mydata$Group <- as.logical(mydata$Sample == 'Patients') 

mydata$Distress <- ifelse(mydata$Sex == 'Male', 

age(nrow(mydata), x = 0:42, name = 'Distress'), 

age(nrow(mydata), x = 30:42, name = 'Distress')) 

\``` 

\```r 

\#这段代码用于检验两个群体的distress评分差异，最终显示有差异。实际上是假阳性。 

pacman::p_load(tableone) 

table1 <- CreateTableOne(vars = c('Age', 'Sex', 'Distress'), 

data = mydata, 

factorVars = 'Sex', 

strata = 'Sample') 

table1 <- print(table1, 

printToggle = FALSE, 

noSpaces = TRUE) 

library(knitr) 

kable(table1[,1:3], 

align = 'c', 

caption = 'Table 1: Comparison of unmatched samples') 

\``` 

![](https://raw.githubusercontent.com/QuanWangCN/PSM/master/Fig.1.png) 

\```r 

\#这段代码用于排除偏倚，也就是所谓PSM，就是只拿评分相同那部分来比较组间差异。df.match就是此处用来比较的200个个体。 

set.seed(1234) 

match.it <- MatchIt::matchit(Group ~ Age + Sex, 

data = mydata, 

method="nearest", 

ratio=1) 

a <- summary(match.it) 

kable(a$nn, digits = 2, align = 'c', 

caption = 'Table 2: Sample sizes') 

kable(a$sum.matched[c(1,2,4)], digits = 2, align = 'c', 

caption = 'Table 3: Summary of balance for matched data') 

plot(match.it, type = 'jitter', interactive = FALSE) 

df.match <- MatchIt::match.data(match.it)[1:ncol(mydata)] 

\``` 

![](https://raw.githubusercontent.com/QuanWangCN/PSM/master/Fig.2.png) 

![](https://raw.githubusercontent.com/QuanWangCN/PSM/master/DistributionOfPS.png) 

![](https://raw.githubusercontent.com/QuanWangCN/PSM/master/Fig.3.png) 