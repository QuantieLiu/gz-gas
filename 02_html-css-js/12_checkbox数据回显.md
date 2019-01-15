
<li>前端获取数据字典，js拼接

```
						<input name="roleCode" type="hidden" />
							<tr>
								<th><span class="text-red">*</span>岗位职责：</th>
								<td id="position"></td>
							</tr>

		//需要展示在页面的是哪几个角色code
		var roleCode = '${roleCode}';
    //多选框
		var roleDic = ${roleDic};
    
		//拼装角色编码，处理admin同时分配工业食品计量的情况
			function makePositionHad(){
				if(roleDic.length == 1){
				$("#save_form  [name='roleCode']").val(roleDic[0].id);
					$("#position").text(roleDic[0].text);
				}else if(roleDic.length >1){
					var html=document.getElementById("position").innerHTML;
					for(var i=0; i<roleDic.length; i++){
						html +='<input type="checkbox"  name="roleChoose" id= "'+ roleDic[i].id +'" value="'+ roleDic[i].id +'" style="width:auto;margin-left:3px;">'+roleDic[i].text+'<br>';
					}
					$("#position").append(html);
				}
			}
```

<li>首选
	
```
					boxObj.each(function () {  
					  debugger
						if( -1 < roleCho.indexOf($(this).val()) ) {  
						   $(this).attr("checked",true);  
						}  
					});  
```

<li>修改页面的数据回显-js

```
					var roleCho=selections[0].roleCode;
					var boxObj = document.getElementsByName("roleChoose");
					for(var i=0;i<boxObj.length;i++){
						if( -1 < roleCho.indexOf(boxObj[i].value) )
							boxObj[i].checked= true;
					}
          //保存form时会自动传到后台
```

<li>jquery两层循环
	
```
					var boxObj = $("input:checkbox[name='roleChoose']");  //获取所有的复选框
					 var express = roleCho.split(',');  
					$.each(express, function(index, expressId){  
					       boxObj.each(function () {  
					    	   debugger
					            if($(this).val() == expressId) {  
					               $(this).attr("checked",true);  
					            }  
					        });  
					    });  
```
