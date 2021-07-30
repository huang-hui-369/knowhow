
# like '%参数%'

不能写
like '%#{mailTitle}%'
应该写
AND h.title LIKE '%${mailTitle}%'

# where
如果不能确定where的第一个条件是否一定有的话，要改用trim
<trim prefix=\"WHERE\" prefixOverrides=\"AND |OR \">

# xml文中的 >号 <号问题

xml文中的 >号 <号 等反义字符应该用<![CDATA[ 符号 ]]>包起来

<![CDATA[ >= ]]>


```java
 @Select(
    	{
    		"<script>"
    	    	    + "SELECT \n"
    	            + "    COUNT(DISTINCT h.id)\n"
    	            + " FROM\n"
    	            + "    t_sendmail_history h\n"
    	            + "        INNER JOIN\n"
    	            
    	            + "    t_sendmail_addr a ON h.id = a.t_sendmail_id\n"
    	            + "        INNER JOIN\n"
    	            + "    m_email_info m ON h.mail_type = m.belonging\n"
    	            
    	            + " <trim prefix=\"WHERE\" prefixOverrides=\"AND |OR \">\n"
    	            + "	 <if test='mailType != null'>\n"
    	            + "     h.mail_type = #{mailType}\n"
    	            + "	 </if>\n"
    	            
    				+ "	 <if test='mailTitle != null'>\n"
    	            + "        AND h.title LIKE '%${mailTitle}%'\n"
    	            + "	 </if>\n"
    	            
    				+ "	 <if test='mailText != null'>\n"
    	            + "        AND h.mail_text LIKE '%${mailText}%'\n"
    	            + "	 </if>\n"
    	            
    				+ "	 <if test='mailAddr != null'>\n"
    	            + "        AND a.to_addr LIKE '%${mailAddr}%'\n"
    	            + "	 </if>\n"
    	            
    				+ "	 <if test='fromSendTime != null'>\n"
    			    + "        AND h.send_time <![CDATA[ >= ]]> #{fromSendTime}\n"
    			    + "	 </if>\n"
    			    
    				+ "	 <if test='toSendTime != null'>\n"
    			    + "        AND h.send_time <![CDATA[ <= ]]> #{toSendTime}\n"
    			    + "	 </if>\n"
    	    
    	            + " </trim>\n"
    	       + "</script>"
    	}
    )
	
    int getCountByParams(@Param(value = "mailType") String mailType
    		, @Param(value = "mailTitle") String mailTitle
    		, @Param(value = "mailText") String mailText
    		, @Param(value = "mailAddr") String mailAddr
    		, @Param(value = "fromSendTime") LocalDateTime fromSendTime
    		, @Param(value = "toSendTime") LocalDateTime toSendTime
    		);
```