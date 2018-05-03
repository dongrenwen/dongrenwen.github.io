---
layout:      post
title:       "通过注册表读取Matlab列表并判断可执行文件是否存在的Demo"
categories:  [软件开发, C++]
description: "通过注册表读取Matlab列表并判断可执行文件是否存在的Demo"
keywords:    Windows, 软件开发, C++, 注册表
---

一段通过注册表读取Matlab列表并判断可执行文件是否存在的Demo

``` C++
#include <iostream>
#include "Windows.h"
#include <string>

LPCTSTR matlab_path_array[] = {
	TEXT("Matlab.Application.Single.7.10\\CLSID"),
	TEXT("Matlab.Application.Single.7.11\\CLSID"),
	TEXT("Matlab.Application.Single.7.12\\CLSID"),
	TEXT("Matlab.Application.Single.7.13\\CLSID"),
	TEXT("Matlab.Application.Single.7.14\\CLSID"),
	TEXT("Matlab.Application.Single.8.0\\CLSID"),
	TEXT("Matlab.Application.Single.8.1\\CLSID"),
	TEXT("Matlab.Application.Single.8.2\\CLSID"),
	TEXT("Matlab.Application.Single.8.3\\CLSID"),
	TEXT("Matlab.Application.Single.8.4\\CLSID"),
	TEXT("Matlab.Application.Single.8.5\\CLSID"),
	TEXT("Matlab.Application.Single.8.6\\CLSID"),
	TEXT("Matlab.Application.Single.9.0\\CLSID"),
	TEXT("Matlab.Application.Single.9.1\\CLSID")
};

std::string matlab_names[] = {
	"Matlab R2010a",
	"Matlab R2010b",
	"Matlab R2011a",
	"Matlab R2011b",
	"Matlab R2012a",
	"Matlab R2012b",
	"Matlab R2013a",
	"Matlab R2013b",
	"Matlab R2014a",
	"Matlab R2014b",
	"Matlab R2015a",
	"Matlab R2015b",
	"Matlab R2016a",
	"Matlab R2016b"
};

int get_supported_matlab(char* versions)
{
	char out_versions[1024] = {0};
	bool first = false;
	for (int i = 0; i < 14; i++)
	{
		HKEY hKey = NULL;
		WCHAR wcCLSID[MAX_PATH] = {'\0'};
		DWORD dwSize = sizeof(DWORD);
		DWORD dwType = REG_SZ;
		LONG ret;
		ret = RegOpenKeyEx(HKEY_CLASSES_ROOT, matlab_path_array[i], 0, KEY_READ, &hKey);
		wprintf(L"RegOpenKeyEx returns %d\n%s\n", ret, matlab_path_array[i]);
		if (ERROR_SUCCESS == ret)
		{
			ret = RegQueryValueEx(hKey, TEXT(""), 0, &dwType, NULL, &dwSize);
			wprintf(L"RegQueryValueEx returns %d, dwSize=%d\n", ret, dwSize);

			ret = RegQueryValueEx(hKey, TEXT(""), 0, &dwType, (LPBYTE)&wcCLSID, &dwSize);
			wprintf(L"RegQueryValueEx returns %d, dwSize=%d\n", ret, dwSize);
			if (ERROR_SUCCESS == ret)
			{
				wprintf(L"wcCLSID: %s\n", wcCLSID);
				
				HKEY hKey1 = NULL;
				DWORD dwSize1 = sizeof(DWORD);
				DWORD dwType1 = REG_SZ;
				WCHAR wcClsidPath[MAX_PATH] = {'\0'};
				WCHAR wcClsidValue[MAX_PATH] = {'\0'};
				swprintf_s(wcClsidPath, MAX_PATH, L"SOFTWARE\\Classes\\CLSID\\%s\\LocalServer32", wcCLSID);

				//ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE, wcClsidPath, 0, KEY_READ, &hKey1);
				//ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE, wcClsidPath, 0, KEY_WOW64_32KEY|KEY_READ, &hKey1);
				ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE, wcClsidPath, 0, KEY_WOW64_64KEY|KEY_READ, &hKey1);
				wprintf(L"RegOpenKeyEx returns %d\n%s\n", ret, wcClsidPath);
				if (ERROR_SUCCESS == ret)
				{
					ret = RegQueryValueEx(hKey1, TEXT(""), 0, &dwType1, NULL, &dwSize1);
					wprintf(L"RegQueryValueEx returns %d, dwSize=%d\n", ret, dwSize1);

					ret = RegQueryValueEx(hKey1, TEXT(""), 0, &dwType1, (LPBYTE)&wcClsidValue, &dwSize1);
					wprintf(L"RegQueryValueEx returns %d, dwSize=%d\n", ret, dwSize1);
					if (ERROR_SUCCESS == ret)
					{
						wprintf(L"wcClsidValue: %s\n", wcClsidValue);
						std::wstring wsExePathTmp(wcClsidValue);
						std::wstring wsExePath = wsExePathTmp.substr(0,wsExePathTmp.rfind(L" /MLAutomation"));
						LPCWSTR exeName = (LPCWSTR)wsExePath.c_str();

						WIN32_FIND_DATA  FindFileData;
						HANDLE hFind;

						hFind = FindFirstFile(exeName, &FindFileData);
						if (hFind != INVALID_HANDLE_VALUE)
						{
							if (first == false)
							{
								sprintf_s(out_versions, 1024, "%s", matlab_names[i].c_str());
								first = true;
							}
							else
							{
								sprintf_s(out_versions, 1024, "%s/%s", out_versions, matlab_names[i].c_str());
							}
							wprintf(L"The first file found is %s ", FindFileData.cFileName);
							FindClose(hFind);
						}
					}
					RegCloseKey(hKey1);
				}
			}
			RegCloseKey(hKey);
		}
	}
    memcpy(versions, out_versions, strlen(out_versions)+1);
	return 0;
}
```

**本段代码使用的是Windows的API**

