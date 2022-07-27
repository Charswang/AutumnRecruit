https://blog.csdn.net/Wxiaozhong/article/details/124373954

在comparator里面，-1代表小于，0代表等于，1代表大于

- 返回负数或者0就不需要换位置
- 返回正数就需要更换位置

正常情况下直接返回o1-o2就是升序；o2-o1就是降序。