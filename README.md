# api-description

[api blueprint examples](https://apiblueprint.org/documentation/examples/)

##  api blueprint 学习笔记

api blueprint 采用渐进增强原则，这点比 swagger 好，极简写法可以没有任何冗余的样板代码

1. 比如 Simplest API，以下内容是一个最简化 API 文档描述的全部要求

    1. FORMAT 关键字: 版本号
    2. 一个井号跟着 API 文档名字
    3. 一个井号跟着 Action 关键字 自定义的 Resource URI
    4. 一个加号跟着 Response 关键字 状态码

    ```markdown
    FORMAT: 1A
 
    # The Simplest API   
 
    # GET /message
    
    + Response 200
    ```
   
2. 如果一个 Resource 下面有多个 Action，则渐进增强，出现两个井号

    1. 一个井号跟着 自定义的 Resource URI
    2. 该 Resource 下的所有 Action 都跟在下面
    3. 两个井号跟着 Action 关键字
    4. 一个加号跟着 Request 关键字 (Header Content-Type)
    5. 一个加号跟着 Response 关键字 状态码

    ```markdown
    # /message
    
    ## GET
    
    + Response 200
    
    ## POST
    
    + Request (text/plain)
    
    + Response 200
    ```

3. 如果要给 Resource URI Action 命名, 则渐进增强，出现中括号

    1. 井号后跟着 自定义名字，再跟着原来的 Resource URI ，并用中括号括起来
    2. 井号后跟着 自定义名字，再跟着原来的 Resource Action ，并用中括号括起来

    ```markdown
    # Message [/message]
    
    ## Retrieve Message [GET]
    
    + Response 200
    ```

4. 如果要给 Resource 分组，则渐进增强，出现 Group 关键字和三个井号

    1. 一个井号跟着 Group关键字 自定义组名，该组下的所有 Resource 都跟在下面，注意和上面3示例的区别，就是层级增加了，原来的 Resource URI Action 都增加了一个井号

    ```markdown
    # Group Messages
    
    ## Message [/message]
    
    ### Retrieve Message [GET]
    
    + Response 200
    ```

5. 如果需要进一步定制 Response 的 Headers 和 Body，则渐进增强，出现 Headers 和 Body 关键字

    1. 一个加号跟着 Response 关键字 状态码 (Header Content-Type)
    2. 紧挨上面不要空行，并且相对上面的缩进一个 tab 后，一个加号跟着 Headers 关键字 下面跟着 Headers 设置
    3. 空一行，并且相对上面的缩进两个 tab 后，写上 Headers 属性

    ```markdown
    + Response 200 (application/json)
        + Headers
    
                X-My-Message-Header: X
    
        + Body
    
                { "message": "Hello World!" }
    ```

6. 如果需要定制 Resource 的 Parameters 参数，则渐进增强，出现 Parameters 关键字

    1. 路径参数

        1. 斜杆后跟着花括号括起来的参数名
        2. Parameters 关键字下面的多种写法，路径参数都是必填项，并且没有默认值
            1. 加号 参数名 冒号 默认值 (类型) - 描述
            2. 加号 参数名 (类型) 下面跟着 描述

        ```markdown
        ## Message [/message/{id}]
        
        + Parameters
        ```
        接着上面 Markdownd 代码
        
        ```markdown
            + id: 1 (number) - An unique identifier of the message.
        ```
        或者
        ```markdown
           + id (number) 
               An unique identifier of the message.
        ```
       
    2. 查询参数

        1. 路径名跟着花括号括起来的问号?参数名，多个参数用逗号分隔，不能有空格
        2. Parameters关键字下面，加号 参数名 (类型, 是否必须) - 描述，再下面跟着 加号 Default 关键字 冒号 默认值
 
        ```markdown
        ## Messages Collection [/messages{?limit,offset}]

        + Parameters
            + limit (number, optional) - The maximum number of results to return.
                + Default: `20`
            + offset: 0 (number)
        
        ### Retrieve Messages[GET]
        ```

7. 如果想定义一个对象，可以作为任意Request或Response的Body的值，则渐进增强，出现Attributes关键字

    1. 第一个出现的+ Attributes (object)是定义一个Attributes对象，对象名就是往上查找到的第一个Resource名，这里是Message
    2. 第二个出现的+ Attributes (Message)是使用前面定义的Attributes对象作为Retrieve Message的Response Body
    3. 第三个出现的+ Attributes (array[Message])是定义一个基于Message Attributes对象的Attributes对象，对象名就是往上查找到的第一个Resource名，这里是Messages Collection
    4. 第四个出现的+ Attributes (Messages Collection)是使用前面定义的Attributes对象作为Retrieve Messages Collection的Response Body

    ```markdown
    ## Message [/message/{id}]
    
    + Parameters
        + id (number) 
        
    + Attributes (object)
        + id: 250FF (string, required)
        + created: 1415203908 (number) - Created detail description Time stamp
        + type: 1 (number)
             Type detail description 
             
    ### Retrieve Message [GET]
    
    + Response 200 (application/json)
        + Attributes (Message)
        
    ## Messages Collection [/messages{?limit}]
 
    + Parameters
        + limit (number, optional) 
            The maximum number of results to return.
            + Default: `20`
    
    + Attributes (array[Message, Message])
    
    ### Retrieve Messages[GET]
       
    + Response 200 (application/json)
        + Attributes (Messages Collection)
    ```

8. 如果需要单独在一个地方集中编写可以用于作为Attributes对象初始数据的数据对象，则渐进增强，出现Data Structures关键字

    1. 一个井号#跟着 Data Structures关键字，下面跟着 两个井号# 跟着对象名 (类型) 
    2. 使用数据结构对象 + Attributes (Messages)
    3. 其实一个Attributes对象本身就可以作为另一个Attributes对象的初始数据，但是Attributes必须依附于Resource，使用Resource的命名

    ```markdown
    FORMAT: 1A
    
    # The Simplest API 
    
    # Group Messages
        
    ## Messages Collection [/messages{?limit}]
    
    ### Retrieve Messages[GET]
    
    + Response 200 (application/json)
        + Attributes (Messages)
        
    # Data Structures
    
    ## Message (object)
    + id: 250FF (string, required)
    + created: 1415203908 (number) - Created detail description Time stamp
    + type: 1 (number)
    
    ## Messages
    + count: 100 (number)
    + data (array[Message])
    ```

9. 如果还想定义一个对象，可以作为Request或Response的值，则渐进增强，出现Model关键字

    1. 一个加号+跟着 Model (Header Content-Type) 定义Model对象，对象名就是往上查找到的第一个Resource名，这里是Message
    2. 在Request, Response关键字下面跟着 [Model名][] 使用此Model

    ```apib
    ## Message [/message]
    
    + Model (application/json)
        This is the `application/json` message resource representation.
        + Headers
    
                X-My-Message-Header: 42
    
        + Body
    
                { "message": "Hello World!" }
    
    ### Retrieve Message [GET]
    
    + Response 200
    
        [Message][]
    ```

10. 高级简写法

    1. Named Endpoints
           
    - 实现思路, 省掉原资源Resource名，使用原Action名，并且跟着采用[Action关键字 自定义的Resource URI]写法
             
    ```markdown
    # Group Messages
    
    ## Retrieve Message [GET /message/{id}]
    
    + Response 200
    ```           

    2. Advanced Action
    
    - 实现思路，省掉Group关键字，把Group组名作为第一个资源Resource的名字，后面其他同组资源Resource，使用Named Endpoints方法

    ```markdown
    # Messages [/messages{?limit}]

    + Parameters
        + limit (number, optional) - The maximum number of results to return.
            + Default: `20`
    
    ## Retrieve Messages[GET]

    + Response 200

    ## Retrieve Message [GET /message/{id}]
    
    + Parameters
        + id (number)

    + Response 200   
    ```
    
    

## 总结

Group, Resource, Action, Data Structures前面都要写井号#，用井号数量表示层级关系

命名Resource, Action后，自定义的Resource URI和Action关键字必须用中括号[]括起来

Parameters, Request, Response, Headers, Body, Attributes, Model关键字，以及Parameters下面的参数名, Attributes下面的属性名前面都要写加号+

Request, Response关键字后面跟着的小括号内容，是设置Header Content-Type的一种快捷写法，和下面的+ Headers下面设置Content-Type二选一，只能存其一

Request关键字后面必须有小括号括起来的Content-Type或者下面的Headers或Body的部分有定义

Response关键字后面必须有状态码

Parameters关键字可以出现在Resource或者Action下面

Attributes对象定义在Request, Response关键字下面，会自动作为其Body值

如要复用Attributes和Model对象，Attributes和Model对象必须定义在命名Resource下面，然后通过Attributes (Resource)或[Resource][]的方式复用，Named Endpoints下定义的Attributes和Model对象，无法复用

缩进和换行要求

Headers和Body的设置，包括Request, Response，Model关键字下面直接写的，和Header，Body关键字下面写的，相对上面的关键字，必须缩进两个tab和空一行

井号不用缩进，井号下面第一层级的加号也不用缩进，其他层级加号通常要求缩进一个tab



