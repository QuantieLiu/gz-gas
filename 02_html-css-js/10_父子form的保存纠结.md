
<li>html

```
			<div class="cont-table tab-content">
				<!-- cont-table -->
				<div class="tab-pane fade in active" id="mainTab">
					<form onsubmit="return false;" id="mainform" class="table-normal" enctype="multipart/form-data">
						<jsp:include page="/WEB-INF/jsp/module/view/attach/attach_list.jsp" flush="true"/>
					</form>
				</div>
				<!-- childTab信息 -->
				<c:forEach items="${module.childTab}" var="o">
					<div id="${o.tabid}" style="margin-top: 10px;">	
											<div class="location"><i class="fa fa-map-marker"></i>${o.moduleName}</div>
						<div class="common-option">					
						<div class="inner">
							<button type="button" class="btn btn-icon" onclick="goDetailPage('${o.moduleId}','${o.tabid}','1')"><i class="icon iconfont icon-icon-plus"></i>新增资质</button>
							<button type="button" class="btn btn-icon" onclick="modifyDetailPage('${o.moduleId}','${o.tabid}','2')"><i class="icon iconfont icon-icon-coin"></i>更新资质</button>
 							<button type="button" class="btn btn-icon" onclick="changeDetailPage('${o.moduleId}','${o.tabid}')"><i class="icon iconfont icon-icon-edit"></i>修改资质</button>
							<button type="button" class="btn btn-icon" onclick="deleteByIds('${o.moduleId}','${o.tabid}')"><i class="icon iconfont icon-icon-trash"></i>删除</button>
						</div> 
						</div> 
						<div class="common-table frameBorder" id="${o.tabid}">
							<table id="listTable${o.moduleId}" class="table table-hover tb_sltBox2"></table>
						</div>
					</div>
  				</c:forEach>
			</div>
```

<li>基于已保存父form的js，window
<li>亮点是callback回调

```
//是否还有资质证件可更新，true有，false无
	var canAddUpd = '${canAddUpd}'
//当前审核工单是否有待审核的资质证件，true有，false无
	var canApply = '${canApply}'
  
	/* 新增-资质 */
	function goDetailPage(mdId,mdCode,verify){
		if(pkId){
			FF_SaveForFlow(function(){
				if(window.opener && !window.opener.closed) {
                	window.opener.location.reload();
                }
  	  		});
		}
		WinPrint2("${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId="+mdId+"&mdCode="+mdCode
				+"&verifyId="+pkId+"&verify="+verify+"&menuId=${param.menuId}&openStyle=1&random="+Math.random());
	}	
	/* 更新-资质 */
	function modifyDetailPage(mdId,mdCode,verify){
		if(pkId){
			FF_SaveForFlow(function(){
				if(window.opener && !window.opener.closed) {
                	window.opener.location.reload();
                }
  	  		});
		}
		if(canAddUpd == 2){
			WinPrint2("${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId=&mdCode=CERTIFICATE_VERIFY_UPDATE"
					+"&verifyId="+pkId+"&verify="+verify+"&menuId=${param.menuId}&openStyle=1&random="+Math.random());
		}else{
			BT.showWarning("已无资质可申请更新审核，请新增施工单位资质！"); 
		}
	}	
  /* 关闭当前页面 */
  function FF_Cancel(){
		WinClose();
	}
  /* 保存表单-业务页面调用时reload/流程调用时next */
 	function FF_SaveForFlow(callback){
		$("#mainform").attr("action","${baseURL}/construct/certificateverify/certificateverifysave/saveIncludeFile.do?mdCode=CERTIFICATE_VERIFY&strategy=${strategy}");
		BT.submitForm($("#mainform"), function (data) {
			if (data && data.hasOk) { 
				callback();
	        } else { 
	        	BT.showError("操作失败!失败信息如下:\n" + data.message + "\n如您对以上信息有疑问，请联系管理人员!");
	        }
		});
	} 
```

补充框架的流程继续方法next

```
--括号部分是callback
saveBeforeSubmit(function(){
  	openProcessDialog(_this);
})

// 流程提交前保存表单信息	
	function saveBeforeSubmit(callback){  
		try {
			if (FF_SaveForFlow && typeof (FF_SaveForFlow) == "function") {
				FF_SaveForFlow(callback);
				return false;
			}
		} catch (e) {
		}
		callback();
		return false;
	}
	
```

<li>父form未保存，基于一个window的tab
在只要打开子form页面，不管父form有没有编辑，一律保存父form

```
	/* 理论上id为空专用-当父form有附件或input框有值时，先保存父form工单，再打开子form页面 */
	function checkForm(callback){
		var flag=false;//默认不保存工单
		if(pkId == ""){
			var attId =$('#file-input-0').val();
			var unitName=$('input[name=updateConstructName]').val() + $('input[name= updateAbbr]').val();
			if( (attId != "" && typeof(attId) != "undefined") || (unitName != "" &&  typeof (unitName) != "undefined")){
				flag=true;
			}
		}else{
			flag=true;
		}
	    callback(flag);
	}
	/* 新增-资质 */
	function goDetailPage(mdId,mdCode,verify){
		if(pkId){
			parent.f_addTab("CERTIFICATE_VERIFY_DETAIL", '资质信息', "${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId="+mdId+"&mdCode="+mdCode
					+"&verifyId="+pkId+"&verify="+verify+"&menuId=${param.menuId}&random="+Math.random());
			saveForm();
		}else{
			checkForm(function(flag){
				if(flag){
					BT.showConfirm("提示","当前表单尚未保存，\n你确定保存施工单位资质审核工单信息吗？\n若是请点击确定按钮！",function(){
						saveForm();
					})
				}else{
					parent.f_addTab("CERTIFICATE_VERIFY_DETAIL", '资质信息', "${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId="+mdId+"&mdCode="+mdCode
							+"&verifyId="+pkId+"&verify="+verify+"&menuId=${param.menuId}&random="+Math.random());	
				}
			});
		}
	}	
	/* 更新-资质 */
	function modifyDetailPage(mdId,mdCode,verify){
		if(pkId){
			parent.f_addTab("CERTIFICATE_VERIFY_UPDATE", '资质信息', "${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdCode=CERTIFICATE_VERIFY_UPDATE&verifyId="+pkId
					+"&verify="+verify+"&menuId=${param.menuId}&random="+Math.random());
			saveForm();
		}else{
			checkForm(function(flag){
				if(flag){
					BT.showConfirm("提示","当前表单尚未保存，\n你确定保存施工单位资质审核工单信息吗？\n若是请点击确定按钮！",function(){
						saveForm();
					})
				}else if(canAddUpd == 2){
					parent.f_addTab("CERTIFICATE_VERIFY_UPDATE", '资质信息', "${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId=&mdCode=CERTIFICATE_VERIFY_UPDATE"
							+"&verifyId="+pkId+"&verify="+verify+"&menuId=${param.menuId}&random="+Math.random());
				}else{
					BT.showWarning("该施工单位未有资质可申请更新审核，请新增资质信息！");
				}
			});
		}
	}	
	/* 修改 */
	function changeDetailPage(mdId,mdCode){
		var selection = $("#listTable"+mdId).bootstrapTable('getSelections');
		if(selection.length==1){
			var row = selection[0];
			if(row.verifyType == 1){
				parent.f_addTab("CERTIFICATE_VERIFY_DETAIL", '资质信息', "${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId="
						+"&mdCode=CERTIFICATE_VERIFY_DETAIL&id="+row.id+"&random="+Math.random());
				if(pkId){
					saveForm();
				}
			}else if(row.verifyType == 2){
				parent.f_addTab("CERTIFICATE_VERIFY_UPDATE", '资质信息', "${baseURL}/construct/certificateverifydetail/certificateverifydetailedit/superEdit.do?mdId="
						+"&mdCode=CERTIFICATE_VERIFY_UPDATE&id="+row.id+"&random="+Math.random()); 
				if(pkId){
					saveForm();
				}
			}else{
				BT.showWarning("数据出了点小问题，请刷新页面！"); 
			}
		}else{
			BT.showWarning("请选中一行！"); 
		}
	}
	/* 审核工单-保存按钮 */
	function FF_Save(){
		checkForm(function(flag){
			if(flag){
				saveForm()
			}else{
				//啥也没填，不让你保存
				BT.showWarning("请先填写表单！"); 
			}
		});
	}
	/* 审核工单-保存form */
	function saveForm(){
		$("#mainform").attr("action","${baseURL}/construct/certificateverify/certificateverifysave/saveIncludeFile.do?mdId=${param.mdId}&mdCode=${param.mdCode}&strategy=${strategy}");
		BT.submitForm($("#mainform"), function (data) {
			if (data && data.hasOk) { 
				verifyId = data.bean.id;
				$('#mainform input[name=id]').val(verifyId);
				BT.showSuccess('操作成功!',function(){
					parent.f_addTab("CERTIFICATE_VERIFY", '资质审核工单', "${baseURL}/construct/certificateverify/certificateverifyedit/flowEdit.do?mdCode=CERTIFICATE_VERIFY&menuId=${param.menuId}&id="
							+verifyId+"&random="+Math.random());
				});
            } else { 
            	BT.showError("操作失败!失败信息如下:\n" + data.message + "\n如您对以上信息有疑问，请联系管理人员!");
            }
		});
	}
	function FF_Cancel(){
        win.LG.closeCurrentTab("CERTIFICATE_VERIFY");
	}
	/* 发起流程 */
	$(".l-routeSubmit").click(function(){
		if(canApply == '1'){
			jQuery.ajax({
				type: "POST",
				url:"${baseURL}/jdbc/common/basecommonsave/saveFlow.do?id="+pkId+"&mdId=${param.mdId}&mdCode=${param.mdCode}&strategy=${strategy}",
				dataType:'json',
				success:function(data){
					if(data && data.hasOk == true){
						BT.showSuccess("操作成功!",function(){
							win.LG.closeAndReloadParent("CERTIFICATE_VERIFY", "${param.menuId}");		
						});
					}else{
						BT.showError("操作出错!"+data.message);
					}
				}
			})
		}else if(canApply =='' || canApply == 0){
			BT.showError("请先添加需要审核的施工单位资质!");	
		}else{
			BT.showError("出了点小问题，请刷新当前页面!");
		}
	})
```

