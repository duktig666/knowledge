# 配置模板

## 配置Java方法注释

![image-20200707141000177](https://gitee.com/koala010/typora/raw/master/img/image-20200707141000177.png)

```java
/**
 * description:
 $params$
 * @return $return$
 *
 * @author $USER$
 * Date: $DATE$ $TIME$
 */
```

![image-20200707141100345](https://gitee.com/koala010/typora/raw/master/img/image-20200707141100345.png)

params的脚本：

```groovy
groovyScript("
def result=''; 
def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); 
for(i = 0; i < params.size(); i++) {
    result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\n ' : '')
};
return result", methodParameters())methodParameters())
```



快捷键java，即可调用java的JavaDoc注释



## 配置Swagger模板

![image-20200707141438827](https://gitee.com/koala010/typora/raw/master/img/image-20200707141558537.png)

```java
@ApiOperation(value = "", notes = "")
$swaggerParam$
```



![image-20200707141558537](https://gitee.com/koala010/typora/raw/master/img/image-20200707141438827.png)

swaggerParam脚本:

```java
groovyScript("
def result='';
def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); 
def paramStr ='';
for(i = 0; i < params.size(); i++){
	paramStr+='@ApiImplicitParam(name = \"' + params[i] + '\"\,value = \"' + params[i] + '\"\)' + ((i < params.size() - 1) ? '\,\\n        ' : '\\n')
};
result +='' +((params.size()>1)  ? '@ApiImplicitParams({\\n        '+paramStr+'})': ''+paramStr+'')+'';
return result;
",methodParameters());
```



快捷键swf即可生成swagger的接口方法注释

