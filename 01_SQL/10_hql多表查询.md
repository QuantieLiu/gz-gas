
```
	/**
	 * 根据角色code获取user
	 * <li>默认查询参数user的flgActive=1，只获取正常的
	 * <li>可选参数：org_id,传null表示不需要部门id过滤
	 * @param org_id
	 * @param code
	 * @qly 20190109
	 * @return
	 */
	public List<User> findUserByCode(String code,String org_id);
```

<li>inner join的前提是实体有关联，且hql不支持on

```
	@SuppressWarnings("unchecked")
	@Override
	public List<User> findUserByCode(String code,String org_id){
		StringBuffer sb=new StringBuffer(" SELECT uu FROM Role rr , UserRole ur, User uu where ur.roleId =rr.id and ur.userId =uu.id ");
		sb.append(" and uu.flgActive = 1  and rr.code = ? ");
		if(StringUtils.isNotBlank(org_id)){
			sb.append(" and uu.orgId = ? ");    
			return this.getSession().createQuery(sb.toString()).setParameter(0, code).setParameter(1, org_id).list();
		}
		return this.getSession().createQuery(sb.toString()).setParameter(0, code).list();
	}
```
