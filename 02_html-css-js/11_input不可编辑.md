
<li>动态生成input
<li>业务需求，类型为某值时不可修改

```
--原网页
<input maxlength="64" inputwidth="230" id="certificateCode" name="certificateCode" class=" certificateCode required notblank" 
type="text" edittip="undefined" regexrule="undefined" moduleid="" relatedmoduleid="">

--js后效果
<input maxlength="64" inputwidth="230" id="certificateCode" name="certificateCode" class=" certificateCode required notblank" 
type="text" edittip="undefined" regexrule="undefined" moduleid="" relatedmoduleid="" disabled="">
```

```
--不允许修改的 资质证件分类 值
	var onlyCode='${onlyCode}';
  
--从弹框选择事件的定制function
		if( verify == 2 && row.certificateType.indexOf(onlyCode) >-1 ){
			  document.getElementById('certificateCode').disabled="true";
		}
    
-- 
certificateType：1,2,3,4,5
onlyCode的值：14,表明明细项中的 1 或者 4 都不允许 修改 certificateCode
```

<li>action的superEdit
页面跳转方法

```
--把 不允许修改certificateCode 放在数据字典的 值 要求该值需在 明细项值的范围之内
		request.setAttribute("onlyCode", DictionaryLib.getDictionaryValue(ConstructConst.CERTIFICATE_TYPE));
```

严格来说，应该根据源数据表的id重新获取certificateCode，不该使用前台传过来的value

```
		//若是营业执照，不允许修改证件编号
		if(DictionaryLib.getDictionaryValue(ConstructConst.CERTIFICATE_TYPE).indexOf(custForm.getFieldValue("certificateType")) > -1){
			//custForm.getFieldValue("name")是原来的证件编号，非数据库字段
			custForm.setFieldValue("certificateCode",custForm.getFieldValue("name"));
		}
```

<li>补充知识点

```
disabled 属性规定应该禁用 input 元素，被禁用的 input 元素，不可编辑，不可复制，不可选择，不能接收焦点,后台也不会接收到传值。
设置后文字的颜色会变成灰色。disabled 属性无法与 <input type="hidden"> 一起使用。
示例：<input type="text" disabled="disabled" />
```

```
readonly 属性规定输入字段为只读可复制，但是，用户可以使用Tab键切换到该字段，可选择,可以接收焦点，还可以选中或拷贝其文本。
后台会接收到传值. readonly 属性可以防止用户对值进行修改。
readonly 属性可与 <input type="text"> 或 <input type="password"> 配合使用。
示例：<input type="text" readonly="readonly">
```

