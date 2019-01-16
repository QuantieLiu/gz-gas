
#### 业务
<li>需求:1 依据有效期更新状态 2 依据有效期更新身份证状态 3 依据有效期更新逻辑删

<li>dic配置

```

```

<li>代码

```
	/**
	 * 定时任务
	 * <li>黑名单管理的逻辑删除
	 * <li>人员管理的身份证状态
	 * <li>焊机管理的状态
	 */
	public void updateSingleTables();
  
	@Override
	public void updateSingleTables(){
		//模拟登录
		LogonCertificate logon = new LogonCertificate();
		logon.setLoginId("sa");
		logon.setUserId("U00001");
		logon.setUserName("sa");
		LogonCertificateHolder.setLogonCertificate(logon);
		CommonLoginUser.addCurrentUserId(logon.getUserId());
		
		//获取有哪些表需要依据日期更新状态字段
		List<DictionarySelect> list=DictionaryLib.getDictionarySelectByCode(ConstructConst.PROJECT_STATUS_BY_DAY_TIMER);
		if(list.size()>0){
			for(DictionarySelect dic:list){
				if(StringUtils.isNotBlank(dic.getText())){
					DictionaryCacheObject dicObj = DictionaryLib.getDictionaryByCode(dic.getText());
					updateSingleTableStatus(dicObj);
				}
			}
		}
	}
  
```

```	
  /**
	 * 定时任务允许更新de业务表
	 */
	public static final String TABLE_NOT_UPDATE_TIMER = "C_CONSTRUCT_STAFF,C_CONSTRUCT_MACHINES";

	/**
	 * 具体到某个表的执行任务
	 * <li>在某某日期范围 把 状态 更新为 xx
	 * <li>{endDay:'+60',startDay:'+20',isUpdateAll:true}目前仅支持3个参数;
	 * <li>排序号是执行顺序，允许调号但不允许混乱的顺序;
	 * <li>值的前三个是必填参数，依次是依据某表的某个字段更新某个字段，
	 * <li>值的第四个是默认查询参数,可填
	 * @param dicObj
	 */
	public void updateSingleTableStatus(DictionaryCacheObject dicObj){
		if( dicObj != null){
			JSONObject obj;
			List<ConstructUpdateStatusVo> list =new ArrayList<ConstructUpdateStatusVo>();
			String[] str=dicObj.getValue().split(Globals.SEPARATOR_COMMA);
			boolean flag=false;
			//需在常量增加表名
			if( -1 < ConstructConst.TABLE_NOT_UPDATE_TIMER.indexOf(str[0]) ){
				flag=true;
			}
			if(flag){
				int i=0;
				for(DictionaryItemCacheObject dic: dicObj.getItems()){
					ConstructUpdateStatusVo vo=new ConstructUpdateStatusVo();
					vo.setTableName(str[0]);
					vo.setFieldDate(str[1]);
					vo.setFieldName(str[2]);
					if(StringUtils.isNotBlank(str[3]))
						vo.setDefaultParams(str[3]);
					obj=JSONObject.parseObject(dic.getRemark());
					vo.setStatus(dic.getValue());
					if(obj!=null){
						vo.setEndDay(obj.get("endDay")== null?ConstructConst.STRING_BLANK:obj.get("endDay").toString());
						vo.setStartDay(obj.get("startDay")== null?ConstructConst.STRING_BLANK:obj.get("startDay").toString());
						vo.setUpdateAll(obj.get("isUpdateAll") == null? false:true);
					}
					list.add(i, vo);
					i++;
				}
				if(list.size() == dicObj.getItems().size()){
					for(ConstructUpdateStatusVo v:list){
						constructStaffDAO.updateStatusByQuery(v);	
					}
				}
			}
		}
	}
```

