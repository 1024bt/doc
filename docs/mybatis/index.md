### 解决新增无法返回主键问题

```
在xml中加入这两个标签
useGeneratedKeys="true" 
keyProperty="deptId"
在MyBatis的<insert>标签中，可以通过设置useGeneratedKeys="true"和keyProperty属性来获取自动生成的主键。keyProperty指定了要将获取的主键值设置到哪个对象的哪个属性中。
```

