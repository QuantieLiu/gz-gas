#### ajax提交

##### 前端拼接json，使用bean来接收json数据
 
``` 
<script type="text/javascript">
    $(document).ready(function(){
        $("#button_submit").click(function(){
            var name = $("#userName").val();
            var pass = $("#password").val();
            
            var user = {userName:name,password:pass};//拼装成json格式
       
            $.ajax({
                type:"POST",
                url:"${pageContext.request.contextPath}/user/addUser4",
                data:user,
                success:function(data){
                    alert("成功");
                },
                error:function(e) {
                    alert("出错："+e);
                }
            });
        });
    });
</script>

@RequestMapping("/addUser4")
    public String addUser4(User user) {
        System.out.println("userName is:"+user.getUserName());
        System.out.println("password is:"+user.getPassword());
        return "/user/success";
    }
``` 

##### jQuery的serializeArray() 方法序列化表单元素

``` 
<script type="text/javascript">
    $(document).ready(function(){
        $("#button_submit").click(function(){
            
            //序列化表单元素，返回json数据
            var params = $("#userForm").serializeArray();
            
            //也可以把表单之外的元素按照name value的格式存进来
            //params.push({name:"hello",value:"man"});
            
            $.ajax({
                type:"POST",
                url:"${pageContext.request.contextPath}/user/addUser5",
                data:params,
                success:function(data){
                    alert("成功");
                },
                error:function(e) {
                    alert("出错："+e);
                }
            });
        });
    });
</script>
``` 

#### 基于form提交

##### 通过getParameter逐个get字段 
 
``` 
 request.setCharacterEncoding("UTF-8");
 //获取传过来的表单数据,根据表单中的name获取所填写的值
 String pwd = request.getParameter("pwd");
``` 

##### 框架封装，业务action通过custForm取值

``` 
在抽象类aseSaveStrutsAction
	/**
	 * 将request中参数值设置到form中（按form中属性与参数名称对应）
	 * @return
	 */
	protected Map<String, Object> setCustFormProperties(){
		return BeanUtils.setProperties(custForm, ServletActionContext.getRequest().getParameterMap());
	}
```

##### apache 的 BeanUtils 工具来进行封装数据
 
``` 
@WebServlet("/RequestDemo3")
public class RequestDemo3 extends HttpServlet {
    private static final long serialVersionUID = 1L;
    
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        //获取传过来的表单数据,根据表单中的name获取所填写的值
    
        //方法四：使用beanUtil来封装User类
        User u = new User();
        System.out.println("没有使用BeanUtil封装之前：  "+u);
        try {
            BeanUtils.populate(u, request.getParameterMap());
            System.out.println("使用BeanUtils封装之后：  "+u);
        } catch (IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
        }
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
 
``` 


##### 通过getParameterNames获取form的所有字段
 
``` 
package com.chensi;
import java.io.IOException;
import java.util.Enumeration;
import java.util.Iterator;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/RequestDemo3")
public class RequestDemo3 extends HttpServlet {
    private static final long serialVersionUID = 1L;
       
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        //获取传过来的表单数据,根据表单中的name获取所填写的值
        Enumeration<String> names = request.getParameterNames();
        while (names.hasMoreElements()) {
            String strings = (String) names.nextElement();
            String[] parameterValues = request.getParameterValues(strings);
            for (int i = 0;parameterValues!=null&&i < parameterValues.length; i++) {
                System.out.println(strings+":"+parameterValues[i]+"\t");
            }
        }
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

}
``` 


