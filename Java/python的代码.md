```python
import numpy as np
from matplotlib import pyplot as plt

def Zipf(a:np.float64,min:np.uint64,max:np.uint64,size=None):
    if min == 0:
        raise ZeroDivisionError("")
    v = np.arange(min,max+1)
    p = 1.0 / np.power(v,a)
    p /= np.sum(p)
    return np.random.choice(v,size=size,replace=True,p=p)
min = np.uint64(1)
max = np.uint64(7000)

q = Zipf(1.2,min,max,2000000)
print(q)

# h是每个文件访问的次数
h,bins=np.histogram(q,bins=int(max-min+1),range=(min-0.5,max+0.5))
print(h)
print(bins)

plt.hist(q,bins=bins)
# plt.yscale("log")
plt.title("Zipf")
plt.show()


# 生成数据集文件
file=open('./zipf_data_7000_2000000_1_2.txt','w')
for val in q:
    file.write(str(val)[:-2]+'\n')
file.close()
```

```python
import numpy as np
from matplotlib import pyplot as plt
import math
x=np.arange(1,7001)
y=[]
for i in h:
    y.append(i)
y.sort(reverse=True)
plt.plot(x,y,label="zipf1.2")
plt.xlabel("File Rank")
plt.ylabel("File Count")
plt.yscale("log")
plt.legend()
# plt.savefig("./zipf1.2_access_plot.png")
plt.show();
```

读取hm1的访问情况

```python
f=open("hm_1_onlyRead_AccessNumber.txt")
line = f.readline()
dict={}
list=[]
t=[]
while line:
    t.append(line[0:-1])
    s=line[0:-1]
    dict[s]=dict.get(s,0)+1
    line=f.readline()
f.close()
```

```python
import numpy as np
from matplotlib import pyplot as plt
import math

for k in dict:
    list.append(dict[k])
print(list)

x=np.arange(1,5229)
y=list
plt.plot(x,y,label='sigmoid')
plt.xlabel("x")
plt.ylabel("y")
plt.legend()
plt.show()
```

```python
list.sort(reverse=True)
x = np.arange(1,5229)
y=list
plt.plot(x,y,label="hm1")
plt.xlabel("File Rank")
plt.ylabel("File Count")
plt.yscale("log")
plt.legend()
plt.savefig("./hm1_access_plot.png")
plt.show()


# sum(list[0:500])
```



