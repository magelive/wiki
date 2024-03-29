# 嵌入式编程规范~

copy from [老外的嵌入式编程规范](https://mp.weixin.qq.com/s/chtfkHK_7FpJTuerwg37-g)

### 最重要的规则
  编写代码时最重要的一条规则是：检查周围的代码并尝试模仿它。

  作为维护人员，如果收到的补丁明显与周围代码的编码风格不同，这是令人沮丧的。这是不尊重人的，就像某人穿着泥泞的鞋子走进一间一尘不染的房子。

  因此，无论本文推荐的是什么，如果已经编写了代码并且您正在对其进行修补，请保持其当前的样式一致，即使它不是您最喜欢的样式。

### 一般性的规则
  这里列出了最明显和最重要的一般规则。在你继续阅读其他章节之前，请仔细检查它们。

* 使用C99标准

* 不使用制表符，而是使用空格

* 每个缩进级别使用4个空格

* 在关键字和左括号之间使用一个空格

* 在函数名和左括号之间不要使用空格
```c
int32_t a = sum(4, 3);              /* OK */
int32_t a = sum (4, 3);             /* Wrong */
```

* 不要在变量/函数/宏/类型中使用_或前缀。这是为C语言本身保留的

* 对于严格的模块私有函数，使用prv_ name前缀

* 对于包含下划线_ char的变量/函数/宏/类型，只能使用小写字母

* 左花括号总是与关键字(for, while, do, switch, if，…)在同一行
```c
size_t i;
for (i = 0; i < 5; ++i) {           /* OK */
}
for (i = 0; i < 5; ++i){            /* Wrong */
}
for (i = 0; i < 5; ++i)             /* Wrong */
{
}
```

* 在比较操作符和赋值操作符之前和之后使用单个空格
```c
int32_t a;
a = 3 + 4;              /* OK */
for (a = 0; a < 5; ++a) /* OK */
a=3+4;                  /* Wrong */
a = 3+4;                /* Wrong */
for (a=0;a<5;++a)       /* Wrong */
每个逗号后用单空格
func_name(5, 4);        /* OK */
func_name(4,3);         /* Wrong */
```

* 不要初始化静态和全局变量为0(或NULL)，让编译器为您做
```c
static int32_t a;       /* OK */
static int32_t b = 4;   /* OK */
static int32_t a = 0;   /* Wrong */

void my_func(void) {
    static int32_t* ptr;/* OK */
    static char abc = 0;/* Wrong */
}
```

* 在同一行中声明所有相同类型的局部变量
```c
void my_func(void) {
    char a;             /* OK */
    char b;             /* Wrong, variable with char type already exists */
    char a, b;          /* OK */
}
```

* 按顺序声明局部变量
    * 自定义结构和枚举
    * 整数类型，更宽的无符号类型优先
    * 单/双浮点
```c
int my_func(void) {
    /* 1 */
    my_struct_t my;     /* First custom structures */
    my_struct_ptr_t* p; /* Pointers too */

    /* 2 */
    uint32_t a;
    int32_t b;
    uint16_t c;
    int16_t g;
    char h;
    /* ... */

    /* 3 */
    double d;
    float f;
}
```

* 总是在块的开头声明局部变量，在第一个可执行语句之前

* 在for循环中声明计数器变量
```c
/* OK */
for (size_t i = 0; i < 10; ++i)

/* OK, if you need counter variable later */
size_t i;
for (i = 0; i < 10; ++i) {
    if (...) {
        break;
    }
}
if (i * 10) {

}

/* Wrong */
size_t i;
for (i = 0; i < 10; ++i) ...
```

* 避免在声明中使用函数调用来赋值变量，除了单个变量
```c
void a(void) {
    /* Avoid function calls when declaring variable */
    int32_t a, b = sum(1, 2);

    /* Use this */
    int32_t a, b;
    b = sum(1, 2);

    /* This is ok */
    uint8_t a = 3, b = 4;
}
```

*  除了`char`、`float`或`double`之外，始终使用`stdint.h`标准库中声明的类型。例如，8位的`uint8_t`等

* 不要使用`stdbool.h`库。分别使用`1`或`0`表示真或假
```c
/* OK */
uint8_t status;
status = 0;

/* Wrong */
#include <stdbool.h>
bool status = true;
```

* 永远不要与真实相比较。例如，使用`if(check_func()){…}`替换`if (check_func() * 1)`

* 总是将指针与空值进行比较
```c
void* ptr;

/* ... */

/* OK, compare against NULL */
if (ptr * NULL || ptr != NULL) {

}

/* Wrong */
if (ptr || !ptr) {

}
```

* 总是使用前增量(和递减)，而不是后增量(和递减)
```c
int32_t a = 0;
...

a++;            /* Wrong */
++a;            /* OK */

for (size_t j = 0; j < 10; ++j) {}  /* OK */
```

* 总是使用size_t作为长度或大小变量

* 如果函数不应该修改指针所指向的内存，则总是使用const作为指针

* 如果不应该修改函数的形参或变量，则总是使用const
```c
/* When d could be modified, data pointed to by d could not be modified */
void
my_func(const void* d) {

}

/* When d and data pointed to by d both could not be modified */
void
my_func(const void* const d) {

}

/* Not required, it is advised */
void
my_func(const size_t len) {

}

/* When d should not be modified inside function, only data pointed to by d could be modified */
void
my_func(void* const d) {

}
```

* 当函数可以接受任何类型的指针时，总是使用void *，不要使用uint8_t *。函数在实现时必须注意正确的类型转换
```c
/*
 * To send data, function should not modify memory pointed to by `data` variable
 * thus `const` keyword is important
 *
 * To send generic data (or to write them to file)
 * any type may be passed for data,
 * thus use `void *`
 */
/* OK example */
void send_data(const void* data, size_t len) { /* OK */
    /* Do not cast `void *` or `const void *` */
    const uint8_t* d = data;/* Function handles proper type for internal usage */
}

void send_data(const void* data, int len) {    /* Wrong, not not use int */
}
```

* 总是使用括号和sizeof操作符

* 不要使用变长数组。使用动态内存分配代替标准C malloc和自由函数，或者如果库/项目提供了自定义内存分配，使用它的实现看看LwMEM，一个自定义内存管理库。
```c
/* OK */
#include <stdlib.h>
void
my_func(size_t size) {
    int32_t* arr;
    arr = malloc(sizeof(*arr) * n); /* OK, Allocate memory */
    arr = malloc(sizeof *arr * n);  /* Wrong, brackets for sizeof operator are missing */
    if (arr * NULL) {
        /* FAIL, no memory */
    }

    free(arr);  /* Free memory after usage */
}

/* Wrong */
void
my_func(size_t size) {
    int32_t arr[size];  /* Wrong, do not use VLA */
}
```

* 总是将variable与0进行比较，除非它被视为布尔类型

* 永远不要将布尔处理的变量与0或1进行比较。用NOT(!)代替
```c
size_t length = 5;  /* Counter variable */
uint8_t is_ok = 0;  /* Boolean-treated variable */
if (length)         /* Wrong, length is not treated as boolean */
if (length > 0)     /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */
if (length * 0)    /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */

if (is_ok)          /* OK, variable is treated as boolean */
if (!is_ok)         /* OK, -||- */
if (is_ok * 1)     /* Wrong, never compare boolean variable against 1! */
if (is_ok * 0)     /* Wrong, use ! for negative check */
```

* 对于注释，总是使用/* comment */，即使是单行注释

* 在头文件中总是包含带有extern关键字的c++检查

* 每个函数都必须包含doxygen-enabled注释，即使函数是静态的

* 使用英文名称/文本的函数，变量，注释

* 变量使用小写字母

* 如果变量包含多个名称，请使用下划线。force_redraw。不要使用forceRedraw

* 对于C标准库的包含文件，请始终使用<和>。例如，# include < stdlib.h >

* 对于自定义库，请始终使用""。例如，# include“my_library.h”

* 当转换为指针类型时，总是将星号与类型对齐，例如。uint8_t* t = (uint8_t*)var_width_diff_type

* 始终尊重项目或库中已经使用的代码风格

### 注释相关的规则

* 不允许以`//`开头的注释。总是使用`/* comment */`，即使是单行注释

* 对于多行注释，每行使用`空格+星号`
```c
/*
 * This is multi-line comments,
 * written in 2 lines (ok)
 */

/**
 * Wrong, use double-asterisk only for doxygen documentation
 */

/*
* Single line comment without space before asterisk (wrong)
*/

/*
 * Single line comment in multi-line configuration (wrong)
 */

/* Single line comment (ok) */
```

* 注释时使用12个缩进(12 * 4个空格)偏移量。如果语句大于12个缩进，将注释4-空格对齐(下面的例子)到下一个可用缩进
```c
void my_func(void) {
    char a, b;

    a = call_func_returning_char_a(a);          /* This is comment with 12*4 spaces indent from beginning of line */
    b = call_func_returning_char_a_but_func_name_is_very_long(a);   /* This is comment, aligned to 4-spaces indent */
}
```

### 函数定义的规则

* 每个可以从模块外部访问的函数都必须包含函数原型(或声明)

* 函数名必须小写，可以用下划线_分隔。（ 这个应该尽量根据项目规则来定，有些使用小写+下划线，有些使用驼峰）
```c
/* OK */
void my_func(void);
void myfunc(void);

/* OK */
void myFunc();

/* Wrong */
void my_Func(void);
void MYFunc(void);
```

* 当函数返回指针时，将星号对齐到返回类型
```c
/* OK */
const char* my_func(void);
my_struct_t* my_func(int32_t a, int32_t b);

/* Wrong */
const char *my_func(void);
my_struct_t * my_func(void);
```

* 对齐所有的功能原型(使用相同/相似的功能)以提高可读性
```c
/* OK, function names aligned */
void        set(int32_t a);
my_type_t   get(void);
my_ptr_t*   get_ptr(void);

/* Wrong */
void set(int32_t a);
const char * get(void);
```

* 函数实现必须在单独的行中包含返回类型和可选的其他关键字
```c
/* OK */
int32_t
foo(void) {
    return 0;
}

/* OK */
static const char*
get_string(void) {
    return "Hello world!\r\n";
}

/* Wrong */
int32_t foo(void) {
    return 0;
}
```

### 变量相关的规则

* 使变量名全部小写，下划线_字符可选(根据项目规则来定)
```c
/* OK */
int32_t a;
int32_t my_var;
int32_t myvar;

/* Wrong */
int32_t A;
int32_t myVar;
int32_t MYVar;
```

* 按类型将局部变量分组在一起
```c
void foo(void) {
    int32_t a, b;   /* OK */
    char a;
    char b;         /* Wrong, char type already exists */
}
```

* 不要在第一个可执行语句之后声明变量
```c
void foo(void) {
    int32_t a;
    a = bar();
    int32_t b;      /* Wrong, there is already executable statement */
}
```

* 你可以在下一个缩进级别中声明新的变量
```c
int32_t a, b;
a = foo();
if (a) {
    int32_t c, d;   /* OK, c and d are in if-statement scope */
    c = foo();
    int32_t e;      /* Wrong, there was already executable statement inside block */
}
```

* 用星号声明指针变量与类型对齐
```c
/* OK */
char* a;

/* Wrong */
char *a;
char * a;
```

* 当声明多个指针变量时，可以使用星号对变量名进行声明
```c
/* OK */
char *p, *n;
```

### 结构、枚举类型定义

* 结构名或枚举名必须小写，单词之间有下划线_字符

* 结构或枚举可以包含typedef关键字

* 所有结构成员都必须小写

* 所有枚举成员必须是大写的

* 结构/枚举必须遵循doxygen文档语法

* 在声明结构体时，它可以使用以下三种不同的选项之一:
    1. 当结构体仅用名称声明时，它的名称后不能包含_t后缀。
    ```c
    struct struct_name {
        char* a;
        char b;
    };
    ```
    2. 当只使用typedef声明结构时，它的名称后面必须包含_t后缀。
    ```c
    typedef struct {
        char* a;
        char b;
    } struct_name_t;
    ```
    3. 当结构用name和typedef声明时，它不能包含t作为基本名称，它必须在它的名称后面包含t后缀作为typedef部分。
    ```c
    typedef struct struct_name {
        char* a;
        char b;
        char c;
    } struct_name_t;
    ```

* 错误声明的例子及其建议的纠正：
```c
/* a and b must be separated to 2 lines */
/* Name of structure with typedef must include _t suffix */
typedef struct {
    int32_t a, b;
} a;

/* Corrected version */
typedef struct {
    int32_t a;
    int32_t b;
} a_t;

/* Wrong name, it must not include _t suffix */
struct name_t {
    int32_t a;
    int32_t b;
};

/* Wrong parameters, must be all uppercase */
typedef enum {
    MY_ENUM_TESTA,
    my_enum_testb,
} my_enum_t;
```

* 在声明时初始化结构时，使用C99初始化风格
```c
/* OK */
a_t a = {
    .a = 4,
    .b = 5,
};

/* Wrong */
a_t a = {1, 2};
```

* 当为函数句柄引入new typedef时，使用_fn后缀
```c
/* Function accepts 2 parameters and returns uint8_t */
/* Name of typedef has `_fn` suffix */
typedef uint8_t (*my_func_typedef_fn)(uint8_t p1, const char* p2);
```

### 复合语句规则

* 每个复合语句必须包括左花括号和右花括号，即使它只包含1个嵌套语句

* 每个复合语句必须包含单个缩进;嵌套语句时，每个嵌套包含1个缩进大小
```c
/* OK */
if (c) {
    do_a();
} else {
    do_b();
}

/* Wrong */
if (c)
    do_a();
else
    do_b();

/* Wrong */
if (c) do_a();
else do_b();
```

* 在if或if-else-if语句的情况下，else必须与第一条语句的右括号在同一行
```c
/* OK */
if (a) {

} else if (b) {

} else {

}

/* Wrong */
if (a) {

}
else {

}

/* Wrong */
if (a) {

}
else
{

}
```

* 在do-while语句的情况下，while部分必须与do部分的右括号在同一行
```c
/* OK */
do {
    int32_t a;
    a = do_a();
    do_b(a);
} while (check());

/* Wrong */
do
{
/* ... */
} while (check());

/* Wrong */
do {
/* ... */
}
while (check());
```

* 每一个开括号都需要缩进
```c
if (a) {
    do_a();
} else {
    do_b();
    if (c) {
        do_c();
    }
}
```

* 不要做没有花括号的复合语句，即使是单个语句。下面的例子展示了一些不好的做法
```c
if (a) do_b();
else do_c();

if (a) do_a(); else do_b();
```

* 空while循环、do-while循环或for循环必须包含花括号
```c
/* OK */
while (is_register_bit_set()) {}

/* Wrong */
while (is_register_bit_set());
while (is_register_bit_set()) { }
while (is_register_bit_set()) {
}
```

* 如果while(或for、do-while等)为空(嵌入式编程中也可能是这种情况)，请使用空的单行括号
```c
/* Wait for bit to be set in embedded hardware unit
uint32_t* addr = HW_PERIPH_REGISTER_ADDR;

/* Wait bit 13 to be ready */
while (*addr & (1 << 13)) {}        /* OK, empty loop contains no spaces inside curly brackets */
while (*addr & (1 << 13)) { }       /* Wrong */
while (*addr & (1 << 13)) {         /* Wrong */

}
while (*addr & (1 << 13));          /* Wrong, curly brackets are missing. Can lead to compiler warnings or unintentional bugs */
```

* 尽量避免在循环块内递增变量，参见示例
```c
/* Not recommended */
int32_t a = 0;
while (a < 10) {
    .
    ..
    ...
    ++a;
}

/* Better */
for (size_t a = 0; a < 10; ++a) {

}

/* Better, if inc may not happen in every cycle */
for (size_t a = 0; a < 10; ) {
    if (...) {
        ++a;
    }
}
```

### 分支语句规则

* 为每个case语句添加单个缩进

* 使用额外的单缩进break语句在每个case或default
```c
/* OK, every case has single indent */
/* OK, every break has additional indent */
switch (check()) {
    case 0:
        do_a();
        break;
    case 1:
        do_b();
        break;
    default:
        break;
}

/* Wrong, case indent missing */
switch (check()) {
case 0:
    do_a();
    break;
case 1:
    do_b();
    break;
default:
    break;
}

/* Wrong */
switch (check()) {
    case 0:
        do_a();
    break;      /* Wrong, break must have indent as it is under case */
    case 1:
    do_b();     /* Wrong, indent under case is missing */
    break;
    default:
        break;
}
```

* 总是包含default语句
```c
/* OK */
switch (var) {
    case 0:
        do_job();
        break;
    default: break;
}

/* Wrong, default is missing */
switch (var) {
    case 0:
        do_job();
        break;
}
```

* 如果需要局部变量，则使用花括号并在里面放入break语句。将左花括号放在case语句的同一行
```c
switch (a) {
    /* OK */
    case 0: {
        int32_t a, b;
        char c;
        a = 5;
        /* ... */
        break;
    }

    /* Wrong */
    case 1:
    {
        int32_t a;
        break;
    }

    /* Wrong, break shall be inside */
    case 2: {
        int32_t a;
    }
    break;
}
```

### 宏和预处理指令

* 总是使用宏而不是文字常量，特别是对于数字

* 所有的宏必须是全大写的，并带有下划线_字符(可选)，除非它们被明确标记为function，将来可能会被常规函数语法替换
```c
/* OK */
#define MY_MACRO(x)         ((x) * (x))

/* Wrong */
#define square(x)           ((x) * (x))
```

* 总是用圆括号保护输入参数
```c
/* OK */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))

/* Wrong */
#define MIN(x, y)           x < y ? x : y
```

* 总是用括号保护最终的宏计算
```c
/* Wrong */
#define MIN(x, y)           (x) < (y) ? (x) : (y)
#define SUM(x, y)           (x) + (y)

/* Imagine result of this equation using wrong SUM implementation */
int32_t x = 5 * SUM(3, 4);  /* Expected result is 5 * 7 = 35 */
int32_t x = 5 * (3) + (4);  /* It is evaluated to this, final result = 19 which is not what we expect */

/* Correct implementation */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))
#define SUM(x, y)           ((x) + (y))
```

* 当宏使用多个语句时，使用do-while(0)语句保护它
```c
typedef struct {
    int32_t px, py;
} point_t;
point_t p;                  /* Define new point */

/* Wrong implementation */

/* Define macro to set point */
#define SET_POINT(p, x, y)  (p)->px = (x); (p)->py = (y)    /* 2 statements. Last one should not implement semicolon */

SET_POINT(&p, 3, 4);        /* Set point to position 3, 4. This evaluates to... */
(&p)->px = (3); (&p)->py = (4); /* ... to this. In this example this is not a problem. */

/* Consider this ugly code, however it is valid by C standard (not recommended) */
if (a)                      /* If a is true */
    if (b)                  /* If b is true */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    else
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */

/* Evaluates to code below. Do you see the problem? */
if (a)
    if (b)
        (&p)->px = (3); (&p)->py = (4);
    else
        (&p)->px = (5); (&p)->py = (6);

/* Or if we rewrite it a little */
if (a)
    if (b)
        (&p)->px = (3);
        (&p)->py = (4);
    else
        (&p)->px = (5);
        (&p)->py = (6);

/*
 * Ask yourself a question: To which `if` statement `else` keyword belongs?
 *
 * Based on first part of code, answer is straight-forward. To inner `if` statement when we check `b` condition
 * Actual answer: Compilation error as `else` belongs nowhere
 */

/* Better and correct implementation of macro */
#define SET_POINT(p, x, y)  do { (p)->px = (x); (p)->py = (y); } while (0)    /* 2 statements. No semicolon after while loop */
/* Or even better */
#define SET_POINT(p, x, y)  do {    \   /* Backslash indicates statement continues in new line */
    (p)->px = (x);                  \
    (p)->py = (y);                  \
} while (0)                             /* 2 statements. No semicolon after while loop */

/* Now original code evaluates to */
if (a)
    if (b)
        do { (&p)->px = (3); (&p)->py = (4); } while (0);
    else
        do { (&p)->px = (5); (&p)->py = (6); } while (0);

/* Every part of `if` or `else` contains only `1` inner statement (do-while), hence this is valid evaluation */

/* To make code perfect, use brackets for every if-ifelse-else statements */
if (a) {                    /* If a is true */
    if (b) {                /* If b is true */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    } else {
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */
    }
}
```

* 不缩进子语句内#if语句
```c
/* OK */
#if defined(XYZ)
#if defined(ABC)
/* do when ABC defined */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* Do when XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
    #if defined(ABC)
        /* do when ABC defined */
    #endif /* defined(ABC) */
#else /* defined(XYZ) */
    /* Do when XYZ not defined */
#endif /* !defined(XYZ) */
```

### 文档

* 文档化的代码允许doxygen解析和通用的html/pdf/latex输出，因此正确地执行是非常重要的。

* 对变量、函数和结构/枚举使用doxygen支持的文档样式

* 经常使用\作为doxygen，不要使用@

* 始终使用5x4空格(5个制表符)作为文本行开始的偏移量
```c
/**
 * \brief           Holds pointer to first entry in linked list
 *                  Beginning of this text is 5 tabs (20 spaces) from beginning of line
 */
static
type_t* list;
```

* 每个结构/枚举成员都必须包含文档

* 注释的开头使用12x4空格偏移量
```c
/**
 * \brief           This is point struct
 * \note            This structure is used to calculate all point
 *                      related stuff
 */
typedef struct {
    int32_t x;                                  /*!< Point X coordinate */
    int32_t y;                                  /*!< Point Y coordinate */
    int32_t size;                               /*!< Point size.
                                                    Since comment is very big,
                                                    you may go to next line */
} point_t;

/**
 * \brief           Point color enumeration
 */
typedef enum {
    COLOR_RED,                                  /*!< Red color. This comment has 12x4
                                                    spaces offset from beginning of line */
    COLOR_GREEN,                                /*!< Green color */
    COLOR_BLUE,                                 /*!< Blue color */
} point_color_t;
```

* 函数的文档必须在函数实现中编写(通常是源文件)

* 函数必须包括简要和所有参数文档

* 如果每个参数分别为in或out输入和输出，则必须注意

* 如果函数返回某个值，则必须包含返回形参。这不适用于void函数

* 函数可以包含其他doxygen关键字，如note或warning

* 在参数名和描述之间使用冒号:
```c
/**
 * \brief           Sum `2` numbers
 * \param[in]       a: First number
 * \param[in]       b: Second number
 * \return          Sum of input values
 */
int32_t
sum(int32_t a, int32_t b) {
    return a + b;
}

/**
 * \brief           Sum `2` numbers and write it to pointer
 * \note            This function does not return value, it stores it to pointer instead
 * \param[in]       a: First number
 * \param[in]       b: Second number
 * \param[out]      result: Output variable used to save result
 */
void
void_sum(int32_t a, int32_t b, int32_t* result) {
    *result = a + b;
}
```

* 如果函数返回枚举的成员，则使用ref关键字指定哪个成员
```c
/**
 * \brief           My enumeration
 */
typedef enum {
    MY_ERR,                                     /*!< Error value */
    MY_OK                                       /*!< OK value */
} my_enum_t;

/**
 * \brief           Check some value
 * \return          \ref MY_OK on success, member of \ref my_enum_t otherwise
 */
my_enum_t
check_value(void) {
    return MY_OK;
}
```

* 对常量或数字使用符号(' NULL ' => NULL)
```c
/**
 * \brief           Get data from input array
 * \param[in]       in: Input data
 * \return          Pointer to output data on success, `NULL` otherwise
 */
const void *
get_data(const void* in) {
    return in;
}
```

* 宏的文档必须包括hideinitializer doxygen命令
```c
/**
 * \brief           Get minimal value between `x` and `y`
 * \param[in]       x: First value
 * \param[in]       y: Second value
 * \return          Minimal value between `x` and `y`
 * \hideinitializer
 */
#define MIN(x, y)       ((x) < (y) ? (x) : (y))
```

### 头/源文件

* 在文件末尾留下一个空行

* 每个文件都必须包括文件的doxygen注释和后跟空行的简要描述(使用doxygen时)
```c
/**
 * \file            template.h
 * \brief           Template include file
 */
                    /* Here is empty line */
```

* 每个文件(头文件或源文件)必须包含许可证(开始注释包括单个星号，因为doxygen必须忽略这个)

* 使用与项目/库已经使用的相同的许可证
```c
/**
 * \file            template.h
 * \brief           Template include file
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */
 ```

* 头文件必须包含保护符`#ifndef`

* 头文件必须包含`c++`检查

* 在`c++`检查之外包含外部头文件

* 首先用STL C文件包含外部头文件，然后是应用程序自定义文件

* 头文件必须包含其他所有头文件，以便正确编译，但不能包含更多头文件(如果需要，.c应该包含其余的头文件)

* 头文件必须只公开模块公共变量/类型/函数

* 在头文件中使用extern作为全局模块变量，稍后在源文件中定义它们
```c
/* file.h ... */
#ifndef xxx 
extern int32_t my_variable; /* This is global variable declaration in header */
#endif

/* file.c ... */
int32_t my_variable;        /* Actually defined in source */
```

* 不要把.c文件包含在另一个.c文件中

* .c文件应该首先包含相应的.h文件，然后是其他文件，除非另有明确的必要

* 在头文件中不包含模块私有声明

* 头文件示例(示例中没有license)

```c
/* License comes here */

#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/* Include headers */

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */
/* File content here */
#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif /* TEMPLATE_HDR_H */

```
