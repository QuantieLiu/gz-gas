
#### 业务
<li>需求:1 依据有效期更新状态 2 依据有效期更新身份证状态 3 依据有效期更新逻辑删

<li>dic配置

```
--控制哪些定时任务的数据字典
code：PROJECT_STATUS_BY_DAY_TIMER
item-(id-text): 1-PROJECT_STAFF_IDCARDSTATUS  1-PROJECT_MACHINES_STATUS  1-PROJECT_STAFF_BLACK

--以人员的身份证状态说明
code:PROJECT_STAFF_IDCARDSTATUS
value:C_CONSTRUCT_STAFF,id_expiry_date,ID_CARD_STATUS,FLG_DELETED=0
item(priority-id-text-value-remark):
1   0  正常      0   {isUpdateAll:true}
2   1  即将过期   1   {endDay:'+60'}
4   2  过期      2

该数据字典只能由开发人员添加或修改;
{endDay:'+60',startDay:'+20',isUpdateAll:true}目前仅支持3个参数;
注意按排序号填写，排序号是执行顺序，允许调号但不允许混乱的顺序;
值的前三个是必填参数，依次是依据某表的某个字段更新某个字段，
值的第四个是默认查询参数，可填
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

```
	@Override
	public String updateStatusByQuery(ConstructUpdateStatusVo vo){
		if(StringUtils.isBlank(vo.getTableName()) || StringUtils.isBlank(vo.getFieldName()) || StringUtils.isBlank(vo.getStatus()) )
			return null;
		StringBuilder sb =new StringBuilder("UPDATE "+ vo.getTableName() +" SET "+ vo.getFieldName() +" =(:upStatus) ");
		sb.append(" WHERE "+ vo.getFieldDate() +" IS NOT NULL ");
		if( !vo.isUpdateAll() ){
			if(StringUtils.isNotBlank(vo.getStartDay())  ||  StringUtils.isNotBlank(vo.getEndDay()) ){
				sb.append(" AND "+ vo.getFieldDate()+" between  sysdate"+ vo.getStartDay()+ " AND sysdate"+ vo.getEndDay());
			}else{
				sb.append(" AND "+vo.getFieldDate() + " < SYSDATE ");
			}
		}
		if(StringUtils.isNotBlank(vo.getDefaultParams()))
			sb.append(" AND " +vo.getDefaultParams());
		getSession().createSQLQuery(sb.toString()).setParameter("upStatus", vo.getStatus()).executeUpdate();
		return "success";
	}
```

```
/**
 * 定时任务-根据日期更新某状态字段
 * <li>用于拼接sql，执行update语句
 * <li>仅适用于单表 
 * @author qly
 */
public class ConstructUpdateStatusVo {
	
	/**
	 * 表名
	 */
	protected String tableName;
	/**
	 * 字段名
	 */
	protected String fieldName;
	/**
	 * 依据的哪个日期
	 */
	protected String fieldDate;
	/**
	 * 默认SYSDATE
	 * <li>若只有正常和过期，可不填
	 * <li>今天开始的30天，则填写+30
	 */
	protected String startDay;
	/**
	 * 30-60天，则填写-60天
	 */
	protected String endDay;
	/**
	 * 状态
	 */
	protected String status;
	/**
	 * 默认查询条件，可不填
	 * <li>以sql格式填写
	 */
	protected String defaultParams;
	/**
	 * 是否全表更新，默认否
	 */
	protected boolean isUpdateAll=false;
}	
```
