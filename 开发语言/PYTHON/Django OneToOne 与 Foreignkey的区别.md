# OneToOne Relationship与Foreignkey的区别



**两者的反向查询是有差别的：**

ForeignKey反向查询返回的是一个列表（一个车有多个轮子）。

OneToOneField反向查询返回的是一个模型示例（因为一对一关系）。



**OneToOneField**

A one-to-one relationship. Conceptually, this is similar to a `ForeignKey` with `unique=True`, but the "reverse" side of the relation will directly return a single object.





