<li>实体对应的dao层，两表联查

```
	@Override
	public Pagelet findStaffByUnit(String constructId){
		String hql="select cs from ConstructStaffEntity cs,ConstructUnitStaffEntity us where cs.id =us.staffId and us.constructId = ? and  cs.flgDeleted=0 ";	
		return super.queryPagedResult(hql,constructId);
	}
```

<li>sql-update，in过滤

```
	@Override
	public void updateServiceStatusByService(String[] serviceStatus,String status){
		StringBuilder sb =new StringBuilder("UPDATE C_CONSTRUCT_UNIT sd SET sd.SERVICE_STATES=? WHERE sd.SERVICE_STATES in (:systemStatus) ");
		getSession().createSQLQuery(sb.toString()).setParameter(0, status).setParameterList("systemStatus", serviceStatus).executeUpdate();
	}
```

<li>hql-notin查询

```
	@SuppressWarnings("unchecked")
	@Override
	public List<ConstructCertificateEntity> getCertificateNotinVerify(String verifyId,String unitId){
		StringBuilder sb=new StringBuilder().append("SELECT CM FROM ConstructCertificateEntity CM WHERE CM.flgDeleted =0 AND CM.constructId =? ");
		sb.append(" AND CM.id NOT IN (SELECT MVD.certificateId FROM CertificateVerifyDetailEntity MVD WHERE MVD.certificateVerifyId = ? ");
		sb.append(" AND MVD.verifyType ='2' AND MVD.flgDeleted =0 )");
		return getSession().createQuery(sb.toString()).setParameter(0, unitId).setParameter(1, verifyId).list();
	}
```

<li>从数据库中获取最大的员工编号
	
```
	@Override
	public Object getStaffMaxCode(){
		StringBuilder sb =new StringBuilder("select max(ABS(code)) from C_CONSTRUCT_STAFF Where DATA_SOURCE = 0 ");
		return getSession().createSQLQuery(sb.toString()).uniqueResult();
	}
```

<li>QueryCriterion
	
```
		QueryCriterion queryCriterion = new QueryCriterion();
		if (StringUtils.isNotBlank(idCard)){
			queryCriterion.addParam(new QueryHibernatePlaceholderParam("isBlack", ConstructConst.Staff_NOT_BLACK, 
					null, QueryOperSymbolEnum.eq, QueryParam.PARAM_PLACEHOLDER_NAME))
					.addParam(new QueryHibernatePlaceholderParam("idCard", idCard, 
					null, QueryOperSymbolEnum.eq, QueryParam.PARAM_PLACEHOLDER_NAME));
			if(StringUtils.isNotBlank(unitId)){
				queryCriterion.addParam(new QueryHibernatePlaceholderParam("constructId", unitId, 
						null, QueryOperSymbolEnum.eq, QueryParam.PARAM_PLACEHOLDER_NAME));
			}
			queryCriterion.addOrder(new QueryOrderby(0, "createTime", SortOrderEnum.desc));
			queryCriterion.setMaxResults(1);
			List<ConstructStaffEntity> list = this.find(queryCriterion);
```

<li>查询结果无实体，且手动处理分页
	
```
	@SuppressWarnings("unchecked")
	@Override
	public Pagelet getByConstruct(String constructId,Pagelet pagelet){
		StringBuilder sb =new StringBuilder("SELECT TT.ID as teamId,TT.TEAM_NAME,TT.STATUS,TT.LEADER_ID,TT.LEADER_NAME,SS.STAFF_NAME,SS.STAFF_TYPE,");
		sb.append(" SS.ID_CARD as idCard,SS.ID_EXPIRY_DATE as expiryDate,SS.STATUS as staffStatus ");
		sb.append(" FROM C_CONSTRUCT_TEAM_STAFF TF,C_CONSTRUCT_STAFF SS,C_CONSTRUCT_TEAM TT ");
		sb.append(" where TF.STAFF_ID=SS.ID and TF.TEAM_ID=TT.ID and TT.FLG_DELETED=0 AND TT.CONSTRUCT_ID= ? ");
		sb.append(" order by teamId ");
		SQLQuery query = this.getSession().createSQLQuery(sb.toString());
		List<Object> list;
		list = query.setParameter(0, constructId).setFirstResult(pagelet.getPerPageSize()*(pagelet.getPageNo()-1)).setMaxResults(pagelet.getPerPageSize()).list();
		pagelet.setDatas(list);
		String sqlcount="SELECT COUNT(*) FROM ("+ sb.toString() +")";
		List<Object> listCount=this.getSession().createSQLQuery(sqlcount).setParameter(0, constructId).list();		
		pagelet.setTotalCount(Integer.parseInt(listCount.get(0).toString()));
		return pagelet;
	}
```

<li>hql查询，中间表只做过滤
	
```
	@SuppressWarnings("unchecked")
	@Override
	public List<ConstructUnitStaffEntity> getByUnitStaff(String unitId,String[] staffId){
		StringBuilder sb =new StringBuilder("select us from ConstructUnitStaffEntity us where us.constructId = (:constructId) and us.staffId in (:staffId) ");
		return getSession().createQuery(sb.toString()).setParameter("constructId", unitId).setParameterList("staffId", staffId).list();
	}
```

<li>
	
```

```
