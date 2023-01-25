# [Boost.JSON Boost的JSON解析库（1.75首发）](https://www.cnblogs.com/ink19/p/Boost_JSON.html)

## 目录[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E7%9B%AE%E5%BD%95)

- [目录](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E7%9B%AE%E5%BD%95)
- [Boost的1.75版本新库](https://www.cnblogs.com/ink19/p/Boost_JSON.html#boost%E7%9A%84175%E7%89%88%E6%9C%AC%E6%96%B0%E5%BA%93)
- [JSON库简介](https://www.cnblogs.com/ink19/p/Boost_JSON.html#json%E5%BA%93%E7%AE%80%E4%BB%8B)
- [JSON的简单使用](https://www.cnblogs.com/ink19/p/Boost_JSON.html#json%E7%9A%84%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8)
    - [编码](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E7%BC%96%E7%A0%81)
        - [最通用的方法](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E6%9C%80%E9%80%9A%E7%94%A8%E7%9A%84%E6%96%B9%E6%B3%95)
        - [使用`std::initializer_list`](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E4%BD%BF%E7%94%A8stdinitializer_list)
        - [json对象的输出](https://www.cnblogs.com/ink19/p/Boost_JSON.html#json%E5%AF%B9%E8%B1%A1%E7%9A%84%E8%BE%93%E5%87%BA)
        - [两种对比](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E4%B8%A4%E7%A7%8D%E5%AF%B9%E6%AF%94)
    - [解码](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E8%A7%A3%E7%A0%81)
        - [简单的解码](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E7%AE%80%E5%8D%95%E7%9A%84%E8%A7%A3%E7%A0%81)
        - [增加错误处理](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%A2%9E%E5%8A%A0%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86)
        - [非严格模式](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E9%9D%9E%E4%B8%A5%E6%A0%BC%E6%A8%A1%E5%BC%8F)
        - [流输入](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E6%B5%81%E8%BE%93%E5%85%A5)
- [进阶应用](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E8%BF%9B%E9%98%B6%E5%BA%94%E7%94%A8)
    - [对象序列化](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%AF%B9%E8%B1%A1%E5%BA%8F%E5%88%97%E5%8C%96)
    - [反序列化](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96)
    - [`Boost.JSON`的类型](https://www.cnblogs.com/ink19/p/Boost_JSON.html#boostjson%E7%9A%84%E7%B1%BB%E5%9E%8B)
        - [array](https://www.cnblogs.com/ink19/p/Boost_JSON.html#array)
        - [object](https://www.cnblogs.com/ink19/p/Boost_JSON.html#object)
        - [string](https://www.cnblogs.com/ink19/p/Boost_JSON.html#string)
        - [value](https://www.cnblogs.com/ink19/p/Boost_JSON.html#value)
- [总结](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E6%80%BB%E7%BB%93)
- [引用](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%BC%95%E7%94%A8)

## Boost的1.75版本新库[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#boost%E7%9A%84175%E7%89%88%E6%9C%AC%E6%96%B0%E5%BA%93)

12月11日，Boost社区发布了1.75版本，相比较于原定的12月9日，推迟了两天。这次更新带来了三个新库：`JSON`，`LEAF`，`PFR`。

其中`JSON`自然是json格式的解析库，来自`Vinnie Falco`和`Krystian Stasiowski`。

`LEAF`是一个轻量的异常处理库，来自`Emil Dotchevski`。

`PFR`是一个基础的反射库，不需要用户使用宏和样版代码（由于还未仔细阅读此库，可能翻译有一些不准确），来自`Antony Polukhin`。

## JSON库简介[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#json%E5%BA%93%E7%AE%80%E4%BB%8B)

其实在之前，`Boost`就已经有能够解析JSON的库了，名字叫做`Boost.PropertyTree`。`Boost.PropertyTree`不仅仅能够解析`JSON`，还能解析`XML`，`INI`和`INFO`格式的文件。但是由于成文较早及需要兼容其他的数据格式，相比较于其他的`C++`解析库，其显得比较笨重，使用的时候有很多的不方便。

`Boost.JSON`相对于`Boost.PropertyTree`来所，其只能支持`JSON`格式的解析，但是其使用方法更为简便，直接。华丽胡哨的东西也更多了。

## JSON的简单使用[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#json%E7%9A%84%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8)

有两种方法使用`Boost.JSON`，一种是动态链接库，此时引入头文件`boost/json.hpp`，同时链接对应的动态库；第二种是使用header only模式，此时只需要引入头文件`boost/json/src.hpp`即可。两种方法各有优缺点，酌情使用。

### 编码[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E7%BC%96%E7%A0%81)

#### 最通用的方法[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E6%9C%80%E9%80%9A%E7%94%A8%E7%9A%84%E6%96%B9%E6%B3%95)

我们要构造的json如下，包含了各种类型。

```json
{
    "a_string" : "test_string",
    "a_number" : 123,
    "a_null"   : null,
    "a_array"  : [1, "2", {"123" : "123"}],
    "a_object" : {
        "a_name": "a_data"
    },
    "a_bool"   : true
}
```

构造的方法也很简单：

```C++
boost::json::object val;
val["a_string"] = "test_string";
val["a_number"] = 123;
val["a_null"] = nullptr;
val["a_array"] = {
    1, "2", boost::json::object({{"123", "123"}})
};
val["a_object"].emplace_object()["a_name"] = "a_data";
val["a_bool"] = true;
```

首先定义一个`object`，然后往里面塞东西就好。其中有一个`emplace_object`这个比较重要，后面会提到。

结果：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219210900343-91590508.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219210900343-91590508.png)

#### 使用`std::initializer_list`[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E4%BD%BF%E7%94%A8stdinitializer_list)

`Boost.JSON`支持使用`std::initializer_list`来构造自己的对象。所以也可以这样使用：

```C++
boost::json::value val2 = {
    {"a_string", "test_string"},
    {"a_number", 123},
    {"a_null", nullptr},
    {"a_array", {1, "2", {{"123", "123"}}}},
    {"a_object", {{"a_name", "a_data"}}},
    {"a_bool", true}
};
```

结果如下：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219210935628-186020789.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219210935628-186020789.png)

#### json对象的输出[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#json%E5%AF%B9%E8%B1%A1%E7%9A%84%E8%BE%93%E5%87%BA)

生成了`json`对象以后，就可以使用`serialize`对对象进行序列化了。

```C++
std::cout << boost::json::serialize(val2) << std::endl;
```

结果如前两图。

除了直接把整个对象直接输出，`Boost.JSON`还支持分部分进行流输出，这种方法在数据量较大时，可以有效降低内存占用。

```C++
boost::json::serializer ser;
ser.reset(&val);

char temp_buff[6];
while (!ser.done()) {
    std::memset(temp_buff, 0, sizeof(char) * 6);
    ser.read(temp_buff, 5);
    std::cout << temp_buff << std::endl;
}
```

结果：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219210958851-1229465610.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219210958851-1229465610.png)

如果缓存变量是数组，还可以直接使用`ser.read(temp_buff)`。

需要注意的是，`ser.read`并不会默认在字符串末尾加`\0`，所以如果需要直接输出，在输入时对缓存置0，同时为`\0`空余一个字符。

也可以直接使用输出的`boost::string_view`。

#### 两种对比[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E4%B8%A4%E7%A7%8D%E5%AF%B9%E6%AF%94)

这两种方法对比的话，各有各的优点。前一种方法比较时候边运行边生成，后者适合一开始就需要直接生成的情形，而且相对来说，后者显得比较的直观。

但是第二种方法有一个容易出现问题的地方。比如以下两个`json`对象：

```json
// json1
[["data", "value"]]

//json2
{"data": "value"}
```

如果使用第二种方法进行构建，如果一不小心的话，就有可能写出一样的代码：

```C++
boost::json::value confused_json1 = {{"data", "value"}};
    
boost::json::value confused_json2 = {{"data", "value"}};

std::cout << "confused_json1: " << boost::json::serialize(confused_json1) << std::endl;
std::cout << "confused_json2: " << boost::json::serialize(confused_json2) << std::endl;
```

而得到的结果，自然也是一样的：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211010192-115924265.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211010192-115924265.png)

如果需要消除这一歧义，可以直接使用`Boost.JSON`提供的对象构建有可能产生歧义的地方：

```C++
boost::json::value no_confused_json1 = {boost::json::array({"data", "value"})};
boost::json::value no_confused_json2 = boost::json::object({{"data", "value"}});
```

结果为：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211037544-1028703067.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211037544-1028703067.png)

### 解码[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E8%A7%A3%E7%A0%81)

`JSON`的解码也比较简单。

#### 简单的解码[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E7%AE%80%E5%8D%95%E7%9A%84%E8%A7%A3%E7%A0%81)

```C++
auto decode_val = boost::json::parse("{\"123\": [1, 2, 3]}");
```

直接使用`boost::json::parse`，输入相应的字符串就行了。

#### 增加错误处理[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%A2%9E%E5%8A%A0%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86)

```C++
boost::json::error_code ec;
boost::json::parse("{\"123\": [1, 2, 3]}", ec);
std::cout << ec.message() << std::endl;

boost::json::parse("{\"123\": [1, 2, 3}", ec);
std::cout << ec.message() << std::endl;
```

结果：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211053398-1718286523.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211053398-1718286523.png)

#### 非严格模式[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E9%9D%9E%E4%B8%A5%E6%A0%BC%E6%A8%A1%E5%BC%8F)

在这个模式下，`Boost.JSON`可以选择性的对一些不那么严重的错误进行忽略。

```C++
unsigned char buf[4096];
boost::json::static_resource mr(buf);
boost::json::parse_options opt;
opt.allow_comments = true;          // 允许注释
opt.allow_trailing_commas = true;   // 允许最后的逗号
boost::json::parse("[1, 2, 3, ] // comment test", ec, &mr, opt);
std::cout << ec.message() << std::endl;
boost::json::parse("[1, 2, 3, ] // comment test", ec, &mr);
std::cout << ec.message() << std::endl;
```

结果如下：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211115269-224341345.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211115269-224341345.png)

可以看到，增加了选项的解释器成功的解析了结果。

#### 流输入[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E6%B5%81%E8%BE%93%E5%85%A5)

和输出一样，输入也有流模式。

```C++
boost::json::stream_parser p;
p.reset();

p.write("[1, 2,");
p.write("3]");
p.finish();

std::cout << boost::json::serialize(p.release()) << std::endl;
```

结果：  
[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211128349-1433092488.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211128349-1433092488.png)

## 进阶应用[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E8%BF%9B%E9%98%B6%E5%BA%94%E7%94%A8)

### 对象序列化[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%AF%B9%E8%B1%A1%E5%BA%8F%E5%88%97%E5%8C%96)

有时候我们需要将对象转换为`JSON`，对对象进行序列化然后保存。`Boost.JSON`提供了一个非常简单的方法，能够使我们非常简单的将一个我们自己定义的对象转化为`JSON`对象。

我们只需要在需要序列化的类的命名空间中，定义一个重载函数`tag_invoke`。注意，是类所在的命名空间，而不是在类里面定义。

使用示例：

```C++
namespace MyNameSpace {
class MyClass {
public:
    int a;
    int b;
    MyClass (int a = 0, int b = 1):
    a(a), b(b) {}
};

void tag_invoke(boost::json::value_from_tag, boost::json::value &jv, MyClass const &c) {
    auto & jo = jv.emplace_object();
    jo["a"] = c.a;
    jo["b"] = c.b;
}
}
```

其中，`boost::json::value_from_tag`是作为标签存在的，方便`Boost.JSON`分辨序列化函数的。`jv`是输出的`JSON`对象，`c`是输入的对象。

```C++
boost::json::value_from(MyObj)
```

使用的话，直接调用`value_from`函数即可。

结果：

[![](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211142199-634862451.png)](https://img2020.cnblogs.com/blog/2105008/202012/2105008-20201219211142199-634862451.png)

序列化还有一个好处就是，可以在使用`std::initializer_list`初始化`JSON`对象时，直接使用自定义对象。譬如：

```C++
boost::json::value val = {MyObj};
```

注意，这里的`val`是一个数组，里面包含了一个对象`MyObj`。

### 反序列化[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96)

有序列化，自然就会有反序列化。操作和序列化的方法差不多，也是定义一个`tag_invoke`函数，不过其参数并不一致。

```C++
MyClass tag_invoke(boost::json::value_to_tag<MyClass>, boost::json::value const &jv) {
    auto &jo = jv.as_object();
    return MyClass(jo.at("a").as_int64(), jo.at("b").as_int64());
}
```

需要注意的是，由于传入的`jv`是被`const`修饰的，所以不能类似于`jv["a"]`使用。

使用也和上面的类似，提供了一个`value_to<>`模板函数。

```C++
auto MyObj = boost::json::value_to<MyNameSpace::MyClass>(vj);
```

无论是序列化还是反序列化，对于标准库中的容器，`Boost.JSON`都可以直接使用。

### `Boost.JSON`的类型[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#boostjson%E7%9A%84%E7%B1%BB%E5%9E%8B)

#### array[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#array)

数组类型，用于储存`JSON`中的数组。实际使用的时候类似于`std::vector<boost::json::value>`，差异极小。

#### object[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#object)

对象类型，用于储存`JSON`中的对象。实际使用时类似于`std::map<std::string, boost::json::value>`，但是相对来说，它们之间的差异较大。

#### string[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#string)

字符串类型，用于储存`JSON`中的字符串。实际使用时和`std::basic_string`类似，不过其只支持`UTF-8`编码，如果需要支持其他编码，在解码时候需要修改option中相应的选项。

#### value[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#value)

可以储存任意类型，也可以变换为各种类型。其中有一些特色的函数比如`as_object`，`get_array`，`emplace_int64`之类的。它们的工作都类似，将`boost::json::value`对象转化为对应的类型。但是他们之间也有一定的区别。

- `as_xxx` 返回一个引用，如果类型不符合，会抛出异常
- `get_xxx` 返回一个引用，不检查类型，如果类型不符合，可能导致未定义行为
- `is_xxx` 判断是否为`xxx`类型
- `if_xxx` 返回指针，如果类型不匹配则返回`nullptr`
- `emplace_xxx` 返回一个引用，可以直接改变其类型和内容。

## 总结[#](https://www.cnblogs.com/ink19/p/Boost_JSON.html#%E6%80%BB%E7%BB%93)

大致的使用方法就这些了。如果还要更进一步的话，就是涉及到其内存管理了。

纵观整个库的话，感觉其对于模板的使用相当克制，能不使用就不使用，这在一定程度上也提高了编译的速度。