---
layout:     post
title:      "Production Models with Linear Constraints"
subtitle:   "简单的生产模型"
date:       2020-07-01 14:00:00
author:     "Meng"
header-img:  "img/20200701-production.jpg"
catalog: true
tags:
    - python
    - pyomo
    - 线性规划

---



> [pyomo-cookbook](https://nbviewer.jupyter.org/github/jckantor/ND-Pyomo-Cookbook/blob/master/notebooks/02.01-Production-Models-with-Linear-Constraints.ipynb#Example:-Production-plan-for-a-single-product-plant) 案例积累 
>
> Production plan for a single product plant



### 问题描述

假设您正在考虑建立一家生产产品X的企业。您确定每周有40个X产品需求，每个产品的价格为270美元。每个产品的生产需要100美元的原材料，1小时的人工A和2小时的人工B。您拥有无限量的原材料，但是每周只有80个小时的人工A，每小时需要花费50美元，而每周只有100个小时的人工B，每小时需要花费40美元。

另外，您的营销部门也制定了Y产品的计划。该产品的售价为每只210美元，他们希望您能够出售自己能做的所有产品。它的制造成本也更低，仅需90美元的原材料，一小时的A人工，一小时的B人工。

考虑在同一工厂中生产这两种产品的可能性。营销部门向我们保证，产品Y不会影响产品X的销售，请问最大利润是多少？

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggbgbrm0uzj31ds0iwgp9.jpg)

### 建模

1. 决策变量:

   1）x

   2）y

2. 目标函数：最大化  (270-100-50-80) * x + (210-90-50-40) * y  -> 40x+30y              

3. 约束：

   1）Demand: x <= 40

   2）Labor A: 1\*x+1\*y<80

   3）Labor B: 2\*x+1\*y<100

   


### Pyomo实现

```python
from pyomo.environ import *
from matplotlib.pyplot import *
matplotlib.use('TkAgg')
from numpy import *

# 1. Create a model
model = ConcreteModel()

# 2. Instantiate the model
# 2.1 declare decision variables
model.product = Var([1,2], within=NonNegativeIntegers)

# 2.2 declare objctives
model.profit = Objective(expr = 40*model.product[1]+ 30*model.product[2],sense = maximize)

# 2.3 declare constraints
conCoeff = {1:[1,0],2:[1,1],3:[2,1]}
conBound = {1:40, 2:80, 3:100}

model.con = ConstraintList()
for keys in conCoeff:
    model.con.add(conCoeff[keys][0]*model.product[1]+ conCoeff[keys][1]*model.product[2]<= conBound[keys])

# 3. Apply solver
opt = SolverFactory('cbc')
results = opt.solve(model)

# 4. Display results
print('Profit = ', model.profit(), 'per week')
print('ProductA = ', model.product[1](), 'per week')
print('ProductB = ', model.product[2](), 'per week')

# results.write()
# where stores time?
print('Time uesd',results.solver[0]['time'])

```



结果： 

```python
Profit =  2600.0 per week
ProductA =  20.0 per week
ProductB =  60.0 per week
Time uesd 0.022930145263671875
```

