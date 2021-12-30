# 命令行下使用markdown和plantuml生成对应文档

## plantuml 安装

### download

至[plantuml 官网](https://plantuml.com/zh/download)中下载最新的的plantuml.jar包。

### plantuml launcher

/usr/bin/plantuml

```shell
#!/bin/bash
if [ -n "${JAVA_HOME}" ] && [ -x "${JAVA_HOME}/bin/java" ] ; then
    JAVA="${JAVA_HOME}/bin/java"
elif [ -x /usr/bin/java ] ; then
    JAVA=/usr/bin/java
else
    echo "Can't find the java."
    exit 1
fi

$JAVA -jar /usr/share/plantuml/plantuml.jar ${@}

```

## 工具及插件安装

```shell
pip3 install markdown_py plantuml_markdown
```

## 使用

```shell
markdown_py --noisy -x plantuml_markdown test1.md -f test1.html
```

## 常见问题

使用markdown_py直接报错，建议直接升级plantuml。跟进python源码后，发现是plantuml版本过低引起的。建议升至最新。

另还可能需要安装GraphViz工具，这里就不介绍了。


## 参考

[plantuml cmdline](https://plantuml.com/zh/command-line)
[plantuml-markdown](https://www.cnpython.com/pypi/plantuml-markdown)



```plantuml
a->b
b->c
c->d
```

```puml
a->b
b->c
c->d
```


```uml
a->d
```

```{uml}
c->d
```