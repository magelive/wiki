<!--
author: Magelive
date: 2017-02-10
title: json_object分析
tags: 
category: libjsonC
status: publish
summary: 
head: 
images: 
-->

struct json_object是json-c库中基础结构，以此结构为基础，构建了json-c的基础操作等。
## json_object

### 基础的typedef
```
typedef int json_bool;
typedef struct printbuf printbuf;
typedef struct lh_table lh_table;
typedef struct array_list array_list;
typedef struct json_object json_object;
typedef struct json_object_iter json_object_iter;
typedef struct json_tokener json_tokener;

```
### struct json_object
json_object详细结构定义在json_object_private.h中，其详细数据定义如下：
```
struct json_object
{
	enum json_type o_type;
	json_object_private_delete_fn *_delete;
	json_object_to_json_string_fn *_to_json_string;
	int _ref_count;
	struct printbuf *_pb;
	union data {
		json_bool c_boolean;
		double c_double;
		int64_t c_int64;
		struct lh_table *c_object;
		struct array_list *c_array;
		struct {
			char *str;
			int len;
		} c_string;
	} o;
	json_object_delete_fn *_user_delete;
	void *_userdata;
};
```

#### o_type
o_type表示objcet的类型，类型值如下：
```
typedef enum json_type {
  /* If you change this, be sure to update json_type_to_name() too */
  json_type_null,
  json_type_boolean,
  json_type_double,
  json_type_int,
  json_type_object,
  json_type_array,
  json_type_string,
} json_type;
```
我们可使用 json_type_to_name获取type的名称及描述
```
#define NELEM(a)        (sizeof(a) / sizeof(a[0]))
static const char* json_type_name[] = {
  /* If you change this, be sure to update the enum json_type definition too */
  "null",
  "boolean",
  "double",
  "int",
  "object",
  "array",
  "string",
};

const char *json_type_to_name(enum json_type o_type)
{
	int o_type_int = (int)o_type;
	if (o_type_int < 0 || o_type_int >= (int)NELEM(json_type_name))
	{
		MC_ERROR("json_type_to_name: type %d is out of range [0,%d]\n", o_type, NELEM(json_type_name));
		return NULL;
	}
	return json_type_name[o_type];
}
```
从以上代码中可以看出，如果我们需要再添加一个类型的话，至少则需要更改两处地方，一是在enum json_type中添加类型，二是在json_type_name数据中添加类型名称。

#### _delete
_delete为object的私有删除函数，其定义如下
```
typedef void (json_object_private_delete_fn)(struct json_object *o);
```

#### _to_json_string
_to_json_string函数功能则是
```
typedef int (json_object_to_json_string_fn)(struct json_object *jso,
											struct printbuf *pb,
											int level,
											int flags);
```

#### _ref_count
_ref_count 为object的引用计数器，每引用一些object时，object中的_ref_count都会加1，每当使用Putobject时，_ref_count则减1，当_ref_count的值为0时，object则会调用自身的free函数将其所占资源释放掉。

#### _pb
_pb
```
struct printbuf {
  char *buf;
  int bpos;
  int size;
};
struct printbuf* printbuf_new(void);
int printbuf_memappend(struct printbuf *p, const char *buf, int size);
int printbuf_memset(struct printbuf *pb, int offset, int charvalue, int len);
int sprintbuf(struct printbuf *p, const char *msg, ...);
void printbuf_reset(struct printbuf *p);
void printbuf_free(struct printbuf *p);
```
#### data
data为object实际存储的值，如int，bool，double等
```
union data {
		json_bool c_boolean;
		double c_double;
		int64_t c_int64;
		struct lh_table *c_object;
		struct array_list *c_array;
		struct {
			char *str;
			int len;
		} c_string;
```
##### c_object
c_object为Json中的key-value对，其实现是以hash table的方式来存储
```
struct lh_table {
	/**
	 * Size of our hash.
	 */
	int size;
	/**
	 * Numbers of entries.
	 */
	int count;

	/**
	 * Number of collisions.
	 */
	int collisions;

	/**
	 * Number of resizes.
	 */
	int resizes;

	/**
	 * Number of lookups.
	 */
	int lookups;

	/**
	 * Number of inserts.
	 */
	int inserts;

	/**
	 * Number of deletes.
	 */
	int deletes;

	/**
	 * Name of the hash table.
	 */
	const char *name;

	/**
	 * The first entry.
	 */
	struct lh_entry *head;

	/**
	 * The last entry.
	 */
	struct lh_entry *tail;

	struct lh_entry *table;

	/**
	 * A pointer onto the function responsible for freeing an entry.
	 */
	lh_entry_free_fn *free_fn;
	lh_hash_fn *hash_fn;
	lh_equal_fn *equal_fn;
};
``` 

###### hash函数定义
```
typedef void (lh_entry_free_fn) (struct lh_entry *e);
typedef unsigned long (lh_hash_fn) (const void *k);
typedef int (lh_equal_fn) (const void *k1, const void *k2);
```
```
struct lh_entry {
	/**
	 * The key.
	 */
	void *k;
	/**
	 * The value.
	 */
	const void *v;
	/**
	 * The next entry
	 */
	struct lh_entry *next;
	/**
	 * The previous entry.
	 */
	struct lh_entry *prev;
};
```
##### c_array
c_array为果断中数据结构，其结构内容如下:
```
typedef void (array_list_free_fn) (void *data);

struct array_list
{
  void **array;
  int length;
  int size;
  array_list_free_fn *free_fn;
};
```
###### array_list 常用操作函数
```
struct array_list* array_list_new(array_list_free_fn *free_fn);
void array_list_free(struct array_list *arr);
void* array_list_get_idx(struct array_list *arr, int i);
int array_list_put_idx(struct array_list *arr, int idx, void *data);
int array_list_add(struct array_list *arr, void *data);
void array_list_sort(struct array_list *arr, int(*sort_fn)(const void *, const void *));
int array_list_length(struct array_list *arr);

```

##### c_string
json中存储字符串数据的结构
```c
struct {
	char *str;
	int len;
} c_string;
```

#### _user_delete;
```
typedef void (json_object_delete_fn)(struct json_object *jso, void *userdata);
```
#### _userdata;
