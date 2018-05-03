---
layout:      post
title:       "wchar_t to string, string to wstring"
categories:  [软件开发, C++]
description: “wchar_t to string, string to wstring”
keywords:    C++, 软件开发
---

wchar_t to string

``` C++
#include <iostream>
#include "Windows.h"
#include <string>
void Wchar_tToString(std::string& szDst, wchar_t *wchar)
{
	wchar_t * wText = wchar;
	DWORD dwNum = WideCharToMultiByte(CP_OEMCP,NULL,wText,-1,NULL,0,NULL,FALSE);// WideCharToMultiByte的运用
	char *psText; // psText为char*的临时数组，作为赋值给std::string的中间变量
	psText = new char[dwNum];
	WideCharToMultiByte (CP_OEMCP,NULL,wText,-1,psText,dwNum,NULL,FALSE);// WideCharToMultiByte的再次运用
	szDst = psText;// std::string赋值
	delete []psText;// psText的清除
}
```

string to wstring
``` C++
#include <iostream>
#include "Windows.h"
#include <string>
void StringToWstring(std::wstring& szDst, std::string str)
{
	std::string temp = str;
	int len=MultiByteToWideChar(CP_ACP, 0, (LPCSTR)temp.c_str(), -1, NULL,0); 
	wchar_t * wszUtf8 = new wchar_t[len+1]; 
	memset(wszUtf8, 0, len * 2 + 2); 
	MultiByteToWideChar(CP_ACP, 0, (LPCSTR)temp.c_str(), -1, (LPWSTR)wszUtf8, len);
	szDst = wszUtf8;
	std::wstring r = wszUtf8;
	delete[] wszUtf8;
}
```

**本段代码使用的是Windows的API**
