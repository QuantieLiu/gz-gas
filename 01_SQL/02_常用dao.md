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
