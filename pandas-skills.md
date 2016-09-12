---
title: pandas常用计算技巧
date: 2016-08-11
category : 数据挖掘
published : False
tags : [数据挖掘，pandas，python]
---
1. 计算一列或者一行数据的种类:
   s.value_counts()
2. 把矩形数据变成方形数据：
```python
   index = df.index.union(result.columns)
   df = df.reindex(index=index, columns=index, fill_value=0.0)
```
3. 计算数据对角线的和：`np.trace(df, 0)`

4. 计算数据矩形所有元素的和：`df.sum(axis=0).sum()`

5. 遍历DataFrame的行与列：

   ```python
   # 遍历列
   for col in df.columns:
       column = df[col]
   # 遍历行
   for i in range(len(df)):
       row = df.iloc[i]
   for i, row in df.iterrows():
       dosomething()
   ```

6. 判断数据类型：

   ```python
   np.issubdtype(column.dtype, np.number)
   ```

7. 选择数据的某些列:

   ```python
   df[['col1','col2']]
   ```

   ​