 
##### 通过getParameter逐个get字段 
 
``` 
 request.setCharacterEncoding("UTF-8");
 //获取传过来的表单数据,根据表单中的name获取所填写的值
 String pwd = request.getParameter("pwd");
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


