##### 业务需求：统计特定状态的单位下的所有班组的项目数均大于5的 单位
##### 思路：join和having count 获取 特定状态的单位下的所有班组的项目数均大于5的班组id，再和单位的所有班组数比较

##### sql查询结果：单位id  班组数（项目数>5） 总班组数

###### 如果是查询 单位id 班组id 项目数，则在sql返回的list中是二维处理的数据，相对复杂
    
<li>service层
<li>数据字典code：PROJECT_TEAM_NOTOVER_NUM
<li>数据字典value：5
<li>数据字典明细项：参与统计的项目的systemStatus

``` 
    //获取 参与统计的项目的systemStatus后，组成 String[] 格式
    StringBuffer sb2 = new StringBuffer();
		List<DictionarySelect> listOver=DictionaryLib.getDictionarySelectByCode(ProjectConst.PROJECT_TEAM_NOTOVER_NUM);
		for(DictionarySelect dic:listOver){
			sb2.append(Globals.SEPARATOR_COMMA+dic.getId());
		}
    //Globals.SEPARATOR_COMMA是分隔符逗号,
    //StringBuffer -->  String (经过split) -->  String[]
    List<Object> list2=projectInfoDAO.getByUnitIdAndStatusHasTeam(sb2.substring(1).split(Globals.SEPARATOR_COMMA), numNot);
    //查询后结果处理
    if(list2.size()>0){
			List<String> list3=new ArrayList<String>();
			for(Object obj: list2){
				Object[] objs=(Object[])obj;
        //object对象比较是整个对象，这里只需比较内容是否相同即可，曾考虑转换成int比较数值相等，担忧转换失败的处理
				if( objs[1].toString().equals(objs[2].toString())){
					list3.add(objs[0].toString());
				}
			}
			if(list3.size()>0){
				String[] str2=new String[list3.size()];
				list3.toArray(str2);
				constructUnitDAO.updateServiceStatus(str2, ConstructConst.TEAM_STATUS_OVER);
			}
		}
``` 

<li>dao层

``` 
	@SuppressWarnings("unchecked")
	@Override
	public List<Object> getByUnitIdAndStatusHasTeam(String[] systemStatus,int num){
		//统计班组的项目数大于num
		StringBuilder sbIn =new StringBuilder(" SELECT COUNT(INFO.ID) AS PROCOUNT,PUNIT.CONSTRUCT_TEAM_ID ");
		sbIn.append(" FROM P_PROJECT_INFO INFO ");	
		sbIn.append(" JOIN P_PROJECT_UNIT PUNIT ON INFO.ID = PUNIT.PROJECT_ID AND PUNIT.CONSTRUCT_TEAM_ID IS NOT NULL ");	
		sbIn.append(" WHERE INFO.SYSTEM_STATUS in(:systemStatus) AND INFO.FLG_DELETED=0 GROUP BY PUNIT.CONSTRUCT_TEAM_ID ");
		sbIn.append(" HAVING COUNT(INFO.ID) > "+ num);	
		//统计哪些单位的班组项目数大于num的班组个数
		StringBuilder sbOIN1 =new StringBuilder(" SELECT TEAM.CONSTRUCT_ID, COUNT(TEAM.ID) AS TEAMTIMES ");
		sbOIN1.append(" FROM C_CONSTRUCT_TEAM TEAM JOIN( "+ sbIn +" ) JJ ");
		sbOIN1.append(" ON JJ.CONSTRUCT_TEAM_ID=TEAM.ID AND TEAM.FLG_DELETED=0 GROUP BY TEAM.CONSTRUCT_ID ");
		//统计单位的总班组个数
		StringBuilder sbOIN2 =new StringBuilder(" SELECT TEA.CONSTRUCT_ID, COUNT(TEA.ID) AS TEAMTOTAL ");
		sbOIN2.append(" FROM C_CONSTRUCT_TEAM TEA  ");
		sbOIN2.append(" JOIN C_CONSTRUCT_UNIT UNIT ON TEA.CONSTRUCT_ID=UNIT.ID AND  UNIT.SERVICE_STATES = 0  ");
		sbOIN2.append(" AND  UNIT.FLG_DELETED =0 AND TEA.FLG_DELETED=0 GROUP BY TEA.CONSTRUCT_ID ");
		//查询结果：单位id，班组项目数大于num的班组个数，总班组个数
		StringBuilder sbOut =new StringBuilder(" SELECT PP.CONSTRUCT_ID,PP.TEAMTIMES,UU.TEAMTOTAL ");
		sbOut.append(" FROM ( "+ sbOIN1 +" )PP JOIN ( "+ sbOIN2 +" )UU ON UU.CONSTRUCT_ID=PP.CONSTRUCT_ID ");
		return getSession().createSQLQuery(sbOut.toString()).setParameterList("systemStatus",systemStatus).list();
	}
``` 
