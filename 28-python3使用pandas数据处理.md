<!--markdown-->Dataframe 布尔逻辑使用`&`和`|`

```python
np.random.seed(0)
df = pd.DataFrame(np.random.randn(5,3), columns=list('ABC'))
>>> df
          A         B         C
0  1.764052  0.400157  0.978738
1  2.240893  1.867558 -0.977278
2  0.950088 -0.151357 -0.103219
3  0.410599  0.144044  1.454274
4  0.761038  0.121675  0.443863

>>> df.loc[(df.C > 0.25) | (df.C < -0.25)]
          A         B         C
0  1.764052  0.400157  0.978738
1  2.240893  1.867558 -0.977278
3  0.410599  0.144044  1.454274
4  0.761038  0.121675  0.443863


```	