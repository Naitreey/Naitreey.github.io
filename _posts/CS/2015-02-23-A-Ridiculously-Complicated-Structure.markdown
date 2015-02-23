---
layout: post
title: A Ridiculously Complicated Structure
tags: C
---

# A Ridiculously Complicated Structure #
Learning structure in C, I constructed the following structure. Yes, I was just testing how far my logic can go...

```c
#include <stdio.h>
#define LEN 50

struct department {
	char name[LEN];
	float fund;
};

struct province {
	char name[LEN];
	struct department distro[10];
	float fund;
};

struct government {
	char name[LEN];
	struct department general[10];
	struct province pv[30];
	float tax;
};

int main(void)
{
	struct government China[2] = {
		{
			.name = "PRC",
			.general = {
				[0]={"GuoWuYuan", 100},
				[1]={"ZhongYangDangXiao", 30},
				[2]={"WeiShengBu", 50},
				[3]={"ShangWuBu", 50},
				[4]={"SiFaBu", 50}
			},
			.pv = {
				[0]={
					"ShanXi", 
					.distro = {
						[0]={"GuoWuYuan.ShanXiBanShiChu", 5},
					},
					200,
				}
			},
			2.50
		},
		{
			.name = "ORC",
			.general = {
				[0]={"ZongTongFu", 1},
				[1]={"LiFaYuan", 7},
			},
			.pv = {
				[0]={
					"TaiWan",
					.distro = {
						[0]={"ShengZhengFu", 1},
					},
					2
				}
			},
			0.5
		}
	};

	struct government *country;

	country=China;
	printf( "%s: %s, %s\n" , country[0].name, (*country).pv[0].name, country->pv->distro->name );
	printf( "%s: %s, %s\n" , (*(China+1)).name, (country+1)->pv->name, (*(China[1].pv[0].distro)).name );
	
	return 0;
}
```

The reall magical thing is: It worked!

Incidentally, this post might well be _the_ one that get this very site blocked by GFW.
