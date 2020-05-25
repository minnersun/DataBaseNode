## Mysql代码积累

----

### count

##### 过滤条件进行查询

###### 案例

```xml
<!--首页-各级危险源数量分析-各级危险源报警数量查询--><select id="getAlarmAtAllLevels" resultType="com.wiscom.domain.CAlarmLevel">    
    SELECT COUNT(CASE WHEN c.ALERT_LEVEL='E3802' THEN 1 END) alarmOneNumber ,     
        COUNT(CASE WHEN c.ALERT_LEVEL='E3803' THEN 1 END) alarmTwoNumber,     
        COUNT(CASE WHEN c.ALERT_LEVEL='E3804' THEN 1 END) alarmThreeNumber,     
        COUNT(CASE WHEN c.ALERT_LEVEL='E3805' THEN 1 END) alarmFourNumber    
    FROM c_mw_sensoralert c
</select>
```

### 子查询

###### 案例

```xml
    <select id="getgetEarlyWarningEnterpriseTrees" resultType="com.wiscom.domain.Trees">
        SELECT ent.ENTERPRISE_ID enterpriseId,ent.precinct_name enterpriseName,cp1.precinct_id DistrictId,
         cp1.precinct_name DistrictName,cp2.precinct_id provincialId,cp2.precinct_name provincialName
         FROM
	        (SELECT cms.ENTERPRISE_ID ,cp.precinct_name,cp.up_precinct_id
		        FROM c_mw_sensoralert cms
		            LEFT JOIN cfg_precinct cp
			            ON cms.`ENTERPRISE_ID`=cp.`precinct_id`
	        GROUP BY ENTERPRISE_ID) AS ent
	        LEFT JOIN cfg_precinct cp1
		        ON cp1.`precinct_id`=ent.up_precinct_id
	        LEFT JOIN cfg_precinct cp2
		        ON cp2.`precinct_id`=cp1.`up_precinct_id`
    </select>
```



### group by与order by结合

###### 案例

```xml
    <select id="getAllEnterpriseAlarmLevels" resultType="com.wiscom.domain.CAlarmLevel">
		SELECT COUNT(CASE WHEN c.ALERT_LEVEL='E3802' THEN 1 END) alarmOneNumber ,
			COUNT(CASE WHEN c.ALERT_LEVEL='E3803' THEN 1 END) alarmTwoNumber,
			COUNT(CASE WHEN c.ALERT_LEVEL='E3804' THEN 1 END) alarmThreeNumber,
			COUNT(CASE WHEN c.ALERT_LEVEL='E3805' THEN 1 END) alarmFourNumber,
			COUNT(CASE WHEN c.ALERT_LEVEL='E3802' OR c.ALERT_LEVEL='E3803' OR  c.ALERT_LEVEL='E3804' OR c.ALERT_LEVEL='E3805' THEN 1  END) allAlarmNumber,
			ENTERPRISE_NAME enterpriseName,
			ENTERPRISE_ID enterpriseId
		FROM c_mw_sensoralert c
		<where>
			<if test="startTime != null and startTime !='' ">
				CREATE_TIME <![CDATA[>=]]> ${startTime}
			</if>
			<if test="endTime != null and endTime !='' ">
				and CREATE_TIME <![CDATA[<=]]> ${endTime}
			</if>
		</where>
		GROUP BY ENTERPRISE_ID
		ORDER BY ${alertLevel} DESC
		limit 0,5
    </select>
```



### union all 查询不关联的表

###### 案例

```xml
	<select id="getMajorHazardSourcesClassificationNumber" resultType="java.lang.String">
			SELECT * FROM (
				  (SELECT COUNT(JAR_ID) AS Numbar FROM c_ei_jar )
			UNION ALL (SELECT COUNT(PRODDEV_ID) FROM c_ei_proddev )
			UNION ALL (SELECT COUNT(CHMSTOR_ID) FROM c_ei_chmstor)
			 ) c
	</select>
```



### mysql实现decode的方式

###### 案例

```xml
	<select id="getEarlyWarningCause" resultType="com.wiscom.vo.SensorEquipmentVO">
		SELECT (CASE ALERT_REASON WHEN 'HF901' THEN '仪表故障'
					WHEN 'HF902' THEN '操作不当'
					WHEN 'HF903' THEN '误操作'
					WHEN 'HF904' THEN '误报'
					WHEN 'HF905' THEN '生产波动'
					WHEN 'HF906' THEN '物料泄露'
					WHEN 'HF907' THEN '外来干扰'
				ELSE '其他'
			END) ALERT_REASON,COUNT(1) number
		FROM c_mw_sensoralert GROUP BY ALERT_REASON
	</select>
```

### 组内排序

> `group_concat([DISTINCT] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符'])`

###### 案例

> join连接后的表根据`id_number`进行分组，组中的`direction`字段根据`add_time`进行排序，数据间以`,`间隔

```xml
    <select id="selectRegionOnDutyReal" resultType="com.wiscom.domain.HisFace">
        SELECT hf.face_name face_name,hf.id_number id_number,hf.precinct_id precinct_id,cp.`job` job,
        
        GROUP_CONCAT(hf.direction ORDER BY hf.add_time DESC) direction
        
        FROM his_face hf
        LEFT JOIN cfg_person cp ON hf.`id_number`=cp.`id_number`
        <where>
            <if test="precinctId != null and precinctId !='' ">
                hf.precinct_id = #{precinctId}
            </if>
            <if test="startTime != null and startTime != '' ">
                AND hf.add_time <![CDATA[>=]]> #{startTime}
            </if>
            <if test="endTime != null and endTime != '' ">
                AND hf.add_time <![CDATA[<]]> #{endTime}
            </if>
        </where>
        GROUP BY id_number
    </select>
```