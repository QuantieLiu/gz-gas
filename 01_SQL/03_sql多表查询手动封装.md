
<li>service

```
	/**
	 * 资质流程-更新资质-选择资质证件的弹框-数据来源
	 * <li>同一部资质不允许在流程中重复发起
	 * @param verifyid
	 * @param pagelet
	 * @param queryCriterion
	 * @return
	 * @throws Exception
	 */
	public Pagelet getCertificateNotinVerify(String verifyid,String unitId,QueryCriterion queryCriterion)throws Exception;
  
	@Override
	public Pagelet getCertificateNotinVerify(String verifyid,String unitId,QueryCriterion queryCriterion)throws Exception{
		Pagelet page=constructCertificateDAO.getCertificateNotinVerify(verifyid, unitId, queryCriterion);
		List<Object> list=page.getDatas();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");//要转换的时间格式
		List<ConstructCertificateForm> listMac=new ArrayList<ConstructCertificateForm>();
		ConstructCertificateEntity entity;
		if(list.size()>0){
			for(Object obj:list){
				entity=(ConstructCertificateEntity)obj;
				ConstructCertificateForm form=new ConstructCertificateForm();
				form.setId(entity.getId()== null?ConstructConst.STRING_BLANK:entity.getId());
				form.setConstructId(entity.getConstructId()== null?ConstructConst.STRING_BLANK:entity.getConstructId());
				form.setCertificateCode(entity.getCertificateCode()== null?ConstructConst.STRING_BLANK:entity.getCertificateCode());
				form.setExpiryDate((entity.getExpiryDate()==null?ConstructConst.STRING_BLANK:sdf.format(entity.getExpiryDate())));
				form.setCertificateType(entity.getCertificateType()== null?ConstructConst.STRING_BLANK:entity.getCertificateType());
				form.setCertificateTypeDesc(entity.getCertificateType()== null?ConstructConst.STRING_BLANK:DictionaryLib.getItemName(ConstructConst.CERTIFICATE_TYPE,entity.getCertificateType()));
				form.setPriority(entity.getPriority());
				form.setRemark(entity.getRemark()== null?ConstructConst.STRING_BLANK:entity.getRemark());
				listMac.add(form);
			}
		}
		page.getDatas().clear();
		page.getDatas().addAll(listMac);	
		return page;
	}
```


<li>dao-notin多表查询
<li>手动处理查询参数，从前台传实体的属性到后台
<li>查询参数仅支持单表

```	
  /**
	 * 根据单位id获取在该流程中未被选择的资质
	 * <li>仅支持单表ConstructCertificateEntity查询-请把属性传到后台
	 * @param unitId
	 * @param pagelet
	 * @param queryCriterion
	 * @return
	 */
	public Pagelet getCertificateNotinVerify(String unitId,String verifyId,QueryCriterion queryCriterion);


	@Override
	public Pagelet getCertificateNotinVerify(String verifyId,String unitId,QueryCriterion queryCriterion){
		List<Object> objs = new ArrayList<Object>();
		StringBuilder sb=new StringBuilder().append("SELECT CM FROM ConstructCertificateEntity CM WHERE CM.flgDeleted =0 AND CM.constructId =? ");
		objs.add(unitId);
		if(StringUtils.isNotBlank(verifyId)){
			sb.append(" AND CM.id NOT IN (SELECT MVD.certificateId FROM CertificateVerifyDetailEntity MVD WHERE MVD.certificateVerifyId = ? ");
			sb.append(" AND MVD.verifyType ='2' AND MVD.flgDeleted =0 )");
			objs.add(verifyId);
		}
		if(queryCriterion !=null){
			if(queryCriterion.getQueryParams().size() > 0){
				for (QueryParam p : queryCriterion.getQueryParams()) {
					sb.append(" and CM.").append(p.toQueryString());
					p.addParamValuesTo(objs);
				}
			}
		}
		return super.queryPagedResult(sb.toString(),objs.toArray(new Object[]{}));
	}
```

<li>非hql实体查询，需手动处理分页参数
	
```
	public Pagelet queryUserBySearch(String[] code, String searchStr,Pagelet pagelet){
		//先用sql拼装每个user对应的角色名称
		StringBuffer sbin = new StringBuffer(" SELECT uin.ID,rin.ROLE_NAME,rin.CODE  FROM sys_user uin ");
		sbin.append(" JOIN sys_user_role urin on uin.id=urin.user_id ");
		sbin.append(" JOIN sys_role rin on urin.ROLE_ID=rin.ID ");
		if(code.length >0){
			sbin.append("and rin.code in (:codes) ");
		}
		//聚合获取角色名称
		StringBuffer sbout = new StringBuffer(" SELECT tt.id,GROUP_CONCAT(tt.ROLE_NAME SEPARATOR  ',') as role_name, GROUP_CONCAT(tt.`CODE` SEPARATOR ',') AS role_code");
		sbout.append(" from ( "+ sbin +" )tt GROUP BY tt.id");
		//查询数据集
		//distinct(uc.LOGON_ID)避免重复的数据显示到页面
		StringBuffer sb = new StringBuffer(" SELECT distinct(uc.LOGON_ID),uc.NAME,u.id,u.USER_INFO_ID,uc.FLG_ACTIVE,u.ORG_ID,u.ORG_NAME,tr.role_name,tr.role_code ");
		sb.append(" FROM sys_user u JOIN sys_user_comminfo uc on uc.ID=u.USER_INFO_ID ");
		sb.append(" JOIN sys_user_role ur ON ur.user_id=u.id ");
		sb.append(" JOIN sys_role r ON r.ID=ur.ROLE_ID ");
		sb.append(" JOIN (" + sbout +")tr ON tr.ID=u.id ");	
		//正常情况都会有值
		if(code.length >0){
			sb.append("and r.code in (:codes) ");
		}
		if (StringUtils.isNotBlank(searchStr)) {
			sb.append("and  (uc.LOGON_ID like (:searchStr) or uc.name like (:searchStr)) ");
		}
		sb.append("order by u.CREATE_TIME DESC ");
		SQLQuery query = this.getSession().createSQLQuery(sb.toString());
		List<Object> list;
		//放入查询参数
		if(code.length >0){
			query.setParameterList("codes", code);
		}
		if (StringUtils.isNotBlank(searchStr)) {
			query.setParameter("searchStr", "%"+searchStr+"%");
		}
		list = query.setFirstResult(pagelet.getPerPageSize()*(pagelet.getPageNo()-1)).setMaxResults(pagelet.getPerPageSize()).list();
		pagelet.setDatas(list);
		//手动处理分页参数
		//查询总数
		StringBuffer sbCount = new StringBuffer(" SELECT COUNT(*) FROM ( ");
		sbCount.append(sb.toString() +" )aa ");
		SQLQuery queryCount = this.getSession().createSQLQuery(sbCount.toString());
		//放入查询参数
		if(code.length >0){
			queryCount.setParameterList("codes", code);
		}
		if (StringUtils.isNotBlank(searchStr)) {
			queryCount.setParameter("searchStr", "%"+searchStr+"%");
		}
		List<Object> listCount=queryCount.list();
		pagelet.setTotalCount(Integer.parseInt(listCount.get(0).toString()));
		return pagelet;
	}
```
