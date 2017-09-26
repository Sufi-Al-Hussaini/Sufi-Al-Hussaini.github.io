---
layout: post
title: Export MySQL tables to JSON
description: A simple and fast C program for exporting MySQL tables to JSON
date: '2017-09-22 11:43:02'
tags: MySQL JSON
---


## A simple and fast C program for exporting MySQL tables to JSON.

***

### Build
* Using CMake: 
```sh
mkdir -p build
cd build
cmake ..
```

* Using supplied Makefile:
```sh
# Set `BUILD_DIR` in Makefile
# Run make from root dir
make 
```

***

### Run

Pipe the output of your mysql query to `export-mysql-to-json`. 
This can be done from the command line like so:
```sh
$ mysql -u <username> -p<password> -e "select * from <schema>.<table>;" | ./build/export-mysql-to-json
```

Here's the resulting output for a sample table:

id | col1 | col2
-- | ---- | ----
1 | 11 | 12
2 | 22 | 22


```json
[ 
	{
		"id" : "1",
		"col1" : "11",
		"col2" : "12"
	},{
		"id" : "2",
		"col1" : "21",
		"col2" : "22"
	}
]
```


`export-mysql-to-json` uses column names as json keys by default, but you can change them using Mysql's `AS` statement like so:
```sh
$ mysql -u <username> -p<password> -e "select <column name> as <column name alias> from <schema>.<table>;" | ./build/export-mysql-to-json
```



