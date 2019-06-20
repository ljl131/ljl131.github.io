---
title: C 查找前缀和后缀函数
date: 2019-06-19 13:43:22
tags: C语言
---
```
#include <stdio.h>
 
//查找后缀
bool hasEnding (char *& strFull,char*&  strEnd)
{
	char * pFull = strFull;
	while(*pFull != 0)
		pFull++;
 
	char * pEnd = strEnd;
	while(*pEnd != 0)
		pEnd++;
 
	while(1)
	{
		pFull--;
		pEnd--;
		if (*pEnd == 0)
		{
			break;
		}
 
		if (*pFull != *pEnd)
		{
			return false;
		}
 
	}
	return true;
}
 
//查找前缀
bool hasStarting (char *& strFull,char*&  strStart)
{
	char * pFull = strFull;
	char * pStart = strStart;
 
	while(1)
	{
		if (*pFull != *pStart)
		{
			return false;
		}
 
		pFull++;
		pStart++;
		if (*pStart == 0)
		{
			break;
		}
	}
	return true;
}
 
 
int main()
{
	char * url = "http://jpg";
	char * end = ".ajpg";
	bool b = hasEnding(url,end);
 
 
	char * start = "http://";
	b = hasStarting(url,start);
	return 0;
}
```
