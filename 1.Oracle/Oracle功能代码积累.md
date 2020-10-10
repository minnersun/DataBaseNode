## Oracle功能代码积累

-------

### oracle分页查询

> 参考
>
> > `https://www.cnblogs.com/wuxu/p/11198764.html`
> >
> > `https://www.cnblogs.com/chenning/p/4924798.html`

```xml
<!-- 
	 解析： 
	 	row_.* : 查询row_表中的所有字段
		rownum : oracle提供自动给查询数据编号的关键字，单条语句时只能做 '<','<=' 判断，不能做 '>','>=' 判断，在子查询中可以使用'>','>=',但需要起别名
	 	rownum_ :rownum的别名
	 	row_ : 为查询出所有的字段建的一张临时表
		<![CDATA[  ]]> : 防止'<','>'等被转义
-->
 select * 
 from ( select row_.*, rownum rownum_ from ( 
     
 ) row_ ) 
<![CDATA[
 	where rownum_ > #{record.page.begin} and rownum_ <= #{record.page.end} 
]]>
```

###### 案例：

```xml
    <!-- ORACLE数据库分页查询通用模板 -->
    <sql id="OracleDialectPrefix">
        <if test="record.page != null">
            select * from ( select row_.*, rownum rownum_ from (
        </if>
    </sql>
    <sql id="OracleDialectSuffix">
        <if test="record.page != null">
            <!-- 有特殊符号 "<",">="等特殊符,使用 <![CDATA[ ]]> 以防被转义 --> 
            <![CDATA[ ) row_ ) where rownum_ > #{record.page.begin} and rownum_ <= #{record.page.end} ]]>
        </if>
    </sql>

        <!-- 涉及数据表：PSP_TR_MOBILEDEVICE t1，psp_db_org t2，PSP_TR_DEVICE_TYPE t3，PSP_TR_DEVICE_MFRS t4-->
    <!-- 数据表的关联关系
        t1.org_code = t2.org_code
        t1.device_type = t3.code
        t1.manufacturer = t4.mfrs_num
     -->
    <select id="getTrMobileDeviceListPage" parameterType="com.wiscom.module.manualGenerate.pojo.TrMobileDevice" resultType="java.util.Map">
        <!-- 格式: <include refid="namespace.sqlid"/> -->
        <!-- 引用同一个xml中的sql片段 -->
        <include refid="OracleDialectPrefix" />
            select t1.device_id               as "deviceId",
				   t1.device_no               as "deviceNo",
				   t1.device_name             as "deviceName",
				   t1.org_code                as "orgCode",
				   t2.org_name                as "orgName",
				   t1.place_code              as "placeCode",
				   t1.device_type             as "deviceType",
				   t3.name                    as "deviceTypeName",
				   t1.manufacturer            as "manufacturer",
				   t4.mfrs_name               as "manufacturerName",
				   t1.illegal_collection      as "illegalCollection",
        			<!-- decode()将查询结果以其他形式表现出来 详情参考网址 https://www.cnblogs.com/Ant-soldier/p/5635936.html-->
				   decode(t1.illegal_collection, '0', '否', '1', '是') as "illegalCollectionName",
				   t1.pass_vehicle_collection as "passVehicleCollection",
				   decode(t1.pass_vehicle_collection, '0', '否', '1', '是') as "passVehicleCollectionName"
			  from PSP_TR_MOBILEDEVICE t1
			  left join psp_db_org t2
				on t1.org_code = t2.org_code
			  left join PSP_TR_DEVICE_TYPE t3
				on t1.device_type = t3.code
			  left join PSP_TR_DEVICE_MFRS t4
				on t1.manufacturer = t4.mfrs_num
            <where>
				<if test="record.deviceId != null and record.deviceId != ''">
                    <!-- 为了防止因为数据报错，jdbcType将数据类型指定为VARCHAR类型 -->
					and device_id = #{record.deviceId,jdbcType=VARCHAR}
				</if>
                <if test="record.deviceNo != null and record.deviceNo != ''">
                    and DEVICE_NO = #{record.deviceNo,jdbcType=VARCHAR}
                </if>
                <if test="record.deviceName != null and record.deviceName != ''">
                    and DEVICE_NAME like '%' ||  #{record.deviceName,jdbcType=VARCHAR} || '%'
                </if>
                <if test="record.orgCode != null and record.orgCode != ''">
                    and ORG_CODE = #{record.orgCode,jdbcType=VARCHAR}
                </if>
                <if test="record.placeCode != null and record.placeCode != ''">
                    and PLACE_CODE = #{record.placeCode,jdbcType=VARCHAR}
                </if>
                <if test="record.deviceType != null and record.deviceType != ''">
                    and DEVICE_TYPE = #{record.deviceType,jdbcType=VARCHAR}
                </if>
                <if test="record.manufacturer != null and record.manufacturer != ''">
                    and MANUFACTURER = #{record.manufacturer,jdbcType=VARCHAR}
                </if>
                <if test="record.illegalCollection != null and record.illegalCollection != ''">
                    and ILLEGAL_COLLECTION = #{record.illegalCollection,jdbcType=VARCHAR}
                </if>
                <if test="record.passVehicleCollection != null and record.passVehicleCollection != ''">
                    and PASS_VEHICLE_COLLECTION = #{record.passVehicleCollection,jdbcType=VARCHAR}
                </if>
            </where>
        <include refid="OracleDialectSuffix" />
    </select>
```





### decode()

> 将查询结果以其他形式表现出来

> 详情参考网址 
>
> > `https://www.cnblogs.com/Ant-soldier/p/5635936.html`

###### 案例：

```xml
    <select id="getTrMobileDeviceListPage" parameterType="com.wiscom.module.manualGenerate.pojo.TrMobileDevice" resultType="java.util.Map">
        <!-- 格式: <include refid="namespace.sqlid"/> -->
        <!-- 引用同一个xml中的sql片段 -->
        <include refid="OracleDialectPrefix" />
            select t1.device_id               as "deviceId",
				   t1.device_no               as "deviceNo",
				   t1.device_name             as "deviceName",
				   t1.org_code                as "orgCode",
				   t2.org_name                as "orgName",
				   t1.place_code              as "placeCode",
				   t1.device_type             as "deviceType",
				   t3.name                    as "deviceTypeName",
				   t1.manufacturer            as "manufacturer",
				   t4.mfrs_name               as "manufacturerName",
				   t1.illegal_collection      as "illegalCollection",
        			<!-- decode()将查询结果以其他形式表现出来 详情参考网址 https://www.cnblogs.com/Ant-soldier/p/5635936.html-->
				   decode(t1.illegal_collection, '0', '否', '1', '是') as "illegalCollectionName",
				   t1.pass_vehicle_collection as "passVehicleCollection",
				   decode(t1.pass_vehicle_collection, '0', '否', '1', '是') as "passVehicleCollectionName"
			  from PSP_TR_MOBILEDEVICE t1
			  left join psp_db_org t2
				on t1.org_code = t2.org_code
			  left join PSP_TR_DEVICE_TYPE t3
				on t1.device_type = t3.code
			  left join PSP_TR_DEVICE_MFRS t4
				on t1.manufacturer = t4.mfrs_num
            <where>
				<if test="record.deviceId != null and record.deviceId != ''">
                    <!-- 为了防止因为数据报错，jdbcType将数据类型指定为VARCHAR类型 -->
					and device_id = #{record.deviceId,jdbcType=VARCHAR}
				</if>
                <if test="record.deviceNo != null and record.deviceNo != ''">
                    and DEVICE_NO = #{record.deviceNo,jdbcType=VARCHAR}
                </if>
                <if test="record.deviceName != null and record.deviceName != ''">
                    and DEVICE_NAME like '%' ||  #{record.deviceName,jdbcType=VARCHAR} || '%'
                </if>
                <if test="record.orgCode != null and record.orgCode != ''">
                    and ORG_CODE = #{record.orgCode,jdbcType=VARCHAR}
                </if>
                <if test="record.placeCode != null and record.placeCode != ''">
                    and PLACE_CODE = #{record.placeCode,jdbcType=VARCHAR}
                </if>
                <if test="record.deviceType != null and record.deviceType != ''">
                    and DEVICE_TYPE = #{record.deviceType,jdbcType=VARCHAR}
                </if>
                <if test="record.manufacturer != null and record.manufacturer != ''">
                    and MANUFACTURER = #{record.manufacturer,jdbcType=VARCHAR}
                </if>
                <if test="record.illegalCollection != null and record.illegalCollection != ''">
                    and ILLEGAL_COLLECTION = #{record.illegalCollection,jdbcType=VARCHAR}
                </if>
                <if test="record.passVehicleCollection != null and record.passVehicleCollection != ''">
                    and PASS_VEHICLE_COLLECTION = #{record.passVehicleCollection,jdbcType=VARCHAR}
                </if>
            </where>
        <include refid="OracleDialectSuffix" />
    </select>
```





### TO_CHAR()

> 把数据库中的日期类型转换成我们需要的字符串

> 参考网址
>
> > `https://www.cnblogs.com/aipan/p/7941917.html`

###### 案例：

```xml
<select id="illegalReportQuery" parameterType="java.util.HashMap" resultType="java.util.HashMap">
    <include refid="OracleDialectPrefix"/>
    SELECT T1.ILLEGAL_ID AS "illegalId",T1.PLATE_NO AS "plateNo",T1.ERROR_INFO AS "errorInfo",T2.CLASS_NAME AS "className",
    TO_CHAR (T1.ILLEGAL_TIME, 'YYYY-MM-DD HH24:MI:SS') AS "illegalTimeStr",
    TO_CHAR (T1.CREATE_TIME, 'YYYY-MM-DD HH24:MI:SS') AS "createTimeStr",
    TO_CHAR (T1.REPORT_TIME, 'YYYY-MM-DD HH24:MI:SS') AS "reportTimeStr",
    CASE
    WHEN T1.CAPTURE_TYPE='1' THEN (SELECT toll.TOLLGATE_NAME FROM PSP_DB_TOLLGATE toll WHERE T1.TOLLGATE_ID=toll.TOLLGATE_ID)
    WHEN T1.CAPTURE_TYPE='2' THEN (SELECT camera.CAMERA_NAME FROM PSP_DB_CAMERA camera WHERE T1.TOLLGATE_ID=camera.CAMERA_ID)
    WHEN T1.CAPTURE_TYPE='3' THEN (SELECT md.DEVICE_NAME FROM PSP_TR_MOBILEDEVICE md WHERE T1.TOLLGATE_ID=md.DEVICE_NO)
    ELSE '未知设备' END AS "tollgateName",
    T3.TYPE_NO AS "typeNo",
    T3.TYPE_NO||':'||T3.TYPE_NAME AS "typeNameStr",
    T1.REPORT_STATE AS "reportState",
    DECODE(T1.REPORT_STATE,0,'未上报',1,'上报成功',2,'上报失败',3,'作废无效',4,'无效申请中','空值') AS "reportStateStr",
    T4.IMAGE_COUNT "imageCount",T4.IMAGE_STORAGE "imageStorage",T4.IMAGE_ONE "imageOne",T4.IMAGE_TWO "imageTwo",T4.IMAGE_THREE "imageThree",
    T4.IMAGE_FOUR "imageFour",T4.IMAGE_FIVE "imageFive",T4.IMAGE_SIX "imageSix",T4.IMAGE_SEVEN "imageSeven",T4.IMAGE_EIGHT "imageEight",
    DECODE(T5.IMAGE_CONFIG,NULL,NULL,T5.IMAGE_CONFIG) "imageConfig"
    FROM PSP_TR_ILLEGAL_RESPORT T1
    LEFT JOIN PSP_DB_PLATECLASS T2 ON T1.PLATE_CLASS=T2.PLATE_CLASS
    LEFT JOIN PSP_TR_ILLEGAL_TYPE T3 ON T1.ILLEGAL_TYPE=T3.TYPE_NO
    LEFT JOIN PSP_TR_ILLEGAL_EVENT T4 ON T1.ILLEGAL_ID=T4.OBJECT_ID
    LEFT JOIN PSP_TR_ILLEGAL_IMAGECONFIG T5 ON T4.IMAGE_COUNT=T5.IMAGE_NUM
    WHERE 1=1
    <if test="captureType!=null">AND T1.CAPTURE_TYPE=#{captureType}</if>
    <if test="illegalType!=null">AND T1.ILLEGAL_TYPE=#{illegalType}</if>
    <if test="plateClass">AND T1.PLATE_CLASS=#{plateClass}</if>
    <choose>
      <when test="timeType==1">AND T1.CREATE_TIME BETWEEN to_date('${startTime}', 'yyyy-mm-dd hh24:mi:ss') AND to_date('${endTime}', 'yyyy-mm-dd hh24:mi:ss')</when>
      <when test="timeType==2">AND T1.ILLEGAL_TIME BETWEEN to_date('${startTime}', 'yyyy-mm-dd hh24:mi:ss') AND to_date('${endTime}', 'yyyy-mm-dd hh24:mi:ss')</when>
      <otherwise></otherwise>
    </choose>
    <if test="reportState!=null">AND T1.REPORT_STATE=#{reportState}</if>
    <if test="plateNo!=null">AND T1.PLATE_NO like concat(concat('%',#{plateNo}),'%')</if>
    <include refid="OracleDialectSuffix"/>
  </select>
```



### TO_DATE()

> 把我们需要的字符串转换成数据库中的日期类型

> 参考网址
>
> > `https://blog.csdn.net/idomyway/article/details/78785112`

###### 案例：

```xml
  <sql id="timeTypeSql">
    <if test="timeType=='createTime'">
      CREATE_TIME
    </if>
    <if test="timeType=='illegalTime'">
      ILLEGAL_TIME
    </if>
    <if test="timeType=='reportTime'">
      REPORT_TIME
    </if>
  </sql>
<sql id="ymd">
    <if test="countType == 'month'">
      'yyyy-mm'
    </if>
    <if test="countType == 'day'">
      'yyyy-mm-dd'
    </if>
  </sql>
<select id="illegalReportCountByDate" parameterType="java.util.HashMap" resultType="com.wiscom.module.manualGenerate.pojo.IllegalReportCount">
    SELECT t.*,"state0"+"state1"+"state2"+"state3" as "stateall" FROM (
      SELECT timestr as "timestr",decode("0",null,0,"0") as "state0",decode("1",null,0,"1") as "state1",decode("2",null,0,"2") as "state2",decode("3",null,0,"3") as "state3" FROM (
        SELECT TO_CHAR(<include refid="timeTypeSql"/> ,<include refid="ymd"/> ) TIMESTR,REPORT_STATE,COUNT(0) COUNTNUM
        FROM PSP_TR_ILLEGAL_RESPORT
        WHERE <include refid="timeTypeSql"/> BETWEEN TO_DATE(#{startTime},'YYYY-MM-DD HH24-MI-SS') AND TO_DATE(#{endTime},'YYYY-MM-DD HH24-MI-SS')
        GROUP BY TO_CHAR(<include refid="timeTypeSql"/>,<include refid="ymd"/>),REPORT_STATE
      ) PIVOT(SUM(COUNTNUM) FOR REPORT_STATE IN(0,1,2,3))
    )t ORDER BY "timestr"
  </select>
```



### String 与 Blob互相转换

##### Blob转为String输出

> `utl_raw.cast_to_varchar2(dbms_lob.substr(MATERIAL_CONTENT,4000)) AS MATERIAL_CONTENT`

```xml
    <select id="getProgramMaterialList" resultType="com.wiscom.module.autoGenerate.pojo.ProgramMaterial"
            resultMap="ProgramMaterial">
        select
        PROGRAM_MATERIAL_ID,
        MATERIAL_ID,
        MATERIAL_TITLE,
        MATERIAL_TYPE,
        utl_raw.cast_to_varchar2(dbms_lob.substr(MATERIAL_CONTENT,4000)) AS MATERIAL_CONTENT,
        TIME_SPAN,
        PLAN_TYPE,
        TIME_PERIOD_ID
         from PSP_TR_PROGRAM_MATERIAL
        <if test="timePeriodId!=null">
            <where>
                TIME_PERIOD_ID = #{timePeriodId}
            </where>
        </if>
    </select>
```

##### String转为Blob输入

> `TO_BLOB(UTL_RAW.CAST_TO_RAW(#{materialContent}))`

```xml
	<insert id="addProgramMaterial">

<!--        <selectKey keyProperty="programMaterialId" order="BEFORE" resultType="java.math.BigDecimal">-->
<!--            SELECT PSP_TR_PROGRAM_MATERIAL_SEQ.Nextval from DUAL-->
<!--        </selectKey>-->

        insert into PSP_TR_PROGRAM_MATERIAL
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="materialId!=null">
                MATERIAL_ID,
            </if>
            <if test="materialTitle!=null">
                MATERIAL_TITLE,
            </if>
            <if test="materialType!=null">
                MATERIAL_TYPE,
            </if>
            <if test="materialContent!=null">
                MATERIAL_CONTENT,
            </if>
            <if test="timeSpan!=null">
                TIME_SPAN,
            </if>
            <if test="planType!=null">
                PLAN_TYPE,
            </if>
            <if test="timePeriodId!=null">
                TIME_PERIOD_ID
            </if>
        </trim>
        VALUES
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="materialId!=null">
                #{materialId},
            </if>
            <if test="materialTitle!=null">
                #{materialTitle},
            </if>
            <if test="materialType!=null">
                #{materialType},
            </if>
            <if test="materialContent!=null">
                TO_BLOB(UTL_RAW.CAST_TO_RAW(#{materialContent})),
            </if>
            <if test="timeSpan!=null">
                #{timeSpan},
            </if>
            <if test="planType!=null">
                #{planType},
            </if>
            <if test="timePeriodId!=null">
                #{timePeriodId}
            </if>
        </trim>
    </insert>
```