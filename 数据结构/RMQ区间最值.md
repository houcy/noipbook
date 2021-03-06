# RMQ区间最值


## 基本常识

 - 从1开始数10个数,停下的数是:10,`10=10+1-1`
 - 从1开始数99个数,停下的数是:99,`99=99+1-1`
 - 从2开始数99个数,停下的数是:100,`100=99+2-1`
 - 从n开始数m个数,停下的数是:**n+m-1**

 - 从10开始往前数3个数,停下的数是:8,`8=10-3+1`
 - 从n开始往前数m个数,停下的数是:**n-m+1**

## rmq原理

我们设$$f[i,j]$$表示**从第i个元素开始的长度为$$2^j$$个元素的最值**

由于元素个数为$$2^j$$个，所以从中间平均分成两部分，每一部分的元素个数刚好为$$2^{j-1}$$个，如下图:

![1](./images/rmq1.png)

整个区间的最大值一定是左右两部分最大值的较大值，满足动态规划的最优原理

状态转移方程：

```math
f[i,j]=max(f[i,j-1],f[i+2^{j-1},j-1])
```
边界条件为$$f[i,0]=a[i]$$

这样就可以在$$O(nlogn)$$的时间复杂度内预处理f数组:

```c
for(i=1;i<=n;i++) f[i][0] = a[i]; //初始化

for(j=1;(1<<j)<=n;j++){ //1<<j 表示处理的范围
    for(i=1;i+(1<<j)-1<=n;i++) // i+(1<<j)-1<=n 表示所求的范围的最后一个值在原数组范围内
        f[i][j] = max (f[i][j-1],f[i+(1<<(j-1))][j-1]);
}
```

其中`i+(1<<j)-1`表示$$i+2^j-1$$表示所求的范围的最后一个值在原数组范围内


## query查询

想一想:我们有`1-->10`个元素,我们想查$$[1,8]$$范围内的最值怎么查?查$$[1,9]$$范围内的最值怎么查?

如果查$$[1,8]$$那就是$$f[1][3]$$
如果查$$[1,9]$$那就是$$f[1][3]$$和$$f[2][3]$$

对于查询区间$$[L,R]$$，令$$x$$为满足$$2^x<=R-L+1$$的最大整数,即$$x=int(log(R-L+1)/log(2))$$

$$[L,R]=[L,L+2^x-1] \cup [R-2^x+1,R]$$，两个子区间元素个数都是$$2^x$$个，如图:

![2](./images/rmq2.png)



```math
RMQ(L,R)=max(f[L,x],f[R+1-2^x,x])
```

```c
int query(int l,int r){
    int x = int( log(r-l+1)/log(2));
    return max(f[l][x],f[r-(1<<x)+1][r]);
}
```
## 具体代码

数据:

```
10 3
1 5 7 2 9 7 2 5 9 6
1 5
6 9
1 10
```

代码:

```c
#include <cstdio>
#include <cmath>

#define N 10000

int a[N];
int f[N][32];

int n,m;//n个数,m次查询


int max(int a,int b){
    if( a > b )
        return a;
    return b;
}

int query(int l,int r){
    int x = int( log(r-l+1)/log(2));
    return max(f[l][x],f[r-(1<<x)+1][r]);
}

void rmq(){ //预处理
    int i,j;
    for(i=1;i<=n;i++ ) f[i][0]  = a[i];
    for(j=1;(1<<j)<=n;j++)
        for(i=1;i+(1<<j)-1 <=n;i++){
            f[i][j] = max(f[i][j-1],f[i+(1<<(j-1))][j-1]);
        }
}

int main(){
    int i,j;
    scanf("%d%d",&n,&m);
    //读入
    for (i=1;i<=n;i++)
        scanf("%d",&a[i]);
    rmq();
    for (i=1;i<=m;i++){
        int t1,t2;
        scanf("%d%d",&t1,&t2);
        printf("%d\n",query(t1,t2));
    }
    return 0;
}
```

## 练习题目

 - P1440 求m区间内的最小值
 - P1816 忠诚
 - P2048 [NOI2010]超级钢琴
 - P2214 [USACO14MAR]哞哞哞Mooo Moo
 - P2216 [HAOI2007]理想的正方形
 - P2251 质量检测
 - P2471 [SCOI2007]降雨量
 - P2880 [USACO07JAN]平衡的阵容Balanced Lineup
 - P3763 [TJOI2017]DNA
 - P4085 [USACO17DEC]Haybale Feas
