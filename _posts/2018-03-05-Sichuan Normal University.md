---
layout: post  #这个不变
title: "程序设计刷题" #标题
date: 2019-03-31 19:14 #时间
description: "C语言程序设计"  #说明
tag: C #这是分类标签
---
1.判断m是否是素数

解：素数（只能被1和其自身整除的数）：
让m先后被2到根号m除，如果m能被2~根号m之中任何一个整数整除，则提前结束循环，此时i必然小于或等于k（即根号m）；如果m不能被2~k（即根号m）之间的任何一整数整除，则在完成最后一次循环后，i还要加1，因此i=k+1，然后才终止循环。在循环之后判别i的值是否大于或等于k+1，若是则表明未曾被2~k之间任何一整数整除过，因此输出“是素数”。

1不是素数要单独判断。
```
int isPrimer(int n)//返回1为素数，返回0为不是素数
{
  int i;
  if(n<=3)
    return n>1;  //因为小于等于3的自然数只有2和3是质数
  for(i=2;i<=sqrt(n);i++)
  {
    if(n%i==0)
      return 0;
  }
}
```

```
int main()
{
    int m,i,k;
    scanf("%d",&m);
    k = sqrt(m);
    for(i=2;i<=k;i++)
    {
        if(m%i == 0)
            break;
    }
    if(i > k && m!=1)
        printf("%d is a prime number.\n",m);
    else
        printf("%d is not a prime number.\n",m);
    return 0;
}
```
2.求100-200之间的素数
```
int main()
{
    int i,j;
    for(i=100;i<=200;i++)//100-200之间每个数都进行判断
    {
        for(j=2;j<=sqrt(i);j++) //判断素数
        {
            if(i%j == 0)
                break;
        }
        if(j>sqrt(i)) //未曾被2~sqrt(i)之间任一整数整除过
        {
            printf("%d\n",i); //打印素数
        }
    }
    return 0;
}
```
3.一球从100米高度自由落下，每次落地后反跳回原高度的一半；再落下，求它在第10次落地时，共经过多少米？第 10 次反弹多高？
```
int main()
{
    /*sum记录落地时的距离，h记录将弹起的高度*/
    float sum = 100,h = sum/2;// 第一次落地后，已经走了100m，反弹向上有50m
    int i;
    for(i=0;i<9;i++)//还有9次
    {
        sum=sum+h*2; //第二次落地时：是从第一次落地的距离+反弹与掉下来的距离
        h=h/2;
    }
    printf("sum = %f\n",sum);
    printf("h = %f\n",h);
    return 0;
}
```
4.有个 15 数按由大到小顺序存放在一个数组中，输入一个数，要求用折半查找法找出该数组中第几个元素的值。如果该数不在数组中，则打印出 "无此数"
```
int main()
{
     static int a[15]={260,230,180,156,102,101,62,59,24,23,17,8,7,1,0};
     int low=0,high=14,mid,key;
     scanf("%d",&key);
     while(low<=high)// 是小于等于，均为空间存在。
     {
         mid = (low+high)/2;
         if(a[mid]>key)
            low = mid+1;//因为由大到小排，所以是low
         else if(a[mid]<key)
            high=mid-1;//因为由大到小排，所以是high
         else
         {
            printf("第%d个元素\n",mid+1);
            return 0;//存在输出后，结束程序
         }
     }
     printf("无此数\n");//程序为提前结束，所以不存在。
    return 0;
}
```
5.选择排序
```
int main()
{
    int a[10],i,j,k,t;
    for(i=0;i<10;i++)
        scanf("%d",&a[i]);
    for(i=0;i<9;i++)//从小到大排
    {
        k = i;//k记录值最小的位置
        for(j=k+1;j<10;j++) //k+1
        {
            if(a[k]>a[j])//用k来比
                k=j;//k记录值最小的位置
        }
        if(k!=i)
        {
            t = a[i];
            a[i]=a[k];
            a[k]=t;
        }
    }
    printf("排序后\n");
    for(i=0;i<10;i++)
    {
        printf("%d ",a[i]);
    }
    return 0;
}
```
5.冒泡排序
```
void Bubble_sort(int a[],int len)
{
    int i,j,temp;
    for(i=0;i<len-1;i++)
    {
        for(j=len-1;j>i;j--)  //j>i，每次都确定一个数，所以可以j>i
        {
            if(a[j]>a[j-1])//从大到小排列
            {
                temp = a[j];
                a[j] = a[j-1];
                a[j-1] = temp;
            }
        }
    }
}
```
6.求最大公约数（GCD）：如果数a能被b整除，则b叫做a的约数。eg：12、16的公约数有：1、2、4，其中最大的是4为最大公约数。（几个整数中公有的约数，叫做这几个数的公约数；其中最大的一个，叫做这几个数的最大公约数）

6.1辗转相除法求gcd：
有两个整数a和b；
① a%b=c；
② 若c=0，则b即为两数的最大公约数
③ 若c≠0，则a=b，b=c，再执行①

例如求27和15的最大公约数过程为：27÷15 余12 ，15÷12余3，12÷3余0，因此，3即为最大公约数。

7.最小公倍数（lcm） = 两整数的乘积 / 最大公约数（gcd）
```
int gcd(int x,int y)
{
    int t;
    while(y!=0) /*余数不能为0，继续相除，直到余数为0*/
    {
        t = x%y; //求余数
        x = y;
        y = t;
    }/*余数为0后，也进行了x=y，y=t，所以此时最大公约数是x */
    return x;
}
int main()
{
    int a,b;
    while(~scanf("%d%d",&a,&b))
    {
        printf("%d与%d的gcd = %d\n",a,b,gcd(a,b));
        printf("%d与%d的lcm = %d\n",a,b,(a*b)/gcd(a,b));/*最小公倍数=两整数的乘积 / 最大公约数*/
    }
    return 0;
}
```
提供一种简写的方式：
```
int gcd(int a,int b)
{
    return b==0?a:gcd(b,a%b); /*余数为0，返回a；否则递归b，a%b*/
}
```
###文件部分 9.将100-200之间的素数找出来并保存在a1.dat文件中。
```
#include<stdio.h>
#include<stdlib.h>
#include<math.h>
int main()
{
    int i,j,k,n=0;
    FILE *fp;
    if((fp = fopen("D://a1.dat","w"))==NULL)//打开文件
    {
        printf("cannot open infile\n");
        return 0;
    }
    for(i=100;i<=200;i++)
    {
        k=sqrt(i);
        for(j=2;j<=k;j++)
        {
            if(i%j==0)
                break;
        }
        if(j>k)
        {
            n++;
            printf("%d%c",i,n%5==0?'\n':' ');
            fprintf(fp,"%d%c",i,n%5==0?'\n':' ');//输入数据到文件中
        }
    }
    fclose(fp);//关闭文件
    return 0;
}
```

10.直接插入排序:

A[0]为哨兵的情况：
```
void insert_sort(int a[],int n)//从小到大
{
    int i,j;
    for(i=2; i<=n; i++)//依次将A[2]~A[n]插入到前面已排序序列
    {
        if(a[i]<a[i-1])//若A[i]的关键码小于其前驱，需将A[i]插入有序表
        {
            a[0] = a[i];//复制为哨兵，A[0]不存放元素
            for(j=i-1; a[0]<a[j]; j--)//从后往前查找待插入位置
            {
                a[j+1]=a[j];//向后挪位
            }
            a[j+1] = a[0];//复制到插入位置
        }
    }

}
```
无哨兵的情况：
```
void insert_sort(int a[],int n)//从小到大
{
    int temp,i,j;
    for(i=1;i<n;i++)
    {
        if(a[i]<a[i-1])
        {
            temp = a[i];
            for(j=i-1;temp<a[j]&&j>=0;j--)
            {
                a[j+1]=a[j];
            }
            a[j+1]=temp;
        }
    }
}
```

11.折半插入排序
```
void insert_sort(int a[],int n) //从小到大
{
    int i,j,low,high,mid;
    for(i=2;i<=n;i++)//依次将A[2]~A[n]插入到前面已排序序列
    {
        a[0]=a[i];//将a[i]暂时存到a[0]
        low = 1;high = i-1;//有序里最左边是1 最右边是当前要插入的数的左边一个（即i-1）.
        while(low <= high)//折半查找（默认递增有序）
        {
            mid = (low+high)/2;
            if(a[0]>a[mid])
                low = mid + 1;//查找右半子表
            else
                high = mid - 1;//查找左半子表
        }
        //要插入的数的左边一个（即i-1）开始移动。
        for(j=i-1;j>=high+1;j--) /*此时要插的位置为high+1*/
        {                        /*所以要j一直往后移，一直将high+1移动后后面一个即（high+1）+1 也就是当j=high+1时，让a[j+1]=a[j]*/
            a[j+1]=a[j];
        }
        a[high+1] = a[0];//插入到high+1
    }
}
```
