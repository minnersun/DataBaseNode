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



### 根据首字母查询

##### 案例

```SQL
CREATE OR REPLACE FUNCTION FUNC_GETPY(P_NAME IN VARCHAR2) RETURN VARCHAR2 AS
  V_COMPARE VARCHAR2(100);
  V_RETURN  VARCHAR2(4000);
 
  FUNCTION F_NLSSORT(P_WORD IN VARCHAR2) RETURN VARCHAR2 AS
  BEGIN
    RETURN NLSSORT(P_WORD, 'NLS_SORT=SCHINESE_PINYIN_M');
  END;
BEGIN
  FOR I IN 1 .. LENGTH(P_NAME) LOOP
    V_COMPARE := F_NLSSORT(SUBSTR(P_NAME, I, 1));
    IF V_COMPARE >= F_NLSSORT('吖') AND V_COMPARE <= F_NLSSORT('驁') THEN
      V_RETURN := V_RETURN || 'a';
    ELSIF V_COMPARE >= F_NLSSORT('八') AND V_COMPARE <= F_NLSSORT('簿') THEN
      V_RETURN := V_RETURN || 'b';
    ELSIF V_COMPARE >= F_NLSSORT('嚓') AND V_COMPARE <= F_NLSSORT('錯') THEN
      V_RETURN := V_RETURN || 'c';
    ELSIF V_COMPARE >= F_NLSSORT('咑') AND V_COMPARE <= F_NLSSORT('鵽') THEN
      V_RETURN := V_RETURN || 'd';
    ELSIF V_COMPARE >= F_NLSSORT('妸') AND V_COMPARE <= F_NLSSORT('樲') THEN
      V_RETURN := V_RETURN || 'e';
    ELSIF V_COMPARE >= F_NLSSORT('发') AND V_COMPARE <= F_NLSSORT('猤') THEN
      V_RETURN := V_RETURN || 'f';
    ELSIF V_COMPARE >= F_NLSSORT('旮') AND V_COMPARE <= F_NLSSORT('腂') THEN
      V_RETURN := V_RETURN || 'g';
    ELSIF V_COMPARE >= F_NLSSORT('妎') AND V_COMPARE <= F_NLSSORT('夻') THEN
      V_RETURN := V_RETURN || 'h';
    ELSIF V_COMPARE >= F_NLSSORT('丌') AND V_COMPARE <= F_NLSSORT('攈') THEN
      V_RETURN := V_RETURN || 'j';
    ELSIF V_COMPARE >= F_NLSSORT('咔') AND V_COMPARE <= F_NLSSORT('穒') THEN
      V_RETURN := V_RETURN || 'k';
    ELSIF V_COMPARE >= F_NLSSORT('垃') AND V_COMPARE <= F_NLSSORT('擽') THEN
      V_RETURN := V_RETURN || 'l';
    ELSIF V_COMPARE >= F_NLSSORT('嘸') AND V_COMPARE <= F_NLSSORT('椧') THEN
      V_RETURN := V_RETURN || 'm';
    ELSIF V_COMPARE >= F_NLSSORT('拏') AND V_COMPARE <= F_NLSSORT('瘧') THEN
      V_RETURN := V_RETURN || 'n';
    ELSIF V_COMPARE >= F_NLSSORT('筽') AND V_COMPARE <= F_NLSSORT('漚') THEN
      V_RETURN := V_RETURN || 'o';
    ELSIF V_COMPARE >= F_NLSSORT('妑') AND V_COMPARE <= F_NLSSORT('曝') THEN
      V_RETURN := V_RETURN || 'p';
    ELSIF V_COMPARE >= F_NLSSORT('七') AND V_COMPARE <= F_NLSSORT('裠') THEN
      V_RETURN := V_RETURN || 'q';
    ELSIF V_COMPARE >= F_NLSSORT('亽') AND V_COMPARE <= F_NLSSORT('鶸') THEN
      V_RETURN := V_RETURN || 'r';
    ELSIF V_COMPARE >= F_NLSSORT('仨') AND V_COMPARE <= F_NLSSORT('蜶') THEN
      V_RETURN := V_RETURN || 's';
    ELSIF V_COMPARE >= F_NLSSORT('侤') AND V_COMPARE <= F_NLSSORT('籜') THEN
      V_RETURN := V_RETURN || 't';
    ELSIF V_COMPARE >= F_NLSSORT('屲') AND V_COMPARE <= F_NLSSORT('鶩') THEN
      V_RETURN := V_RETURN || 'w';
    ELSIF V_COMPARE >= F_NLSSORT('夕') AND V_COMPARE <= F_NLSSORT('鑂') THEN
      V_RETURN := V_RETURN || 'x';
    ELSIF V_COMPARE >= F_NLSSORT('丫') AND V_COMPARE <= F_NLSSORT('韻') THEN
      V_RETURN := V_RETURN || 'y';
    ELSIF V_COMPARE >= F_NLSSORT('帀') AND V_COMPARE <= F_NLSSORT('咗') THEN
      V_RETURN := V_RETURN || 'z';
    END IF;
  END LOOP;
  RETURN V_RETURN;
END;
```

```xml
    <select id="getSingalCrossList" parameterType="java.lang.String" resultType="java.util.Map">
        select CROSS_ID "crossId",
        from PSP_TR_SIGNALCROSS
        <if test="_parameter != null and _parameter !=''">
            where CROSS_NAME LIKE #{singalCrossName} or FUNC_GETPY(CROSS_NAME) LIKE #{singalCrossName}
        </if>
    </select>
```





### 对查询结果新增临时列排序，一行分为多行

> 临时新增临时排序列
>
> > `rownum`
>
> 一行分为多行
>
> > `select REGEXP_SUBSTR((t2."id"), '[^分隔符吖,]+', 1, ROWNUM) AS "id"`
> > `CONNECT BY ROWNUM  <= LENGTH(t2."id") - LENGTH(REPLACE(t2."id", '分隔符吖,', '')) + 1`



```sql

 select t5.* from
 (
 select rownum, t4.* from (
            select REGEXP_SUBSTR((t2."id"), '[^分隔符吖,]+', 1, ROWNUM) AS "id"
            from
            (
            select
            WM_CONCAT(t1."id") "id",
            WM_CONCAT(t1."plateClass") "plateClass",
            WM_CONCAT(t1."className") "className",
            WM_CONCAT(t1."remarks") "remarks",
            WM_CONCAT(t1."inputUser") "inputUser",
            WM_CONCAT(t1."inputUserRealName") "inputUserRealName",
            WM_CONCAT(t1."inputTime") "inputTime",
            WM_CONCAT(t1."inputTimeStr") "inputTimeStr",
            WM_CONCAT(t1."state") "state",
            WM_CONCAT(t1."approvalUser1") "approvalUser1",
            WM_CONCAT(t1."approvalUser1RealName") "approvalUser1RealName",
            WM_CONCAT(t1."approvalTime1") "approvalTime1",
            WM_CONCAT(t1."approvalTime1Str") "approvalTime1Str",
            WM_CONCAT(t1."approvalUser2") "approvalUser2",
            WM_CONCAT(t1."approvalUser2RealName") "approvalUser2RealName",
            WM_CONCAT(t1."approvalTime2") "approvalTime2",
            WM_CONCAT(t1."approvalTime2Str") "approvalTime2Str",
            WM_CONCAT(t1."plateColor") "plateColor",
            WM_CONCAT(t1."unitName") "unitName",
            WM_CONCAT(t1."contacrPerson") "contacrPerson",
            WM_CONCAT(t1."phone") "phone",
            WM_CONCAT(t1."termOfValidity") "termOfValidity",
            WM_CONCAT(t1."isAllIllegalType") "isAllIllegalType",
            WM_CONCAT(t1."illegalType") "illegalType"
            from
            (
            select count(1) num,xmlagg(xmlparse(content TO_CHAR(T1.ID) || '分隔符吖' wellformed) order by T1.ID
            ).getclobval() "id",
            xmlagg(xmlparse(content T1.PLATE_CLASS || '分隔符吖' wellformed) order by T1.ID).getclobval() "plateClass",
            xmlagg(xmlparse(content T2.CLASS_NAME || '分隔符吖' wellformed) order by T1.ID).getclobval() "className",
            xmlagg(xmlparse(content T1.REMARKS || '分隔符吖' wellformed) order by T1.ID).getclobval() "remarks",
            xmlagg(xmlparse(content T1.INPUT_USER || '分隔符吖' wellformed) order by T1.ID).getclobval() "inputUser",
            xmlagg(xmlparse(content T3.REAL_NAME || '分隔符吖' wellformed) order by T1.ID).getclobval() "inputUserRealName",
            xmlagg(xmlparse(content TO_CHAR(T1.INPUT_TIME,'YYYY-MM-DD') || '分隔符吖' wellformed) order by
            T1.ID).getclobval() "inputTime",
            xmlagg(xmlparse(content TO_CHAR(T1.INPUT_TIME,'YYYY-MM-DD HH24:MI:SS') || '分隔符吖' wellformed) order by
            T1.ID).getclobval() "inputTimeStr",
            xmlagg(xmlparse(content T1.STATE || '分隔符吖' wellformed) order by T1.ID).getclobval() "state",
            xmlagg(xmlparse(content T1.APPROVAL_USER1 || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "approvalUser1",
            xmlagg(xmlparse(content T4.REAL_NAME || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "approvalUser1RealName",
            xmlagg(xmlparse(content T1.APPROVAL_TIME1 || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "approvalTime1",
            xmlagg(xmlparse(content TO_CHAR(T1.APPROVAL_TIME1,'YYYY-MM-DD HH24:MI:SS') || '分隔符吖' wellformed) order by
            T1.ID).getclobval() "approvalTime1Str",
            xmlagg(xmlparse(content T1.APPROVAL_USER2 || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "approvalUser2",
            xmlagg(xmlparse(content T5.REAL_NAME || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "approvalUser2RealName",
            xmlagg(xmlparse(content T1.APPROVAL_TIME2 || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "approvalTime2",
            xmlagg(xmlparse(content TO_CHAR(T1.APPROVAL_TIME2,'YYYY-MM-DD HH24:MI:SS') || '分隔符吖' wellformed) order by
            T1.ID).getclobval() "approvalTime2Str",
            xmlagg(xmlparse(content t1.PLATE_COLOR || '分隔符吖' wellformed) order by T1.ID).getclobval() "plateColor",
            xmlagg(xmlparse(content t1.UNIT_NAME || '分隔符吖' wellformed) order by T1.ID).getclobval() "unitName",
            xmlagg(xmlparse(content t1.CONTACR_PERSON || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "contacrPerson",
            xmlagg(xmlparse(content t1.PHONE || '分隔符吖' wellformed) order by T1.ID).getclobval() "phone",
            xmlagg(xmlparse(content TO_CHAR(t1.TERM_OF_VALIDITY,'YYYY-MM-DD HH24:MI:SS') || '分隔符吖' wellformed) order by
            T1.ID).getclobval() "termOfValidity",
            xmlagg(xmlparse(content t1.IS_ALL_ILLEGAL_TYPE || '分隔符吖' wellformed) order by T1.ID).getclobval()
            "isAllIllegalType",
            xmlagg(xmlparse(content t1.ILLEGAL_TYPE || '分隔符吖' wellformed) order by T1.ID).getclobval() "illegalType",

            T1.PLATE_NO "plateNo"
            FROM PSP_TR_ILLEGAL_WHITEVEHICLE T1
            LEFT JOIN PSP_DB_PLATECLASS T2 ON T1.PLATE_CLASS=T2.PLATE_CLASS
            LEFT JOIN PSP_APP_USER T3 ON T1.INPUT_USER=T3.USER_ID
            LEFT JOIN PSP_APP_USER T4 ON T1.APPROVAL_USER1=T4.USER_ID
            LEFT JOIN PSP_APP_USER T5 ON T1.APPROVAL_USER2=T5.USER_ID
            group by T1.PLATE_NO
            order by num desc
            ) t1
            ) t2
            CONNECT BY ROWNUM  <= LENGTH(t2."id") - LENGTH(REPLACE(t2."id", '分隔符吖,', '')) + 1
            ) t3
            left join
            (
            SELECT T1.ID "id",
            T1.PLATE_NO "plateNo",
            T1.PLATE_CLASS "plateClass",
            T2.CLASS_NAME "className",
            T1.REMARKS "remarks",
            T1.INPUT_USER "inputUser",
            T3.REAL_NAME "inputUserRealName",
            T1.INPUT_TIME "inputTime",
            TO_CHAR(T1.INPUT_TIME,'YYYY-MM-DD HH24:MI:SS') "inputTimeStr",
            T1.STATE "state",
            T1.APPROVAL_USER1 "approvalUser1",
            T4.REAL_NAME "approvalUser1RealName",
            T1.APPROVAL_TIME1 "approvalTime1",
            TO_CHAR(T1.APPROVAL_TIME1,'YYYY-MM-DD HH24:MI:SS') "approvalTime1Str",
            T1.APPROVAL_USER2 "approvalUser2",
            T5.REAL_NAME "approvalUser2RealName",
            T1.APPROVAL_TIME2 "approvalTime2",
            TO_CHAR(T1.APPROVAL_TIME2,'YYYY-MM-DD HH24:MI:SS') "approvalTime2Str",
            t1.PLATE_COLOR "plateColor",
            t1.UNIT_NAME "unitName",
            t1.CONTACR_PERSON "contacrPerson",
            t1.PHONE "phone",
            TO_CHAR(t1.TERM_OF_VALIDITY,'YYYY-MM-DD HH24:MI:SS') "termOfValidity",
            t1.IS_ALL_ILLEGAL_TYPE "isAllIllegalType",
            t1.ILLEGAL_TYPE "illegalType"
            FROM PSP_TR_ILLEGAL_WHITEVEHICLE T1
            LEFT JOIN PSP_DB_PLATECLASS T2 ON T1.PLATE_CLASS=T2.PLATE_CLASS
            LEFT JOIN PSP_APP_USER T3 ON T1.INPUT_USER=T3.USER_ID
            LEFT JOIN PSP_APP_USER T4 ON T1.APPROVAL_USER1=T4.USER_ID
            LEFT JOIN PSP_APP_USER T5 ON T1.APPROVAL_USER2=T5.USER_ID
            ) t4
            on t3."id" = t4."id"
            where t3."id" is not null						 
						order by rownum 
 ) t5
```