
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
手动处理查询参数，从前台传实体的属性到后台

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
