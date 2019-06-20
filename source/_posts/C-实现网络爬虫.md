---
title: C++实现网络爬虫
date: 2019-06-19 18:44:05
tags: 项目
---
# 原理
根据起始url得到网页的HTML代码。解析此HTML代码得到新的URL和图片资源（任何有用的资源）的地址，新的URL继续此过程。下载图片在一个新的线程里。
## 代码
CHttp.h
```
#include<iostream>
#include<windows.h>
#include<string>
#include<queue>
//#include<WinSock2.h>在windows里边
using namespace std;

#pragma comment(lib,"ws2_32.lib")//网络的库

class CHttp
{
private:
	string m_host;
	string m_object;
	SOCKET m_socket;
	bool AnalyseUrl(string url);//解析URL\http
	bool AnalyseUrl2(string url);//\https
	bool init();//初始化套接字
	bool Connect();//连接web服务器
public:
	CHttp(void);
	~CHttp(void);
	string FetchGet(string url);//通过Get方式获取网页
	void AnalyseHtml(string html);//解析网页，获得图片地址和其他的链接
};
```
CHttp.cpp
```
#include "CHttp.h"

CHttp::CHttp(void)
{

}


CHttp::~CHttp(void)
{
	closesocket(m_socket);
	WSACleanup();
}

//解析URL\http
bool CHttp::AnalyseUrl(string url)
{
	if(string::npos == url.find("http://"))
		return false;
	if(url.length()<=7)
		return false;
	int pos =url.find('/',7);
	if(pos==string::npos)
	{
		m_host=url.substr(7);
		m_object='/';
	}
	else
	{
		m_host=url.substr(7,pos-7);
		m_object=url.substr(pos);
	}
	if(m_host.empty())
		return false;
	return true;
}

//解析URL\https
bool CHttp::AnalyseUrl2(string url)
{
	if(string::npos == url.find("https://"))
		return false;
	if(url.length()<=8)
		return false;
	int pos =url.find('/',8);
	if(pos==string::npos)
	{
		m_host=url.substr(8);
		m_object='/';
	}
	else
	{
		m_host=url.substr(8,pos-8);
		m_object=url.substr(pos);
	}
	if(m_host.empty())
		return false;
	return true;
}

bool CHttp::init()
{
	//1 请求协议版本
	WSADATA wsaData;
	WSAStartup(MAKEWORD(2, 2), &wsaData);
	if (LOBYTE(wsaData.wVersion) != 2 ||
		HIBYTE(wsaData.wVersion) != 2){
			printf("请求协议版本失败!\n");
			return false;
	}
	//printf("请求协议成功!\n");
	//2 创建socket
	m_socket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (SOCKET_ERROR == m_socket){
		printf("创建socket失败!\n");
		WSACleanup();
		return false;
	}
	//printf("创建socket成功!\n");
	return true;
}

//连接web服务器
bool CHttp::Connect()
{
	//DNS服务器：将域名解析成IP地址
	hostent *p = gethostbyname(m_host.c_str());
	if(p==NULL)
		return false;
	SOCKADDR_IN sa;
	sa.sin_family=AF_INET;
	sa.sin_port=htons(80);//http的默认端口，https的默认端口443
	memcpy(&sa.sin_addr,p->h_addr,4);

	if(-1==connect(m_socket,(SOCKADDR*)&sa,sizeof(sa)))
	{
		cout<<"服务器连接失败"<<endl;
		return false;
	}
	else
	{
		//cout<<"服务器连接成功"<<endl;
		return true;
	}
}

string CHttp::FetchGet(string url)//通过Get方式获取网页
{
	string html;

	//解析url
	if(false==AnalyseUrl(url))
	{
		if(false==AnalyseUrl2(url))
		{
			cout<<"Html解析失败"<<endl;
			return "";
		}
	}
	//cout<<"主机名"<<m_host<<"\t\t"<<"资源名"<<m_object<<endl;
	if(false==init())//初始化套接字
	{
		return "";
	}
	if(false==Connect())//连接服务器
	{
		return "";
	}
	//发送Get请求  Get请求数据
	string request = "GET " + m_object + 
		" HTTP/1.1\r\nHost:" + m_host + 
		"\r\nConnection: Close\r\n\r\n";

	if( SOCKET_ERROR ==send( m_socket, request.c_str(), request.size(), 0 ) )
	{
		cout << "send request error" <<endl;
		closesocket( m_socket );
		return "";
	}

	//接收数据
	char ch;
	while(recv(m_socket,&ch,1,0))
	{
		html+=ch;
	}
 
	return html;
}
//判断是否以什么结尾
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
void CHttp::AnalyseHtml(string html)//解析网页，获得图片地址和其他的链接
{
	int startIndex =0;
	int endIndex=0;
	//找到所有的图片
	for(int pos=0;pos<html.length();)
	{
		startIndex=html.find("src=\"",startIndex);
		if(startIndex==-1)
		{
			break;
		}
		startIndex+=5;
		endIndex=html.find("\"",startIndex);
		//找到资源链接
		string src = html.substr(startIndex,endIndex-startIndex);
		char *src1 = (char *)src.c_str();
		//cout<<src<<endl;
		//判断连接是否是想要的资源
		char *strend=".jpg";
		if(hasEnding(src1,strend)==true)
		{
			/*if(-1!=src.find("t_s960x600c5"))*/
			if(-1!=src.find("t_s1920x1080c5"))
			{
				cout<<src<<endl;
				//新建一个线程来下载图片
				extern queue<string> p;
				p.push(src);
				extern void loadImage();
				CreateThread(NULL, NULL,(LPTHREAD_START_ROUTINE)loadImage, 
					NULL, NULL, NULL);
			}
			/*system("pause");*/
		}
		startIndex=endIndex+1;
		//system("pause");
	}

	startIndex =0;
	//找到其他URL地址
	for(int pos=0;pos<html.length();)
	{
		startIndex=html.find("href=\"",startIndex);
		if(startIndex==-1)
		{
			break;
		}
		startIndex+=6;
		endIndex=html.find("\"",startIndex);
		//找到资源链接
		string src = html.substr(startIndex,endIndex-startIndex);
		char *src1 = (char *)src.c_str();
		//cout<<src<<endl;
		//判断连接是否是想要的资源
		char *strend=".html";
		if(hasEnding(src1,strend)==true)
		{
			if((-1!=src.find("bizhi")|| -1!=src.find("showpic")) && -1==src.find("http://"))			
			{
				string url="http://desk.zol.com.cn"+src;
				extern queue<string> q;
				q.push(url);
				//cout<<url<<endl;
			}			
		}
		startIndex=endIndex+1;
		//system("pause");
	}

}
```
main.cpp
```
#include "CHttp.h"
#include <urlmon.h>

#pragma comment(lib, "urlmon.lib")

queue<string> q;//url队列
queue<string> p;//图片url队列

void StartCatch(string url);
int main()
{
	cout<<"*****************************************"<<endl<<endl;
	cout<<"           欢迎使用网络爬虫系统          "<<endl;
	cout<<"              开发者：a_byudao           "<<endl<<endl;
	cout<<"*****************************************"<<endl<<endl;

	//创建一个文件夹,点表示当前目录
	CreateDirectory("./image",NULL);

	////从键盘输入一个起始url
	string url;
	//cout<<"请输入起始url:";
	//cin>>url;
	url="http://desk.zol.com.cn/";//爬的是这个网站，可自行修改
	//开始抓取
	StartCatch(url);

	system("pause");
	return 0;
}

void StartCatch(string url)
{
	
	q.push(url);

	while(!q.empty())
	{
		//取出url
		string currenturl = q.front();
		q.pop();

		CHttp http;
		//发送一个Get请求
		string html=http.FetchGet(currenturl);
		//cout<<html;
		http.AnalyseHtml(html);
	}
}


//下载图片的线程
static int num=0;
void loadImage()
{
	while(!p.empty())
	{
		string currenturl = p.front();
		p.pop();
		char Name[20]={0};
		num++;
		sprintf(Name,"./image/%d.jpg",num);
		
		if(S_OK==URLDownloadToFile(NULL, currenturl.c_str(), Name, 0, 0))
		{
			cout<<"download ok"<<endl;
			if(num==24)//爬24张就结束了，也可以去掉这句话
			{
				exit(0);
			}
		}
		else
		{
			cout<<"download error"<<endl;
		}
	}

}
```
大家也可以访问我的个人博客[豆浆and油条er](https://ljl131.github.io/)、个人公众号搜索：豆浆and油条er,或者直接扫描头像
