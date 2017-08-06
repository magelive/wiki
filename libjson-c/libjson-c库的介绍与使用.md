<!--
author: Magelive
date: 2017-02-10
title: libjson-c库的介绍与使用
tags: 
category: libjsonC
status: publish
summary: 
head: 
images: 
-->

## json-c库的介绍与使用
### json-c库介绍
json-c库是为了方便在C语言中使用与解析json数据格式。json对象中的几种常见的数据格式与C语言中的觉格式相对应，如：int、char、double等。同时json中还有自己的数据格式，在使用时需要转换成Json自己的类型
### json-c库安装
#### 下载
```
git clone git@github.com:json-c/json-c.git
```
#### 编译与安装
##### 本地编译与安装
```
./configure
make
```
##### 交叉编译与安装
```
./configure CC= arm-linux-gcc –host=arm-linux –build=i686-pc-linux
make
```

### json-c使用
#### 库函数说明
##### Json对象类型
* json_type_null: null数据
* json_type_int	: int类型数据
* json_type_double : double类型数据
* json_type_boolean : 布尔型数据
* json_type_string : string类型数据
* json_type_object : key/value对集数据
* json_type_array :	数组型数据

##### Json对象常用操作函数
```js
	//创建object
	struct json_object* json_object_new_boolen(json_bool b);
	struct json_object* json_object_new_int(int32_t i);
	struct json_object* json_object_new_int64(int64_t i);
	struct json_object* json_object_new_double(double d);
	
	/*创建精确的double值，将其放置ds中*/
	struct json_object* json_object_new_double_s(double d, const char *ds);

	struct json_object* json_object_new_string(char *s);
	struct json_object* json_object_new_string_len(const char *s, int len);
	struct json_object* json_object_new_array();
	struct json_object* json_object_new_object();

	//set
	/*插入或是替换掉原有array中指定位置的值
	int json_object_array_put_idx(struct json_object *obj, int idx,
				     struct json_object *val);
	

	//获取obj 数据
	int json_object_object_length(struct json_object* obj);
	struct json_object* json_object_object_get(struct json_object* obj,const char *key);	
	struct array_list* json_object_get_array(struct json_object *obj);

	//从array中指定位置获取到json_object
	struct json_object* json_object_array_get_idx(struct json_object *obj,
						     int idx);

	json_bool json_object_get_boolean(struct json_object *obj);
	int32_t json_object_get_int(struct json_object *obj);
	int64_t json_object_get_int64(struct json_object *obj);
	double json_object_get_double(struct json_object *obj);
	const char* json_object_get_string(struct json_object *obj);
	int json_object_get_string_len(struct json_object *obj);
	enum json_type json_object_get_type(struct json_object *obj);
	
	int json_object_array_length(struct json_object *jso);

	/*获取指定的值至Value中，value为双指针
	json_bool json_object_object_get_ex(struct json_object* obj,
										const char *key,
                                        struct json_object **value);
	

	//obj 添加字段
	void json_object_object_add(struct json_object* obj, const char *key,structjson_object *val);
	int json_object_array_add(struct json_object *obj, struct json_object *val);
	
	//obj 删除字段
	void json_object_object_del(struct json_object* obj, const char *key);

	//obj array 中替换数据
	int json_object_array_put_idx(struct json_object *obj, int idx, structjson_object *val);
	
	//obj array sort
	void json_object_array_sort(struct json_object *jso, int(*sort_fn)(const void*, const void *));	

	//判断
	int json_object_is_type(struct json_object *obj, enum json_type type);
	
	//遍历
	json_object_object_foreach(obj,key,val){
	}
	
	//类型转换
	/* 由 json_object 转换成 c 字符串
	const char* json_object_to_json_string(struct json_object *obj);
	
	/*根据flags标志转换成字符串
	const char* json_object_to_json_string_ext(struct json_object *obj, int flags);
	
	/*将字符串转换成json对象*/
	struct json_object* json_tokener_parse(const char *str);
	
```
**特别注意的是，在使用new或是重新get到一个json_object后，需要使用json_object_put函数将json_object的计数减1**
以下函数均是在使用完后需要使用json_object_put函数来回收处理的
```
	struct json_object* json_object_new_boolen(json_bool b);
	struct json_object* json_object_new_int(int32_t i);
	struct json_object* json_object_new_int64(int64_t i);
	struct json_object* json_object_new_double(double d);
	struct json_object* json_object_new_double_s(double d, const char *ds);
	struct json_object* json_object_new_string(char *s);
	struct json_object* json_object_new_string_len(const char *s, int len);
	struct json_object* json_object_new_array();
	struct json_object* json_object_new_object();
	struct json_object* json_tokener_parse(const char *str);
```
```
	int json_object_put(struct json_object *jso);
```
#### 范例
以下范例是从一段Json字符串的数据中获取值
```c
int json_get_value(const char *json, char *key, char **value, enum json_type *type){
	struct json_object *js_obj = json_tokener_parse(json);
	if (!js_obj){
		printf("Not is json format: %s", string);
		return -1;
	}
	struct json_object *tmp_obj;
	if (!json_object_object_get_ex(js_obj, key, &tmp_obj)) {
		json_object_put(new_obj);
		return -1;
	}
	*type = json_object_get_type(tmp_obj);
	switch(*type){
		case json_type_boolean:
			(json_bool)(*(*value)) = json_object_get_boolean(tmp_obj);
			break;
		case json_type_int:
			if (!(*value)){
				*value = (char *)malloc(sizeof(int64_t));
			}
			(int64_t)(*(*value)) = json_object_get_int64(tmp_obj);
			break;
		case json_type_double:
			if (!(*value)){
				*value = (char *)malloc(sizeof(double));
			}
			(double)(*(*value)) = json_object_get_double(tmp_obj);
			break;
		case json_type_string:
			if (!*(value)){
				*value = (char *)calloc(json_object_get_string_len(tmp_obj)+1, 1);
			}
			strncpy(*value, json_object_get_string(struct json_object *jso), 
					json_object_get_string_len(tmp_obj));
			break;
		default:
			char *obj_str = json_object_to_json_string(tmp_obj);
			if (obj_str){
				if (!(*value))
					*value = (char *)calloc(strlen(obj_str)+1,1)
				strncpy(value, obj_str, strlen(obj_str));
			}
			break;
	}
	return 0;			
}
```


### 参考
>[libjson wiki](https://github.com/json-c/json-c/wiki)
>[JSON C库的使用](http://blog.csdn.net/cuishumao/article/details/10197941)
>[JSON_C语言开发指南](http://wenku.baidu.com/link?url=JH5oZJ6wYr7BYEDiPznzIWPksOliUFG4ub29Lmc2Zw4_mBpv2zVM5ydJ7a5x1qJ9-UxcEdUOEt32yYvVT8KnXmhJrfdAxiOIrUYr5D7VUzK)