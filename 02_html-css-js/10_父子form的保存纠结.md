
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

```

```


<li>

```

```


<li>




<li>

```

```


