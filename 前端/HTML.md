# HTML

- ## HTML基本结构

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  <meta charset="utf-8"><!-- -->
  <title>这是我的第一个HTML</title>
  </head>
  <body>
      <h1>我的第一个标题</h1>
      <p>我的第一个段落。</p>
  </body>
  </html>
  ```
  
- html基本标签
  
  - div标签：独占一行
  
  - span标签：
  
    <span> 用于对文档中的行内元素进行组合。
  
    <span> 标签没有固定的格式表现。当对它应用样式时，它才会产生视觉上的变化。如果不对 <span> 应用样式，那么 <span> 元素中的文本与其他文本不会任何视觉上的差异。
  
    <span> 标签提供了一种将文本的一部分或者文档的一部分独立出来的方式。
  
  - <h></h>
    
    - HTML 标题（Heading）是通过<h1> - <h6> 标签来定义的。
    
  - HTML 段落
  
    HTML 段落是通过标签 <p> 来定义的。
  
    ```html
    <p></p>
    ```
  
  - HTML 链接是通过标签 <a> 来定义的。
  
    ```html
    实例
    <a href="https://www.runoob.com">这是一个链接</a>
    ```
    
    - target属性：
    
      - _self(当前页面跳转)
    
      - _blank(另开页面跳转)
    
        ```html
        <a href="http://www.bilibili.com" target="_blank">hello,html</a>
        ```
    
        
    
  -   HTML 图像
    
    HTML 图像是通过标签 <img> 来定义的.
    
    ```html
    <img src="Capture001.png" width="10%" height="10%" border="1" alt="图片加载失败"/>
    
    border为相框
    alt为如果图片找不到
    相对路径：
    .代表当前文件所在目录
    ..上一级目录
    文件名 表示当前文件所在目录的文件（相当于./）
    
    绝对路径
    http://ip:port/工程名/资源路径
    
    ```
    
  -   front标签(字体标签)
    
    ```html
    <front color="green" face="宋体" size="7"></front>
    ```
    
  -   转义字符
    
    | HTML 原代码 | 显示结果 | 描述                   |
    | ----------- | -------- | ---------------------- |
    | &lt;        | <        | 小于号或显示标记       |
    | &gt;        | >        | 大于号或显示标记       |
    | &amp;       | &        | 可用于显示其它特殊字符 |
    | &quot;      | “        | 引号                   |
    | &reg;       | ®        | 已注册                 |
    | &copy;      | ©        | 版权                   |
    | &trade;     | ™        | 商标                   |
    | (&ensp);    |          | 半个空白位(括号里的)   |
    | (&emsp);    |          | 一个空白位(括号里的)   |
    | (&nbsp);    |          | 不断行的空白(括号里的) |
    
  - 列表
    
    - 无序列表（常用）
    
    ```html
    <u>
        <li>wwww</li>
        <li>aaaa</li>
        <li>rrrr</li>
        <li>mmmm</li>
        <li>aaaa</li>
    </u>
    ```
    
    <u>表示无序列表，<li>表示列表项
    
    - 有序标签
    
  - 表格标签
    
    - table为表格标签（有几个table就有几个表）
    - tr为行标签（有几个tr表格就有几行）
    - th为表头标签（有几个th每行就有几列，相比于td只是多了加粗和居中）
    - td为普通表内容
    
    - 属性设置：width设置宽，height设置高，align设置对齐方式，cellspacing设置单元格间距。
    
    - b为加粗符
    
      ```html
      <table width="10%" height="100%" border="2" align="center" cellspacing="1">
          <tr>
              <th colspan="2">boom</th>
              <td>boom????????</td>
      
      
              <th rowspan="2">boom</th>
              <th>boom</th>
          </tr>
      
          <tr>
              <th>boom</th>
              <th>boom</th>
              <th>boom</th>
          </tr>
          <tr>
              <th>boom</th>
              <th>boom</th>
              <th>boom</th>
              <th>boom</th>
          </tr>
      
      </table>
      ```
    
    - colspan实现单元格跨列，rowspan实现单元格跨行
    
  -   iframe标签（内嵌窗口）
    
    - 在页面中开辟一个小窗口用于显示指定的页面
    
    - 常常与a标签联合使用
    
      ```html
      <iframe src="http://www.bilibili.com" width="70%" height="1000" border="10"name="web">
          </iframe>
      <br>
      <br>
      <a href="http://www.bilibili.com" target="web">去B站</a>
      <a href="http://www.lolqq.com" target="web">去lol</a>
      <a href="htmltext.html" target="web">去自己的页面</a>
      ```
    
  -   ## 表单
    
  -   form标签为表单标签
    
    - input type="text" 文本输入框，value设置默认显示内容
    - input type="password" 密码输入框（输入的内容会被隐藏） value设置默认显示内容
    - input type="radio" 单选框 name属性可以对其分组（同一组的只能选一个） 属性：checked="checked"为选项默认选中
    - input type="checkbox" 复选框 属性：check="check"为选项默认选中
    - input type="reset" 重置按钮 重置回默认
    - intput type="submit" 提交按钮 value修改按钮上的文本
    - input type="button" 按钮 value修改按钮上的文本
    - input type="file" 文件上传域
    - input type="hidden" 隐藏域 某些信息的发送不需要用户参与。
    
    
    
    
    
    - select标签 下拉列表 option标签在select中，表示选项，使用selected="selected"可以设置默认选中
    - textarea表示多行文本输入框（起始标签和结束标签中的内容是默认内容）
      - rows为设置行高度
      - cols为设置字符宽度
    
    ```html
    <form>
        <input type="text" value="请输入用户名"><br>
        输入密码：
        <input type="password"><br>
        确认密码
        南<input type="radio" name="sex" checked="checked">&nbsp;
        铝<input type="radio" name="sex" ><br>
        兴趣爱好:
        导<input type="checkbox"  checked="checked">&nbsp;
        南<input type="checkbox" >&nbsp;
        铝<input type="checkbox" ><br>
        请选择国家
        <select>
            <option >--请选择国家--</option>
            <option selected="selected">中国</option>
            <option>美国</option>
            <option>日本</option>
            <option>法国</option>
        </select><br>
    
        <textarea cols="10" rows="10">
            fuckyou
        </textarea><br>
    
        <input type="submit" value="提交"><br>
        <input type="reset" value="重置"><br>
    
    
    </form>
    ```
    
    - 表单经常和表格联合使用(更加工整)
    
      ```html
      <table>
          <tr>
              <th>输入用户名:</th>
              <td><input type="text" value="请输入用户名"></td>
          </tr>
      
          <tr>
              <th> 输入密码：</th>
              <td><input type="password"><br></td>
          </tr>
      
          <tr>
              <th>
                  确认密码
              </th>
              <td>
                  南<input type="radio" name="sex" checked="checked">&nbsp;
                  铝<input type="radio" name="sex" ><br>
              </td>
          </tr>
          
          <tr>
              <th>
                  兴趣爱好:
              </th>
              <td>
                  导<input type="checkbox"  checked="checked">&nbsp;
                  南<input type="checkbox" >&nbsp;
                  铝<input type="checkbox" ><br>
              </td>
          </tr>
      
      
          <tr>
              <th>
                  请选择国家
              </th>
              <td>
                  <select>
                      <option >--请选择国家--</option>
                      <option selected="selected">中国</option>
                      <option>美国</option>
                      <option>日本</option>
                      <option>法国</option>
                  </select><br>
              </td>
          </tr>
      
          <tr>
              <td>
                  <b> 你有什么大病</b>
              </td>
              <td>
                  <textarea cols="100" rows="100">
                      fuckyou
                  </textarea><br>
              </td>
          </tr>
      
      
          <tr>
              <th>
                      提交资料&nbsp;
              </th>
              <td>
                  <input type="submit" value="提交">
              </td>
              <th>
                  重置资料&nbsp;
              </th>
              <td>
                  <input type="reset" value="重置"><br>
              </td>
          </tr>
      </table>
      ```
    
  -   表单提交
    
    - 表单提交需要先在form中指定action值（发送数据的目的地（网址）），method决定提交方式（默认为get）
    
    - 提交方式有两种
    
      - GET
    
        - 提交格式(问号为分隔符)
    
          ```html
          http://localhost:8080/
          ? 
          username=%E7%94%A8%E6%88%B7%E5%90%8D
          &
          userpasswo=password
          &
          sex=%E5%8D%97
          &
          hobby=%E5%AF%BC
          &
          country=%E4%B8%AD%E5%9B%BD
          &
          %E6%9C%89%E4%BB%80%E4%B9%88%E5%A4%A7%E7%97%85=++++++++++++++++fuckyou%0D%0A++++++++++++
          ```
    
        - 不安全（例如密码在提交时会显现出来）
        - 存在提交的字长限制
    
      - POST
    
        - 提交格式
    
          ```html
          http://localhost:8080/
          ```
    
        - 无提交字长
    
        - 安全（提价时只有action的值）
    
    - 表单提交时数据没有发送成功的三种情况
    
      - 表单提交项没有设置name
    
      - 单选，多选，下拉列表没有设置value值（默认发送on或off）
    
      - 表单项不在form中
    
        
  
-         案例设计（一个可以提交的登录网站）
  
  ```html
  <form action="http://localhost:8080" method="post">
  <table align="center">
      <tr>
          <th>输入用户名:</th>
          <td><input type="text"  name="username" value="用户名"></td>
      </tr>
  
      <tr>
          <th> 输入密码：</th>
          <td><input name="userpasswo"  value="password" type="password"><br></td>
      </tr>
  
      <tr>
          <th>
              确认密码
          </th>
          <td>
              南<input type="radio" name="sex" checked="checked" value="南">&nbsp;
              铝<input type="radio" name="sex" value="铝"><br>
          </td>
      </tr>
  
      <tr>
          <th>
              兴趣爱好:
          </th>
          <td>
              导<input type="checkbox" name="hobby" value="导" checked="checked">&nbsp;
              南<input type="checkbox" name="hobby" value="南">&nbsp;
              铝<input type="checkbox" name="hobby" value="铝"><br>
          </td>
      </tr>
  
  
      <tr>
          <th>
              请选择国家
          </th>
          <td>
              <select name="country" >
                  <option >--请选择国家--</option>
                  <option selected="selected">中国</option>
                  <option>美国</option>
                  <option>日本</option>
                  <option>法国</option>
              </select><br>
          </td>
      </tr>
  
      <tr>
          <td>
              <b> 你有什么大病</b>
          </td>
          <td>
              <textarea cols="10%" rows="10%" name="有什么大病">
                  fuckyou
              </textarea><br>
          </td>
      </tr>
  
  
      <tr>
          <th>
                  提交资料&nbsp;
          </th>
          <td>
              <input type="submit" value="提交">
          </td>
          <th>
              重置资料&nbsp;
          </th>
          <td>
              <input type="reset" value="重置"><br>
          </td>
      </tr>
  </table>
  </form>
  ```
  
  ​      
  
  ​      
  
    
  
    
  
    
  
    
  
    
  
  
  

