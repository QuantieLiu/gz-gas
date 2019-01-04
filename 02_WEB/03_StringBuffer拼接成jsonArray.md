### 主要是由于 当前登录的放在第一个，又不想循环2次

``` 
	/**
	 * 同一个手机号码对应的所有施工单位
	 * @throws IOException
	 */
	@Action
	public void listDicUnitUser() throws IOException{
		String code=LogonCertificateHolder.getLogonCertificate().getRoleCode();
		if(code.equals(ConstructConst.U_UNIT)){
			String phone=LogonCertificateHolder.getLogonCertificate().getLoginId();
			StringBuffer sb=new StringBuffer();
			sb=constructUnitService.getUnitByPhone(phone,LogonCertificateHolder.getLogonCertificate().getUserId());
			response.getWriter().write(sb.toString());
		}
	}
``` 

``` 
	public StringBuffer getUnitByPhone(String phone,String nowUserId) {
		StringBuffer buffer1 = new StringBuffer();
		StringBuffer buffer2 = new StringBuffer();
		List<ConstructUnitEntity> listu=constructUnitDAO.findByPhone(phone);
		if(listu.size()>1){
			for(ConstructUnitEntity unit:listu){
				if(unit.getStatus().equals(ConstructConst.STRING_ZERO))
					continue;
				if(StringUtils.isBlank(unit.getSysUserId()))	
					continue;
				JSONObject obj=new JSONObject();
				obj.put("id", unit.getSysUserId());
				if(StringUtils.isNotBlank(unit.getAbbr())){
					obj.put("text", unit.getAbbr());
				}else{
					obj.put("text", unit.getName().substring(0, 4));
				}
				obj.put("other", unit.getStatus());
				//当前登录的放在第一个
				if(unit.getSysUserId().equals(nowUserId)){
					buffer1.append(obj.toString());
				}else{
					buffer2.append(Globals.SEPARATOR_COMMA+obj.toString());
				}
			}
			//帐号无效或无sysUserId的不append buffer2
			int result=buffer2.indexOf(Globals.SEPARATOR_COMMA);
			if(result != -1){
				buffer1.append(buffer2).insert(0, "[").append("]");
			}
		}
		return buffer1;
	}
``` 
