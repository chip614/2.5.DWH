# 2.5.DWH
Задание по 2.5.DWH

1. Для упрощения задачи было создано материализованное представления, собирающее в себя данные о продажах.
   Структура материализованного представления aggr_sales: date (MM.YYYY), shop_name, product_id, sales_fact
   WITH aggr_sales AS (
         SELECT shop_dns.product_id,
            'dns'::text AS shop_name,
            shop_dns.sales_cnt,
            shop_dns.date
           FROM shop_dns
        UNION
         SELECT shop_mvideo.product_id,
            'mvideo'::text AS shop_name,
            shop_mvideo.sales_cnt,
            shop_mvideo.date
           FROM shop_mvideo
        UNION
         SELECT shop_sitilink.product_id,
            'sitilink'::text AS shop_name,
            shop_sitilink.sales_cnt,
            shop_sitilink.date
           FROM shop_sitilink
        )
 SELECT to_char(aggr_sales.date::timestamp with time zone, 'MM.YYYY'::text) AS date_mmyyyy,
    aggr_sales.shop_name,
    aggr_sales.product_id,
    sum(aggr_sales.sales_cnt) AS sales_fact
   FROM aggr_sales
  GROUP BY aggr_sales.product_id, aggr_sales.shop_name, (to_char(aggr_sales.date::timestamp with time zone, 'MM.YYYY'::text))

 2. Запрос для построения витрины:
    WITH pl_pr AS (
	SELECT pl.shop_name, 
		pl.product_id, 
		pr.product_name, 
		pl.plan_cnt as sales_plan, 
		to_char(pl.plan_date, 'MM.YYYY') as plan_date,  
		pr.price
	FROM plan pl LEFT OUTER JOIN products pr ON pl.product_id=pr.product_id)
SELECT pl_pr.plan_date, 
	pl_pr.shop_name, 
	pl_pr.product_name, 
	pl_pr.sales_plan, 
	COALESCE(ags.sales_fact,0), 
	CAST(COALESCE(ags.sales_fact,0) as float)/pl_pr.sales_plan  as sales_fact_sales_plan,
	COALESCE(ags.sales_fact,0)*pl_pr.price as income_fact, 
	pl_pr.sales_plan*pl_pr.price as income_plan,
	CAST(COALESCE(ags.sales_fact,0)*pl_pr.price as float)/pl_pr.sales_plan*pl_pr.price as income_fact_income_plan
FROM pl_pr LEFT OUTER JOIN aggr_sales ags 
	ON pl_pr.plan_date=ags.date_mmyyyy 
	AND pl_pr.product_id=ags.product_id 
	AND pl_pr.shop_name=ags.shop_name
    
