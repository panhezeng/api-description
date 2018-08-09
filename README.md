# api-description

##  api blueprint 学习笔记

api blueprint采用渐进增强原则，这点比swagger好，极简写法可以没有任何冗余的样板代码

1. 比如Simplest API

    1. 一个井号跟着 Action关键字 自定义的Resource URI
    2. 一个加号跟着 Response关键字 状态码
    3. 这六组字符是必须有的，是一个API描述的最低要求

    ```markdown
    # GET /message
    
    + Response 200
    ```


2. 如果一个Resource下面有多个Action，则渐进增强，出现两个井号

    1. 一个井号跟着 自定义的Resource URI
    2. 该Resource下的所有Action都跟在下面
    3. 两个井号跟着 Action关键字
    4. 一个加号跟着 Request关键字 (Header Content-Type)
    5. 一个加号跟着 Response关键字 状态码

    ```markdown
    # /message
    
    ## GET
    
    + Response 200
    
    ## POST
    
    + Request (text/plain)
    
    + Response 200
    ```


3. 如果要给Resource Action命名, 则渐进增强，出现中括号

    1. 井号后跟着 自定义名字，原来的Resource Action顺延跟着，并且用中括号括起来

    ```markdown
    # My Message [/message]
    
    ## Retrieve a Message [GET]
    
    + Response 200
    ```


4. 如果要给Resource分组，则渐进增强，出现Group关键字和三个井号

    1. 一个井号跟着 Group关键字 自定义组名
    2. 该组下的所有Resource都跟在下面
    3. 注意和命名示例的区别，就是层级增加了，原来的Resource Action都增加了一个井号

    ```markdown
    # Group Messages
    
    ## My Message [/message]
    
    ### Retrieve a Message [GET]
    
    + Response 200
    ```


5. 如果需要进一步定制Response的Headers和Body，则渐进增强，出现Headers和Body关键字

    1. 一个加号跟着 Response关键字 状态码 (Header Content-Type)
    2. 一个加号跟着 Headers关键字 下面跟着Headers设置，

    ```markdown
    + Response 200 (application/json)
    
        + Headers
    
                X-My-Message-Header: 42
    
        + Body
    
                { "message": "Hello World!" }
    ```


6. 如果需要定制Resource的Parameters参数，则渐进增强，出现Parameters关键字

    1. 路径参数

        1. 斜杆后跟着花括号括起来的参数名
        2. Parameters关键字下面的多种写法，参数默认都是必填项，并且没有默认值
            1. 一个加号+跟着 参数名 冒号: 默认值 (类型) - 描述
            2. 一个加号+跟着 参数名 (类型) 下面跟着 描述

        ```markdown
        ## My Message [/message/{id}]
        
        + Parameters
        ```
        接着上面Markdownd代码
        
        ```markdown
        
            + id: 1 (number) - An unique identifier of the message.
        ```
        或者
        ```markdown
        
            + id (number) 
             
             An unique identifier of the message.
        ```

    2. 查询参数

        1. 路径名跟着花括号括起来的问号?参数名
        2. Parameters关键字下面，一个加号+跟着 参数名 (类型, 是否必须) - 描述，再下面跟着 一个加号 Default关键字冒号: 默认值
 
        ```markdown
        ## All My Messages [/messages{?limit,offset}]
        
        ### Retrieve all Messages [GET]
        
        + Parameters
        
            + limit (number, optional) - The maximum number of results to return.
            
                + Default: `20`
         
            + offset: 0 (number)
        ```


7. 如果想定义一个对象，可以作为任意Request或Response的Body的值，则渐进增强，出现Attributes关键字

    1. 第一个出现的+ Attributes (object)是定义一个Attributes对象，对象名就是往上查找到的第一个Resource名，这里是My Message
    2. 第二个出现的+ Attributes (My Message)是使用前面定义的Attributes对象作为Retrieve a Message的Response Body
    3. 第三个出现的+ Attributes (array[My Message])是定义一个基于My Message Attributes对象的Attributes对象，对象名就是往上查找到的第一个Resource名，这里是All My Messages
    4. 第四个出现的+ Attributes (All My Messages)是使用前面定义的Attributes对象作为Retrieve all Messages的Response Body

    ```markdown
    ## My Message [/message/{id}]
    
    + Parameters
    
        + id (number) 
        
    + Attributes (object)
        + id: 250FF (string, required)
        + created: 1415203908 (number) - Created detail description Time stamp
        + type: 1 (number)
    
             Type detail description 
             
    ### Retrieve a Message [GET]
    
    + Response 200 (application/json)
        + Attributes (My Message)
        
    ## All My Messages [/messages{?limit}]
    
    + Attributes (array[My Message])
    
    ### Retrieve all Messages [GET]
    
    + Parameters
    
        + limit (number, optional) 
        
            The maximum number of results to return.
        
            + Default: `20`
    
    + Response 200 (application/json)
        + Attributes (All My Messages)
    ```


8. 如果需要单独在一个地方集中编写可以用于作为Attributes对象初始数据的数据对象，则渐进增强，出现Data Structures关键字

    1. 一个井号#跟着 Data Structures关键字，下面跟着 两个井号# 跟着对象名 (类型) 
    2. 使用数据结构对象 + Attributes (Message Base)
    3. 其实一个Attributes对象就可以作为另一个Attributes对象的初始数据，但是Attributes不能想Data Structures一样，脱离资源Resource编写，不能自定义名字等

    ```markdown
    # Data Structures
    
    ## Message Base (object)
        + id: 250FF (string, required)
        + created: 1415203908 (number) - Created detail description Time stamp
        
    ## My Message [/message/{id}]
    
    + Parameters
    
        + id (number) 
        
    + Attributes (Message Base)
        + type: 1 (number)
    ```


9. 如果还想定义一个对象，可以作为Request或Response的值，则渐进增强，出现Model关键字

    1. 一个加号+跟着 Model (Header Content-Type) 定义Model对象，对象名就是往上查找到的第一个Resource名，这里是My Message
    2. 在Request, Response关键字下面跟着 [Model名][] 使用此Model

    ```markdown
    ## My Message [/message]
    
    + Model (application/json)
    
        This is the `application/json` message resource representation.
    
        + Headers
    
                X-My-Message-Header: 42
    
        + Body
    
                { "message": "Hello World!" }
    
    ### Retrieve a Message [GET]
    
    + Response 200
    
        [My Message][]
    ```


10. 高级简写法

    1. Named Endpoints
           
    - 实现思路, 省掉原资源Resource名，使用原Action名，并且跟着采用[Action关键字 自定义的Resource URI]写法
             
    ```markdown
    # Group Messages
    
    ## Retrieve a Message [GET /message/{id}]
    
    + Response 200
    ```           

    2. Advanced Action
    
    - 实现思路，省掉Group关键字，把Group组名作为第一个资源Resource的名字，后面其他同组资源Resource，使用Named Endpoints方法

    ```markdown
    # Messages [/messages{?limit}]

    + Parameters
    
        + limit (number, optional) - The maximum number of results to return.
        
            + Default: `20`
    
    ## Retrieve all Messages [GET]

    + Response 200

    ## Retrieve a Message [GET /message/{id}]
    
    + Parameters
    
        + id (number)

    + Response 200   
    ```
    
    

## 总结

Group, Resource, Action, Data Structures前面都要写井号#，用井号数量表示层级关系

命名Resource, Action后，自定义的Resource URI和Action关键字必须用中括号[]括起来

Parameters, Request, Response, Headers, Body, Attributes, Model关键字，以及Parameters下面的参数名, Attributes下面的属性名前面都要写加号+

Request, Response关键字后面跟着的小括号内容，是设置Header Content-Type的一种快捷写法，和下面的+ Headers下面设置Content-Type二选一，只能存其一

Request关键字后面必须有小括号括起来的Content-Type或者下面有Headers或Body定义

Response关键字后面必须有状态码

Parameters关键字可以出现在Resource或者Action下面

Attributes对象定义在Request, Response关键字下面，会自动作为其Body值


换行要求

最好都空一行，井号#或+号下面紧跟的对上面定义的描述可以不空行

Headers Body关键字和其设置之间必须空一行

缩进要求

对于加号+关键字下面的内容，一般都是缩进一个tab，个别情况要求两个tab，比如Headers和Body下面的设置


