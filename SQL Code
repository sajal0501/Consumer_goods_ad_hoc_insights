SELECT * FROM gdb023.dim_customer;


/* Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region */


select 
	market 
from dim_customer
where 
	customer = "Atliq Exclusive"
And
	region = "APAC"
group by market
order by market;
    
--------------------------------------------------------------------------------------------
    
/*What is the percentage of unique product increase in 2021 vs. 2020? The
final output contains these fields,
unique_products_2020
unique_products_2021
percentage_chg*/

select * from dim_product;
select * from fact_gross_price;
select * from fact_sales_monthly;


with uniq_pro_2020 as (
					select 
						count(distinct(product_code) ) as unique_product_2020
					from fact_sales_monthly
					where fiscal_year = 2020) ,

uniq_pro_2021 as (
					select 
						count(distinct(product_code) ) as unique_product_2021
					from fact_sales_monthly
					where fiscal_year = 2021)

select unique_product_2020 ,unique_product_2021,
round(((unique_product_2021 - unique_product_2020) / unique_product_2020)*100,2) as percent_chg
from uniq_pro_2020 
cross join uniq_pro_2021 ;

----------------------------------------------------------------------------------

/* 3. Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts. The final output contains
2 fields,
segment
product_count
*/
select * from dim_product;

select segment,
count(product) as product_cnt
from dim_product
group by segment
order by  product_cnt desc;

--------------------------------------------------------------------------------------------------------
/*Follow-up: Which segment had the most increase in unique products in
2021 vs 2020? The final output contains these fields,
segment
product_count_2020
product_count_2021
difference*/

select * from fact_sales_monthly;
select * from dim_product;



with ct1 as (
		select 
            a.segment ,
			count(distinct(a.product_code )) as product_cnt_2020,
			b.fiscal_year
		from dim_product as a
		join fact_sales_monthly as b
		using(product_code)
		where b.fiscal_year = 2020
		group by a.segment
) ,

ct2 as (
		select 
			a.segment ,
			count(distinct(a.product_code )) as product_cnt_2021,
			b.fiscal_year
		from dim_product as a
		join fact_sales_monthly as b
		using(product_code)
		where b.fiscal_year = 2021
		group by a.segment
)
select 
	ct2.segment ,
    ct1.product_cnt_2020, ct2.product_cnt_2021,
    (ct2.product_cnt_2021 - ct1.product_cnt_2020) as difference
from ct1
join ct2
using (segment)
group by segment
order by difference desc;


--------------------------------------------------------------------------------------------------------
/*5. Get the products that have the highest and lowest manufacturing costs.
The final output should contain these fields,
product_code
product
manufacturing_cost*/

select * from fact_manufacturing_cost ;
select * from dim_product;

select 
	a.product_code,
	a.product ,
    round(b.manufacturing_cost,2) as manufacturing_cost
from dim_product as a
join fact_manufacturing_cost as b
on a.product_code = b.product_code
where manufacturing_cost in (
						(select max(manufacturing_cost ) from fact_manufacturing_cost),
						(select min(manufacturing_cost) from fact_manufacturing_cost)
                            )
                            order by manufacturing_cost desc;


---------------------------------------------------------------------------------------------------------------

/* Generate a report which contains the top 5 customers who received an
average high pre_invoice_discount_pct for the fiscal year 2021 and in the
Indian market. The final output contains these fields,
customer_code
customer
average_discount_percentage
*/

select * from fact_pre_invoice_deductions;
select * from dim_customer;


select 
	a.customer_code ,
	a.customer,
    round(avg(b.pre_invoice_discount_pct),4) as average_discount_percentage
from dim_customer as a
join fact_pre_invoice_deductions as b
on a.customer_code = b.customer_code
where b.fiscal_year = 2021 and a.market = "India"
group by  a.customer_code , a.customer
order by average_discount_percentage desc
limit 5;
    

----------------------------------------------------------------------------------------------------------

/* Get the complete report of the Gross sales amount for the customer “Atliq
Exclusive” for each month. This analysis helps to get an idea of low and
high-performing months and take strategic decisions.
The final report contains these columns:
Month
Year
Gross sales Amount
*/

select * from fact_gross_price;
select * from dim_customer ;
select * from fact_sales_monthly;


SELECT MONTHNAME(date) AS Month,
	   YEAR(date) AS Year,
       SUM((Sold_quantity * gross_price)/1000000) AS Gross_sales_Amount
FROM   gdb023.fact_sales_monthly s 
JOIN   gdb023.fact_gross_price g 
ON     s.fiscal_year = g.fiscal_year AND
       s.product_code = g.product_code
JOIN   gdb023.dim_customer c 
ON     s.customer_code = c.customer_code        
WHERE customer = 'Atliq Exclusive'
GROUP BY Month, Year  
ORDER BY  Year  ;


-----------------------------------------------------------------------------------------
/*In which quarter of 2020, got the maximum total_sold_quantity? The final
output contains these fields sorted by the total_sold_quantity,
Quarter
total_sold_quantity
*/

select * from fact_sales_monthly;

select
	Month(date) as Quarter,
    sum(sold_quantity) as total_sold_quantity
from fact_sales_monthly
where fiscal_year = 2020
group by quarter
order by total_sold_quantity desc;

--------------------------------------------------

select 
      case
			when month(date) in (9,10,11) then "Q1"
            when month(date) in (12,1,2) then "Q2"
            when month(date) in (3,4,5) then "Q3"
            else 
            "Q4"
	 End as Quarter,
			sum(sold_quantity) as total_sold_quantity
from fact_sales_monthly
where fiscal_year = 2020
group by Quarter 
order by total_sold_quantity desc;

------------------------------------------------------------------------------------

/* Which channel helped to bring more gross sales in the fiscal year 2021
and the percentage of contribution? The final output contains these fields,
channel
gross_sales_mln
percentage */

select * from fact_sales_monthly;
select * from fact_gross_price;
select * from dim_customer;


with gross_sale as (
select 
	c.channel ,
    round(sum(fg.gross_price * fs.sold_quantity/1000000),2) as gross_sales_mln
from fact_sales_monthly as fs
join dim_customer as c
on c.customer_code = fs.customer_code
join fact_gross_price as fg
on fs.product_code = fg.product_code
where fg.fiscal_year = 2021
group by channel
order by gross_sales_mln desc
)
select 
	channel ,
    concat(gross_sales_mln , ' M') as gross_sales_mln,
	gross_sales_mln * 100 /sum(gross_sales_mln) over() as pct
    from gross_sale ;
    
---------------------------------------------------------------------------------------------------------

/*Get the Top 3 products in each division that have a high
total_sold_quantity in the fiscal_year 2021? The final output contains these fields,
division
product_code
product
total_sold_quantity
rank_order */

select * from fact_sales_monthly;
select * from dim_product ;

with RankedProduct as (
Select 
	p.division ,
    p.product ,
    p.product_code ,
    sum(fs.sold_quantity) as total_sold_quantity ,
    row_number() over( partition by p.division order by sum(fs.sold_quantity) desc) as product_rank
from fact_sales_monthly as fs
join dim_product as p
on p.product_code = fs.product_code
where fs.fiscal_year = 2021 
group by p.product_code , p.division , p.product

) 
select division , product_code ,product ,  total_sold_quantity , product_rank
from RankedProduct 
where product_rank <=3 ;
