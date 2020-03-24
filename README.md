# Hello_World
Test
SELECT
	 "DATE_CODE",
	 "BUSI_DEPT_CODE",
	 "BUSI_DEPT_NAME",
	 "RETAIL_CENTER_CODE",
	 "RETAIL_CENTER_NAME",
	 "CENTER_PERSON_LIABLE_CODE",
	 "CENTER_PERSON_LIABLE_NAME",
	 "DEALER_ID",
	 "DEALER_CODE",
	 "DEALER_NAME",
	 "DEALER_BOSS_CODE",
	 "DEALER_BOSS_NAME",
	 "EXTENSION_FLAG",
	 "LOAD_IN_TAX_MONEY",
	 "LOAD_OUT_TAX_MONEY",
	 "DEALER_DATA_SOURCE" 
FROM "DWI"."SYS_EXAMINE_QUOTA_EXTENSION" ) T3 
RIGHT JOIN /*只取2019年4月1日到昨天范围内的周日和昨天对应日期*/ 
   ( SELECT
	      "DATE_CODE",
	      "WEEK_DAY" 
		 FROM "DWI"."DIM_DATE" 
		 WHERE "DATE_CODE">=TO_DATE('20190401','yyyyMMdd') 
           AND "DATE_CODE"<TO_DATE(TO_CHAR(NOW(), 'YYYY-MM-DD'),'YYYY-MM-DD') 
			     AND ("WEEK_DAY"=0 OR "DATE_CODE"=TO_DATE(add_days(CURRENT_DATE ,-1),'YYYY-MM-DD')) ) T2 ON (T3."DATE_CODE" = T2."DATE_CODE")
    ) T7 
	 LEFT JOIN /*从2019年4月1日开始累计至各周日或昨天对应数值(累计采购额、累计零售额、累计销售出库金额)*/ 
   ( SELECT
	      T3."BUSI_DEPT_CODE" AS "事业部编码",
	      T3."RETAIL_CENTER_CODE" AS "运营中心编码",
	      T3."CENTER_PERSON_LIABLE_CODE" AS "中心经理工号",
      	T3."DEALER_ID" AS "经营区域ID",
	      T3."DEALER_CODE" AS "经营区域编码",
	      T3."DEALER_BOSS_CODE" AS "事业伙伴编码",
      	SUM(T3."PUR_TAX_MONEY") AS "累计采购金额",
        SUM(T3."DEAL_PRICE") AS "累计零售金额",
	      SUM(T3."SALE_OUT_MONEY") AS "销售出库金额",
	      T4."DATE_CODE" AS "日期" 
		    FROM 
           ( SELECT
	              "DATE_CODE",
	              "BUSI_DEPT_CODE",
	              "RETAIL_CENTER_CODE",
	              "CENTER_PERSON_LIABLE_CODE",
	              "DEALER_ID",
	              "DEALER_CODE",
	              "DEALER_BOSS_CODE",
	              "PUR_TAX_MONEY",
	              "DEAL_PRICE",
	              "SALE_OUT_MONEY" 
			      FROM "DWI"."SYS_EXAMINE_QUOTA_EXTENSION" ) T3 
		     RIGHT JOIN /*只取2019年4月1日到昨天范围内的周日和昨天对应日期*/ 
           ( SELECT
	              "DATE_CODE",
	              "WEEK_DAY" 
			       FROM "DWI"."DIM_DATE" 
			       WHERE "DATE_CODE">=TO_DATE('20190401','yyyyMMdd') 
			             AND "DATE_CODE"<TO_DATE(TO_CHAR(NOW(),'YYYY-MM-DD'),'YYYY-MM-DD') 
			             AND ("WEEK_DAY"=0 OR "DATE_CODE"=TO_DATE(add_days(CURRENT_DATE,-1),'YYYY-MM-DD')) 
            ) T4 
          ON (T3."DATE_CODE" <= T4."DATE_CODE") 
	 GROUP BY T3."BUSI_DEPT_CODE",
	 T3."RETAIL_CENTER_CODE",
	 T3."CENTER_PERSON_LIABLE_CODE",
	 T3."DEALER_ID",
	 T3."DEALER_CODE",
	 T3."DEALER_BOSS_CODE",
	 T4."DATE_CODE" ) T8 
   ON ( T7."日期"=T8."日期" 
		    AND T7."事业部编码"=T8."事业部编码" 
		    AND T7."运营中心编码"=T8."运营中心编码" 
		    AND T7."中心经理工号"=T8."中心经理工号" 
		    AND T7."经营区域ID"=T8."经营区域ID" 
		    AND T7."经营区域编码"=T8."经营区域编码" 
		    AND T7."事业伙伴编码"=T8."事业伙伴编码" ) 
	 LEFT JOIN ( /*从2019年4月1日开始累计至各周日或昨天对应的一个月前日期的数值(累计至一个月前的零售额)*/ SELECT
	 T5."BUSI_DEPT_CODE" AS "事业部编码",
	 T5."RETAIL_CENTER_CODE" AS "运营中心编码",
	 T5."CENTER_PERSON_LIABLE_CODE" AS "中心经理工号",
	 T5."DEALER_ID" AS "经营区域ID",
	 T5."DEALER_CODE" AS "经营区域编码",
	 T5."DEALER_BOSS_CODE" AS "事业伙伴编码",
	 SUM(T5."DEAL_PRICE") AS "累计至上月零售额",
	 T6."DATE_CODE" AS "日期" 
		FROM ( SELECT
	 "DATE_CODE",
	 "BUSI_DEPT_CODE",
	 "RETAIL_CENTER_CODE",
	 "CENTER_PERSON_LIABLE_CODE",
	 "DEALER_ID",
	 "DEALER_CODE",
	 "DEALER_BOSS_CODE",
	 "DEAL_PRICE" 
			FROM "DWI"."SYS_EXAMINE_QUOTA_EXTENSION" ) T5 
		RIGHT JOIN /*只取2019年4月1日到昨天范围内的周日和昨天对应日期*/ ( SELECT
	 "DATE_CODE",
	 "WEEK_DAY",
	 TO_DATE("LAST_MONTH_DAY",
	 'yyyyMMdd') AS "LAST_MONTH_DATE_CODE" 
			FROM "DWI"."DIM_DATE" 
			WHERE "DATE_CODE">=TO_DATE('20190401',
	 'yyyyMMdd') 
			AND "DATE_CODE"<TO_DATE(TO_CHAR(NOW(),
	 'YYYY-MM-DD'),
	 'YYYY-MM-DD') 
			AND ("WEEK_DAY"=0 
				OR "DATE_CODE"=TO_DATE(add_days(CURRENT_DATE ,
	 -1),
	 'YYYY-MM-DD')) ) T6 ON (T5."DATE_CODE" <= T6."LAST_MONTH_DATE_CODE") 
		GROUP BY T5."BUSI_DEPT_CODE",
	 T5."RETAIL_CENTER_CODE",
	 T5."CENTER_PERSON_LIABLE_CODE",
	 T5."DEALER_ID",
	 T5."DEALER_CODE",
	 T5."DEALER_BOSS_CODE",
	 T6."DATE_CODE" ) T9 ON ( T8."日期"=T9."日期" 
		AND T8."事业部编码"=T9."事业部编码" 
		AND T8."运营中心编码"=T9."运营中心编码" 
		AND T8."中心经理工号"=T9."中心经理工号" 
		AND T8."经营区域ID"=T9."经营区域ID" 
		AND T8."经营区域编码"=T9."经营区域编码" 
		AND T8."事业伙伴编码"=T9."事业伙伴编码" ) ) T10 
LEFT JOIN /*只取参与考核的DEALER_ID*/ ( SELECT
	 "DEALER_ID" AS "经营区域ID(不考核)",
	 "BUSI_DEPT_CODE" AS "事业部编码（不考核）" 
	FROM "HANABI"."MANU_SKIP_CHECK_DEALER" ) T11 ON ( T10."经营区域ID"=T11."经营区域ID(不考核)" 
	and T10."事业部编码"= T11."事业部编码（不考核）") 
WHERE T11."经营区域ID(不考核)"
