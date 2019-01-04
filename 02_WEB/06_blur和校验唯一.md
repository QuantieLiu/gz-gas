##### jQuery的serializeArray() 方法序列化表单元素

``` 
				<div class="modal-body cont-table">
					<div class="table-normal">
					<form id="save_form">
						<table class="tb-modal">
						<input name="id" type="hidden" />
						<input name="orgId" type="hidden" />
						<input name="roleCode" type="hidden" />
							<tr>
								<th><span class="text-red">*</span>所属部门：</th>
								<td id="orgs"></td>
								<th><span class="text-red">*</span>登录帐号：</th>
								<td><input type="text" name="logonId" maxlength="20"/></td>
							</tr>
							<tr>
								<th><span class="text-red">*</span>姓名：</th>
								<td><input type="text" name="name" maxlength="20"/></td>
								<th><span class="text-red">*</span>登录密码：</th>
								<td><input type="text" name="pwd" maxlength="20"/></td>
							</tr>
						</table>
						</form>
					</div>
				</div>
``` 

``` 
			//保存事件
			$('#save_btn').click(function(){
				//保存前校验
				var flag=true;
				var arrError=[];
				var logonId = $("#save_form [name='logonId']");
				var name = $("#save_form [name='name']");
				var pwd = $("#save_form [name='pwd']");
				var orgId = $("#save_form [name='orgId']");
				//登录帐号
				if(logonId.val()==""){
					arrError.push(logonId);
					flag=false;
				}
				//姓名
				if(name.val()==""){
					arrError.push(name);
					flag=false;
				}
				//密码
				if(pwd.val()==""){
					arrError.push(pwd);
					flag=false;
				}
				//所属部门
				if(orgId.val()==""){
					arrError.push(orgId);
					flag=false;
				}
				//必填项缺失
				if(!flag){
					var tips="该字段不能为空";
					for(var i = 0; i < arrError.length; i++){
						addErrorInput(arrError[i],tips);
					}
					return flag;
				}
				$("#save_form").attr("action", baseUrl + "save/save.do");
				//表单提交
				BT.submitForm($("#save_form"), function(data) {
					if (data && data.hasOk == true) {
						BT.showSuccess("保存成功!");
					} else {
						BT.showError(data.message);
					}
					window.location.reload();
				});
			});
      
      //input框提醒信息内容和样式
			function addErrorInput(item){
				item.addClass("required error");
				if(item.parent().find('label.error').length>0){
					item.parent().find('label.error').remove();;
				}
				item.after('<label class="error" style="display: inline-block;">'+tips+'</label>');
				item.bind('input propertychange',function(){
					if($(this).val().length==0){
						$(this).addClass('error');
		   				$(this).parent().find('label.error').css('display','inline-block');
		   			}else{
		   				$(this).removeClass('error');
		   				$(this).parent().find('label.error').css('display','none');
		   			}
				});
			}
      
``` 

``` 
	@Override
	public Boolean validLogonIdUnique(String id,String logonId){
		if(StringUtils.isNotBlank(id) && logonId.equals(userManageDao.find(id).getLogonId())){
			//先和自己比较
			return false;
		}
		return userManageDao.getUserInfoByLogonId(logonId);
	}
``` 


``` 
	@SuppressWarnings("unchecked")
	@Override
	public boolean getUserInfoByLogonId(String logonId){
		StringBuffer sb = new StringBuffer(" SELECT info.LOGON_ID FROM sys_user_comminfo info where info.LOGON_ID = ? ");
		List<Object> list = this.getSession().createSQLQuery(sb.toString()).setParameter(0, logonId).list();
		if(list.size()>0){
			return true;
		}
		return false;
	}
``` 
