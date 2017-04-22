---
title: 网易云课堂C进阶期末编程题 - 大数计算
date: 2015-05-10 10:00
tags: C
---

很大的数就没办法用int或是long long这样的类型直接计算了，用double则无法保证精度什么的，所以，得自己写程序来算。你的程序要读入两个很大的数，范围在[-1050,1050]内，然后你的程序要计算它们的和、差及积并输出。

**输入格式:**

两行，每行一个数字。

**输出格式：** 

三行，每行一个数字，依次表示输入的数字的和、差及积。 <!--more-->

**输入样例：**
``` bash
1853244628050278
506996688545785164
```

**输出样例：**
``` bash
508849933173835442
-505143443917734886
939588889486756266731803978475592
```

这是3月份在网易云课堂开课的c语言程序设计进阶的期末编程题，当时在规定时间没能做出来，主要还是思路比较乱，加上倒计时，心态坏了。今天重写了一下，并且拿小号提交，一次过了，这里把代码分享一下。

大概的方法大家能够想到，按照一般竖式运算的方式就可以进行，先计算个位，然后十位...比较难的是，比如小数减大数就相当于负的大数减小数，正数加负数时如果负数的绝对值大于正数，那么相当于负的负数绝对值减正数。换句话说，如果是做减法，保证做减法的函数保证是大数减小数就容易了做了，故在前期需要对两个数字进行一些分析，包括正负以及绝对值大小的比较。

``` c
#include<stdio.h>
#include<string.h>

void calsum(int *a, int *b, int *ans){
	//从最小位开始依次相加，当超过10给后一位加1
	for(int i=0;i<60;++i){
		ans[i]+=a[i]+b[i];
		if(ans[i]>=10){
			ans[i]%=10;
			++ans[i+1];
		}
	} 
}

void calmins(int *a, int *b, int *ans){
	//已保证a比b大 
	//从最小位开始依次相减
	for(int i=0;i<60;++i){
		ans[i]+=a[i]-b[i];
		while(ans[i]<0){
			--ans[i+1];
			ans[i]+=10;
		}
	} 
}

void printans(int *ans, int len){
	int flag=0;  //标记已经出现非0首位数字 
	for(int i=len-1;i>=0;--i){
		if(ans[i]!=0) flag=1;
		if(flag){
			printf("%d",ans[i]); 
		}
	}
	if(!flag) printf("0");
}

int main(){
	char a[60],b[60];
	gets(a);
	gets(b);
	int n1[60]={0},n2[60]={0},n1sign=1,n2sign=1; //sign=1为正数，0为负数 
	int n1len=strlen(a),n2len=strlen(b);
	if(a[0]=='-') n1sign=0;
	if(b[0]=='-') n2sign=0;
	for(int i=n1len-1,j=0;i>=0&&a[i]!='-';--i,++j){
		n1[j]=a[i]-'0';
	}	
	for(int i=n2len-1,j=0;i>=0&&b[i]!='-';--i,++j){
		n2[j]=b[i]-'0';
	}
	
	//判断a、b的绝对值大小，abig 
	int abig=1;
	if(!n1sign) --n1len; //当a<0时，a绝对值总位数-1 
	if(!n2sign) --n2len; //当b<0时，b绝对值总位数-1 
	if(n2len>n1len) abig=0;  //当b比a位数多的时候，b大于a
	else if(n1len==n2len){  //当位数一样时，比较每一位的大小 
		for(int i=n1len;i>=0;--i){
			if(n1[i]>n2[i]) break;
			else if(n1[i]==n2[i]) continue;
			else{ abig=0; break; }
		}
	}  
	
	//加法运算 
	int sum[60]={0}; //记录加法结果
	if(n1sign&&n2sign){  //ab都是正值，绝对值相加 
		calsum(n1,n2,sum);
		printans(sum,60);
	}else if(!n1sign&&!n2sign){  //ab都是负数，绝对值相加，再加负号 
		printf("-");
		calsum(n1,n2,sum);
		printans(sum,60);		
	}else if(n1sign&&abig){  //a是正数且a的绝对值大 
		calmins(n1,n2,sum);
		printans(sum,60);		
	}else if(n1sign&&!abig){  //a是正数，但b绝对值大 
		calmins(n2,n1,sum);
		if(sum[0]!=0) printf("-");
		printans(sum,60);		
	}else if(n2sign&&abig){  //b是正数，但a绝对值大 
		calmins(n1,n2,sum);
		if(sum[0]!=0) printf("-");
		printans(sum,60);
	}else{                   //b是正数且b绝对值大 
		calmins(n2,n1,sum);
		printans(sum,60);		
	}
	
	printf("\n"); 
	//减法运算
	int mins[60]={0}; //记录减法结果 
	if(n1sign&&!n2sign){
		calsum(n1,n2,mins);
		printans(mins,60);
	}else if(!n1sign&&n2sign){
		printf("-");
		calsum(n1,n2,mins);
		printans(mins,60);		
	}else if(n1sign&&abig){
		calmins(n1,n2,mins);
		printans(mins,60);
	}else if(n1sign&&!abig){
		calmins(n2,n1,mins);
		if(mins[0]!=0) printf("-");
		printans(mins,60);			
	}else if(!n1sign&&abig){
		calmins(n1,n2,mins);	
		if(mins[0]!=0) printf("-");
		printans(mins,60);		
	}else{
		calmins(n2,n1,mins);
		printans(mins,60);
	}
	
	printf("\n");
	//乘法运算 
	int product[121]={0};
	for(int i=0;i<60;++i){
		for(int j=0;j<60;++j){
			product[i+j]+=n1[i]*n2[j];
			while(product[i+j]>=10){
				product[i+j+1]+=product[i+j]/10;
				product[i+j]%=10;
			}
		}
	} 
	if((n1sign&&!n2sign)||(!n1sign&&n2sign)) printf("-");
	printans(product,121);	
	
	return 0;
} 
```