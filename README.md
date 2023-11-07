本人手写的PHP API框架，暂未命名
======================================
百行代码实现的极简PHP API框架
---
### Versions 0.0.1:
* 只有一个文件，仅实现路由转发功能
* 自动加载类定义文件、调用控制器、执行指定的方法、序列化输出
* 执行结果以JSON格式输出，控制器的输出无需处理，只需用 return 语句返回即可
* 需要服务器配置路由重写功能，把路由转发给此文件处理
* 例如，访问 http://sample.com/{类名}/{方法名}?{参数} 将会：
* 加载Controllers目录下的{类名}.php文件
* 引用Controllers\\{类名}，所以自定义控制器应有“namespace Controllers;”语句
* 调用{类名}中定义的{方法名}方法，并把{参数}作为调用参数传递给所调用的方法
* {参数}是一个数组，POST发送的数据会合并到{参数}中一起传给方法
* 所调用的方法用 return 语句返回处理结果之后，自动格式化为JSON输出
* 没有指定方法名，默认调用{类名}的Index方法，没有指定{类名}，默认引用Index类
* 用法：
* 在此文件的目录下，自行创建Controllers目录，用于存放自定义的控制器代码
* 控制器的文件名必须和类名相同，且第一个字母必须大写
* 控制器需要引用的其他代码，自行决定放哪里、如何引用

---
### Versions 0.0.2:
* 前一版本对所有的http请求都执行{类名}/{方法名}操作，不是RESTFul API，此版本把各种http请求的操作分开来，实现真正的RESTFul API
* 前一版本的参数是合并在一起传递的，此版本按照请求的不同分别传递不同的参数
* 各种http请求的操作自行实现，对应的方法名是GET => Index, POST => Insert, PUT、PATCH => Update, DELETE => Delete
* 不仅会在Controllers目录寻找控制器文件，还会在API目录下查找文件，如果Controllers目录找不到的话
* 添加权限检查的功能架构，实际的实现代码请自己编写，在自定义类中实现Permissions方法即可
* Permissions方法会接收到两个字符串参数，前一个是JWT字符串，从http头的Authorization字段读取而来，后一个是大写的http请求方法，如GET、POST等。JWT令牌的生成、发送、验证等功能请自行实现
* Permissions方法返回一个 bool 值，true 表示允许访问，false 表示拒绝访问
* 没有实现Permissions方法的话，默认拒绝访问，可以在框架中修改为默认允许
* 如果自定义的类中存在Count方法，在执行GET请求的时候会调用它，此方法一般用于前端的分页功能

### Version 0.0.3:
* 添加配置文件Setting.json，此框架会在运行时读取配置文件，对运行时的各种参数进行配置
* 配置文件的Path字段指示自定义控制器的搜索路径，是一个数组，实现自定义、多路径查找，从此文件存放的位置不再限于Controllers和API了
* 配置文件的NameSpace字段指示类的命名空间，也是一个数组，实现自定义、多命名空间查找、引用类，从此不再限于Controllers命名空间了
* 支持没有命名空间的类名，从此自定义类中没有 namespace 指令的文件也可以正常使用了。当然，建议加上 namespace 指令
* 各种http请求的处理方法也可以自定义了，不再局限于Index、Insert、Update、Delete这些名称，爱哪个名称就用哪个，只需在配置文件中指定就可以了
* 默认的访问权限也不只是呆板的“拒绝”了，可以在配置文件中为各种操作指定访问权限
* 本来Permissions方法是各个自定义类的访问权限的处理方法，不喜欢“Permissions”这个方法名怎么办？可以在配置文件里修改！只要把Permission->option的值改成喜欢的名称即可

### Version 0.0.4:
* 上一个版本中，虽然增加了配置文件以增强程序的灵活性，但是也出现了一个问题：没有配置文件的话，运行就会出错，也就是说，一定需要配置文件配合，那么本质上这个框架就是两个文件。为了解决这个问题，特意修改了此版本对配置文件的依赖。也就是说，这个版本没有配置文件也可以运行，在程序中直接有默认的运行参数可以支持无配置文件运行
* 仍然保留对配置文件的支持，而且如果存在配置文件的话，那么以配置文件的参数配置为准。也就是说，先加载程序本身的默认参数，然后查找配置文件的对应项，如果存在，这替换为配置文件的参数。由于每个参数都需要查找，所以增加的代码量比较多
* 配置文件要注意大小写，所有的键名都是首字母大写的英文单词，请参照给出的示例，如果字母大小写出差错，程序运行不会出错，但是配置项就不起作用了，因为它找不到。同样要注意的是，控制器的类名和方法名也都是首字母大写，否则程序会找不到类和方法。
* 默认情况下，各种http请求方法的处理方法与请求本身的方法同名，但是大小写不一样，程序默认的处理方法名是首字母大写的小写形式，例如，GET请求的默认处理方法是{类名}下的Get方法，同理POST的处理方法是Post，PUT的处理方法是Put，DELETE的处理方法是Delete
* PATCH请求的处理合并到PUT请求，也就是说发送PATCH请求和发送PUT请求，其处理方法是一样的，都是Put
* 请求处理的默认方法仍然可以更改，只需在配置文件文件中指定即可，比如说要把GET请求指定给Select处理，只需要在配置文件的Action节点上设置"Get":"Select"即可。详细示例配置文件。
* 另一个重大改进是对权限管理的改进，首先是程序本身（全部允许访问），其次是配置文件的设定，在然后会查找{类名}下的授权方法，如果存在，就会执行授权方法获得授权结果，然后再根据授权结果决定是否执行相应的处理方法。同样，授权方法的名称在程序中预先进行了定义，叫作"Permission"，而且，这个授权方法的方法名在配置文件中也可以修改，在配置文件的"Permission"节点中，键名为Option
* 注意：在配置文件缺失或者没有配置相应节点的情况下，控制器文件的查找位置是程序文件所在的目录，类名前面也命名空间的限定，有了限定反而找不到。
* 处理了一个隐藏的问题：如果客户端访问的控制器名称与当前的程序文件同名的时候，此时如果在没有配置文件的情况下，程序在当前目录就会搜索到自己，可能会引起无法预测的错误，因此增加一段代码检测{类名}是否与自己同名，如果同名，就修该为"Error"，从而搜索错误处理类。同样的，配置文件可以更改这个错误处理类的名称，如果你不喜欢"Error"，想改为"ILoveU"，没问题！
