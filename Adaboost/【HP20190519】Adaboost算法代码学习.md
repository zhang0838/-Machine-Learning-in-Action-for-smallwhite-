

```python
import numpy as np
import pandas as pd
```


```python
# 通过一个函数来加载特征矩阵和类别标签
def loadSimpleData():
    print ("-----加载数据的特征矩阵和类别标签------")
    datMat = np.matrix([[1,2.1],[2,1.1],[1.3,1],[1,1],[2,1]])
    classLabels = [1,1,-1,-1,1]
    print("展示特征矩阵")
    print(datMat)
    print("类别标签")
    print(classLabels)
    return datMat, classLabels
```


```python
datMat,classLabels=loadSimpleData()
```

    -----加载数据的特征矩阵和类别标签------
    展示特征矩阵
    [[1.  2.1]
     [2.  1.1]
     [1.3 1. ]
     [1.  1. ]
     [2.  1. ]]
    类别标签
    [1, 1, -1, -1, 1]
    


```python
# 定义单层决策树的阈值过滤函数
# 接着是树的分类函数。这个函数在下面的循环里要用到
# 作用很简单，就是比对每一列的特征值和目标函数，返回比对的结果。四个参数分别是（输入矩阵，第几列，阈值，lt小于或gt大于）
def stumpClassify(dataMatrix,dimen,threshVal,threshIneq):
    #对数据集每一列的各个特征进行阈值过滤，这里是构建一个以数据特征集的行数相等的元素全部为1的向量
    retArray=np.ones((np.shape(dataMatrix)[0],1))
    #阈值的模式，将小于某一阈值的特征归类为-1,下面的逻辑判断将要对每一个元素值与阈值进行比较，如果小于等于这个值，那么就更新这个元素为-1
    if threshIneq=='lt':
        retArray[dataMatrix[:,dimen]<=threshVal]=-1.0
    #将大于某一阈值的特征归类为-1
    else:
        retArray[dataMatrix[:,dimen]>threshVal]=-1.0
    return retArray
```


```python
# 定义单层决策树函数，这个单层决策树函数会通过设置一个巧妙的阈值，然后用阈值去判断当前特征向量的每一列的每一个元素是否大于阈值，据此输出类别预测
def buildStump(dataArr,classLabels,D):
# 将数据集和标签列表转为矩阵形式，其中标签定义为转置矩阵
    dataMatrix=np.mat(dataArr);labelMat=np.mat(classLabels).T
    # 计算矩阵的行数和列数
    m,n=np.shape(dataMatrix)
    # 对关键变量进行初始化，设置步长或区间总数，定义一个字典保存最优决策树信息（迭代的时候这里就用来保存每一次的最新的预测结果）
    # 初始化生成一个m行1列的全0元素的矩阵，这里是为后期最优单层决策树预测结果生成初始化值
    numSteps=10.0;bestStump={};bestClasEst=np.mat(np.zeros((m,1)))
    # 最小错误率初始化为+∞,inf是指正无穷的意思，其中错误率应该是一个正实数
    minError=np.inf
    # 遍历每一列的特征值,这里的n是原始特征矩阵的列数，机器学习实战中的n=2，那么就是分别遍历两列的特征值
    for i in range(n):
        #找出并定义每列中特征值的最小值和最大值
        rangeMin=dataMatrix[:,i].min();rangeMax=dataMatrix[:,i].max()
        #求取步长大小或者说区间间隔，这里的步长设置其实只是其中一种定义步长的算法
        stepSize=(rangeMax-rangeMin)/numSteps
        #遍历各个步长区间，这里实际上是遍历[-1,11]这个闭区间所有的整数
        for j in range(-1,int(numSteps)+1):
            #两种阈值过滤模式
            for inequal in ['lt','gt']:
                #阈值计算公式：最小值+j(-1<=j<=numSteps+1)*步长
                # 以机器学习实战的第一列为准，rangeMin=1，j=-1,stepSize=(2-1)/10=0.1,那么阈值threshVal=[1+(-1)*0.1]=0.9
                # 这里的阈值设置，其实是非常科学的，虽然我不知道为何这样设置，但是这样设置就是能够比较出来元素和阈值的大小
                threshVal=(rangeMin+float(j)*stepSize)
                #选定阈值后，调用阈值过滤函数分类预测，\表示换行符，如果取消，就需要将下一行换行回到当前行，否则会报错
                # stumpClassify(矩阵，第1列，0.9的阈值，lt),初次运行predictedVals的结果为array([[1.],[1.],[1.],[1.],[1.]])
                predictedVals=\
                    stumpClassify(dataMatrix,i,threshVal,inequal)
                # 既然predictedVals已经预测出来，那么就需要去比较预测的结果和真实结果的值的错误率，那么初始化错误向量
                # 这里初始化出来的向量是一个元素都为1的5行1列的矩阵，用来对错误的情况进行比对
                errArr=np.mat(np.ones((m,1)))
                #将错误向量中分类正确项置0
                errArr[predictedVals==labelMat]=0
                #计算加权的错误率，这里用到了矩阵乘法，矩阵与矩阵相乘，还是一个矩阵，但是一个行向量和一个列向量相乘，是一个数
                weightedError=D.T*errArr
                #打印相关信息，可省略
                print("当前遍历的列数为 %d, 阈值 %.2f, 当前大于小于类型: %s, 加权错误率= %.3f" %\
                      (i, threshVal, inequal, weightedError))
                #如果当前错误率小于当前最小错误率，将当前错误率作为最小错误率
                #存储相关信息
                if weightedError<minError:
                    minError=weightedError
                    bestClasEst=predictedVals.copy()
                    bestStump['dim']=i
                    bestStump['thresh']=threshVal
                    bestStump['ineq']=inequal
    #返回最佳单层决策树相关信息的字典，最小错误率，决策树预测输出结果
    return bestStump,minError,bestClasEst
```


```python
# 观察一下权值向量D在做些什么,这里初始化一个权值向量D，其中初始化的向量D的每个权重都是相等的，对应了每一个特征向量
D = np.mat(np.ones((5,1))/5)
print(D)
```

    [[0.2]
     [0.2]
     [0.2]
     [0.2]
     [0.2]]
    


```python
buildStump(datMat,classLabels,D)
```

    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: lt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: gt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: lt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: gt, 加权错误率= 0.400
    




    ({'dim': 0, 'thresh': 1.3, 'ineq': 'lt'}, matrix([[0.2]]), array([[-1.],
            [ 1.],
            [-1.],
            [-1.],
            [ 1.]]))




```python
# 开始定义最终的adaboost算法的主函数部分
```


```python
# adaBoost算法
# dataArr：数据特征矩阵
# classLabels:数据标签向量
# numIt:迭代次数
def adaBoostTrainDS(dataArr,classLabels,numIt=40):
    #弱分类器相关信息列表
    weakClassArr=[]
    #获取数据集行数
    m=np.shape(dataArr)[0]
    # 初始化权重向量的每一项值相等，在机器学习实战中，初始向量共计5个元素，每个元素都等于0.2，全部相加等于1，D的所有权重加起来总是等于1
    D=np.mat(np.ones((m,1))/m)
    # 累计估计值向量,这里初始化的结果是    matrix([[0.],[0.],[0.],[0.],[0.]]).T
    aggClassEst=np.mat(np.zeros((m,1)))
    #循环迭代次数40次，for循环从第一次开始，然后结束在40次
    for i in range(numIt):
        #根据当前数据集，标签及权重建立最佳单层决策树，这里会返回最佳决策树、错误率、分类结果三个结果
        bestStump,error,classEst=buildStump(dataArr,classLabels,D)
        #打印权重向量，初始化未迭代的权重向量为 [[0.2 0.2 0.2 0.2 0.2]]
        print("D:",D.T)
        #求单层决策树的系数alpha
        alpha=float(0.5*np.log((1.0-error)/(max(error,1e-16))))
        #存储决策树的系数alpha，添加到最佳决策树的位置
        bestStump['alpha']=alpha
        #将该决策树存入列表
        weakClassArr.append(bestStump)
        #打印决策树的预测结果
        print("classEst:",classEst.T)
        # 预测正确为exp(-alpha),预测错误为exp(alpha)
        # 即增大分类错误样本的权重，减少分类正确的数据点权重
        # 这里实际上是一个向量，存储每一个分类结果的对应权重，机器学习实战的案例中是5行1列的矩阵
        # 首轮迭代结果    matrix([[ 0.69314718],[-0.69314718],[-0.69314718],[-0.69314718],[-0.69314718]]).T
        expon=np.multiply(-1*alpha*np.mat(classLabels).T,classEst)
        #更新权值向量，这里开始第一轮的更新，后面for循环会不断的更新权重向量D，第一轮更新结果   matrix([[0.4],[0.1],[0.1],[0.1],[0.1]]).T
        D=np.multiply(D,np.exp(expon))
        # D是一个矩阵，这里D.sum()是对矩阵里面的所有元素进行求和，第一轮 D.sum() = 0.7999999
        D=D/D.sum() 
        # 累加当前单层决策树的加权预测值,这里本来aggClassEst是一个矩阵，但是假如alpha*classEst的结果是一个数组
        # 矩阵加法的规则是不能直接相加的，因为不是同型矩阵
        # 但是numpy自带了广播机制，具体可以百度，所以实际上这里是可以相加的，相加的规则是对应位置的横轴进行数据对应一一相加，测试一次就知道结果
        # 在本案例中不涉及广播机制，但是需要了解
        aggClassEst=aggClassEst + alpha*classEst
        print("aggClassEst",aggClassEst.T)
        #求出分类错的样本个数
        aggErrors=np.multiply(np.sign(aggClassEst)!=\
                    np.mat(classLabels).T,np.ones((m,1)))
        #计算错误率
        errorRate=aggErrors.sum()/m
        print("total error:",errorRate,"\n")
        #错误率为0.0退出循环
        if errorRate==0.0:break
    #返回弱分类器的组合列表
    return weakClassArr
```


```python
# 测试结果是迭代三次错误率就为0，因此就此结束循环
classifierArray=adaBoostTrainDS(datMat,classLabels,9)
```

    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: lt, 加权错误率= 0.600
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: gt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.200
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.800
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: lt, 加权错误率= 0.400
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: gt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: lt, 加权错误率= 0.600
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: gt, 加权错误率= 0.400
    D: [[0.2 0.2 0.2 0.2 0.2]]
    classEst: [[-1.  1. -1. -1.  1.]]
    aggClassEst [[-0.69314718  0.69314718 -0.69314718 -0.69314718  0.69314718]]
    total error: 0.2 
    
    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.625
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.375
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: lt, 加权错误率= 0.625
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: gt, 加权错误率= 0.375
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: lt, 加权错误率= 0.625
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: gt, 加权错误率= 0.375
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: lt, 加权错误率= 0.750
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: gt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.125
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.875
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: lt, 加权错误率= 0.250
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: gt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: lt, 加权错误率= 0.750
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: gt, 加权错误率= 0.250
    D: [[0.5   0.125 0.125 0.125 0.125]]
    classEst: [[ 1.  1. -1. -1. -1.]]
    aggClassEst [[ 0.27980789  1.66610226 -1.66610226 -1.66610226 -0.27980789]]
    total error: 0.2 
    
    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: lt, 加权错误率= 0.143
    当前遍历的列数为 0, 阈值 0.90, 当前大于小于类型: gt, 加权错误率= 0.857
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.357
    当前遍历的列数为 0, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.643
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: lt, 加权错误率= 0.357
    当前遍历的列数为 0, 阈值 1.10, 当前大于小于类型: gt, 加权错误率= 0.643
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: lt, 加权错误率= 0.357
    当前遍历的列数为 0, 阈值 1.20, 当前大于小于类型: gt, 加权错误率= 0.643
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.30, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.40, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.50, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.60, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.70, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.80, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: lt, 加权错误率= 0.286
    当前遍历的列数为 0, 阈值 1.90, 当前大于小于类型: gt, 加权错误率= 0.714
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: lt, 加权错误率= 0.857
    当前遍历的列数为 0, 阈值 2.00, 当前大于小于类型: gt, 加权错误率= 0.143
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: lt, 加权错误率= 0.143
    当前遍历的列数为 1, 阈值 0.89, 当前大于小于类型: gt, 加权错误率= 0.857
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: lt, 加权错误率= 0.500
    当前遍历的列数为 1, 阈值 1.00, 当前大于小于类型: gt, 加权错误率= 0.500
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.11, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.22, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.33, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.44, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.55, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.66, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.77, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.88, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: lt, 加权错误率= 0.571
    当前遍历的列数为 1, 阈值 1.99, 当前大于小于类型: gt, 加权错误率= 0.429
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: lt, 加权错误率= 0.857
    当前遍历的列数为 1, 阈值 2.10, 当前大于小于类型: gt, 加权错误率= 0.143
    D: [[0.28571429 0.07142857 0.07142857 0.07142857 0.5       ]]
    classEst: [[1. 1. 1. 1. 1.]]
    aggClassEst [[ 1.17568763  2.56198199 -0.77022252 -0.77022252  0.61607184]]
    total error: 0.0 
    
    


```python
classifierArray
```




    [{'dim': 0, 'thresh': 1.3, 'ineq': 'lt', 'alpha': 0.6931471805599453},
     {'dim': 1, 'thresh': 1.0, 'ineq': 'lt', 'alpha': 0.9729550745276565},
     {'dim': 0, 'thresh': 0.9, 'ineq': 'lt', 'alpha': 0.8958797346140273}]




```python

```
