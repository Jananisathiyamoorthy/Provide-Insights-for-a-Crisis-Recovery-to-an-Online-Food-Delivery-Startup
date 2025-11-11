# Provide-Insights-for-a-Crisis-Recovery-to-an-Online-Food-Delivery-Startup

QuickBite Express is a food-tech startup  launched in 2020, that links customers with local restaurants and cloud kitchens. In June 2025, the company encountered a serious crisis when a viral social media post revealed food safety issues at some partner restaurants, alongside a week-long delivery disruption caused by the monsoon season. This led to strong customer backlash and disrupted QuickBite’s operations significantly.

This project has been tasked with analysing the QuickBite dataset and 
providing actionable insights to guide the turnaround strategy.



# Objectives:-

1.Database Setup:

Design and initialize the Library Management System database, including tables for customers, delivery_partner, delivery_performance, menu_item, order_item, ratings, restaurant, orders.
               
2.Primary Analysis (Based on Available data): 

1. Monthly Orders: Compare total orders across pre-crisis (Jan–May 2025) vs crisis 
(Jun–Sep 2025). How severe is the decline? 
2. Which top 5 city groups experienced the highest percentage decline in orders 
during the crisis period compared to the pre-crisis period? 
3. Among restaurants with at least 50 pre-crisis orders, which top 10 high-volume 
restaurants experienced the largest percentage decline in order counts during 
the crisis period? 
4. Cancellation Analysis: What is the cancellation rate trend pre-crisis vs crisis, 
and which cities are most affected?
5. Revenue Impact: Estimate revenue loss from pre-crisis vs crisis (based on 
subtotal, discount, and delivery fee). 

# Project Structure

## 1. Database Setup

<img width="1365" height="994" alt="data setup" src="https://github.com/user-attachments/assets/04ace877-4d84-493c-9c6c-800ec571b22a" />

Database Creation: Created a database named quickbite.

Table Creation: Created tables for customers, delivery_partner, delivery_performance, menu_item, order_item, ratings, restaurant, orders. Each table includes relevant columns and relationships.   
               
```sql
--database created quickbite

create table customers (customer_id varchar(15) primary key,
                        signup_date date,
						city varchar(20),
						acquisition_channel varchar(15))

select to_char(current_timestamp, 'HH24:MI:SS');

create table orders(order_id varchar (20) primary key,
                    customer_id varchar (20),    --fk
					restaurant_id varchar (20),  --fk
					delivery_partner_id varchar (20), --fk
					order_timestamp time,
					subtotal_amount float,
					discount_amount float,
					delivery_fee float,
					total_amount float,
					is_cod char(1) check(is_cod in('Y','N')),
					is_cancelled char(1) check(is_cancelled in('Y','N')))

create table order_items (order_id varchar(20),--fk
                          item_id varchar(20),--fk
						  menu_item_id varchar(20), --fk
						  restaurant_id varchar(20),--fk
						  quantity int,
						  unit_price float,
						  item_discount float,
						  line_total float)

create  table restaurant(restaurant_id varchar(20)primary key,
              restaurant_name varchar(32),
			  city varchar(20),
			  cuisine_type varchar(20),
			  partner_type varchar(20),
			  avg_prep_time_min varchar(5) check(avg_prep_time_min in('26-40','16-25','<=15','>40')),
			  is_active  char(1) check(is_active in('Y','N')))

create table menu_item (menu_item_id varchar(20) primary key,
                        restaurant_id varchar(20),   --fk
						item_name varchar(30),
						category varchar(20),
					    is_veg char(1) check(is_veg in('Y','N')),
						price float)
						
create table delivery_partner(delivery_partner_id varchar (20)primary key,
                              partner_name varchar(30),
							  city varchar(20),
							  vehicle_type varchar(20),
							  employment_type varchar(20),
							  avg_rating float,
							  is_active char(1) check(is_active in('Y','N'))
							  )

create table delivery_performance (order_id varchar(20) ,  --fk
                                    actual_delivery_time_mins int,
									expected_delivery_time_mins int,
									distance_km float)

create table ratings(order_id varchar(20),--fk
                     customer_id varchar(20),--fk
					 restaurant_id varchar(20),--fk
					 rating int,
					 review_text varchar(30),
					 review_timestamp time,
					 sentiment_score float)

alter table orders
add constraint fk_customers
foreign key(customer_id)
references customers(customer_id);

alter table ratings
add constraint fk_customers_rating
foreign key(customer_id)
references customers(customer_id);

alter table order_items
add constraint fk_orders_item
foreign key(order_id)
references orders(order_id);

alter table delivery_performance
add constraint fk_orders_delivery_performance
foreign key(order_id)
references orders(order_id);

alter table ratings
add constraint fk_orders_rating
foreign key(order_id)
references orders(order_id);

alter table orders
add constraint fk_restaurant
foreign key(restaurant_id)
references restaurant(restaurant_id);

alter table order_items
add constraint fk_restaurant_order_items
foreign key(restaurant_id)
references restaurant(restaurant_id);


alter table menu_item
add constraint fk_restaurant_menu_item
foreign key(restaurant_id)
references restaurant(restaurant_id);

alter table ratings
add constraint fk_restaurant_ratings
foreign key(restaurant_id)
references restaurant(restaurant_id);

alter table orders
add constraint fk_delivery_partner
foreign key(delivery_partner_id)
references delivery_partner(delivery_partner_id);

alter table order_items
add constraint fk_menu_item
foreign key(menu_item_id)
references menu_item (menu_item_id);


select * from customers
select * from delivery_partner
select * from delivery_performance
select * from menu_item
select * from order_items
select * from ratings
select * from restaurant
select * from orders
```

## 1 Monthly Orders: Compare total orders across pre-crisis (Jan–May 2025) vs crisis (Jun–Sep 2025). How severe is the decline?
```sql
create table pre_crisis
as
select 
       c.customer_id,
	   c.signup_date,
	   o.order_id,
	   o.total_amount,
	   c.city,
	   case
	         when c.signup_date between '2025-01-01' and '2025-05-31' then 'pre_crisis'
	   	     when c.signup_date between '2025-06-01' and '2025-09-30' then 'crisis'
             else 'other'
		end period
	  from customers as c
join orders as o
on
o.customer_id=c.customer_id

select * from pre_crisis
where period in('pre_crisis','crisis')

select count(order_id) from pre_crisis    
where period ='pre_crisis'                        --96321
select count(order_id) from pre_crisis    
where period ='crisis'                                                             --13168

```

## 2. Which top 5 city groups experienced the highest percentage decline in orders during the crisis period compared to the pre-crisis period? 

```sql
create table top5_crisis
as
select 
         count(order_id) as total_orders,city,period
from pre_crisis
where period ='crisis'

group by city,period
order by city,period

create table top5_precrisis
as
select 
         count(order_id) as total_orders,city,period
from pre_crisis
where period ='pre_crisis'
group by city,period
order by city,period

select * from top5_crisis
select * from top5_precrisis


select  
       cr.city,
	   pre.total_orders as precrisisorders,
	   cr.total_orders as crisisorders
	   from 
top5_precrisis as pre
join top5_crisis as cr
on pre.city=cr.city

--to find decline

select  
       cr.city,
	   pre.total_orders as precrisisorders,
	   cr.total_orders as crisisorders,
	   round (((pre.total_orders-cr.total_orders)::numeric/pre.total_orders) * 100,2) as percentage_decline                      
	   from 
top5_precrisis as pre
join top5_crisis as cr
on pre.city=cr.city

--top5 decline

select  
       cr.city,
	   pre.total_orders as precrisisorders,
	   cr.total_orders as crisisorders,
	   round (((pre.total_orders-cr.total_orders)::numeric/pre.total_orders) * 100,2) as percentage_decline                      
	   from 
top5_precrisis as pre
join top5_crisis as cr
on pre.city=cr.city
order by percentage_decline desc
limit 5
```

## 3. Among restaurants with at least 50 pre-crisis orders, which top 10 high-volume restaurants experienced the largest percentage decline in order counts during the crisis period? 

```sql
--1st join resturant and order
create table res_order
as
select 
      o.order_id,
	  o.customer_id,
	  o.is_cancelled,
      res.restaurant_id,
	  res.restaurant_name,
	  res.city
from restaurant as res
join orders as o
on res.restaurant_id=o.restaurant_id
select * from res_order
--join res_order with pre_Crisis tables

create table top10 
as
select 
       r.restaurant_id,
	   r.restaurant_name,
	   pre.order_id,
	   pre.period
from pre_crisis as pre
join res_order as r
on pre.order_id=r.order_id
--
create table crisis_restaurant
as
select 
         count(order_id) as total_orders,
		 restaurant_name,
		 restaurant_id,
		 period
from top10
where period ='crisis'
group by restaurant_name,restaurant_id,period

create table precrisis_restaurant
as
select 
         count(order_id) as total_orders,
		 restaurant_name,
		 restaurant_id,
		 period
from top10
where period ='pre_crisis'
group by restaurant_name,restaurant_id,period

--top 10 high-volume 
--restaurants experienced the largest percentage decline
select 
           preres.total_orders as precrisis_total_orders,
		   preres.restaurant_name,
		   cres.restaurant_id,
		   cres.total_orders as crisis_total_orders,
		   round (((preres.total_orders-cres.total_orders)::numeric/preres.total_orders) * 100,2) as percentage_decline                      
	  
from precrisis_restaurant as preres
join crisis_restaurant as cres
on preres.restaurant_id=cres.restaurant_id
order by percentage_decline desc
limit 10
```

## 4.Cancellation Analysis: What is the cancellation rate trend pre-crisis vs crisis, and which cities are most affected? 

```sql

create table canceldata
as
select 
r.city,
p.period,
r.order_id,
r.is_cancelled
from pre_crisis as p
join res_order as r
on p.order_id=r.order_id

create table  cancel_rate
as
select 
        city,
		period,
		count(order_id) as total_orders,
        count(*) filter(where is_cancelled='Y') as cancelled_orders,
		round((count(*) filter(where is_cancelled='Y')::numeric/count(order_id)::numeric)*100,2) 
		as cancellation_rate
from canceldata
group by city,period
--selfjoin
select 
     pre.city,
	 pre.cancellation_rate as pre_crisis_rate,
	 crisis.cancellation_rate as crisis_rate,
	 round(crisis.cancellation_rate-pre.cancellation_rate,2) as rate_change
from
cancel_rate as pre
join cancel_rate as crisis
on pre.city=crisis.city
where pre.period='pre_crisis'
and crisis.period='crisis'
order by rate_change desc
```


## 5. Revenue Impact: Estimate revenue loss from pre-crisis vs crisis (based on subtotal, discount, and delivery fee).

```sql
create table revenue
as
select
   o.order_id,
   o.customer_id,
   o.subtotal_amount,
   o.discount_amount,
   o.delivery_fee,
   round(sum(o.subtotal_amount-o.discount_amount+o.delivery_fee)::numeric,2) as total_revenue,
   pre.period
   from orders as o
   join pre_crisis as pre 
   on o.order_id=pre.order_id
group by o.order_id,pre.period 

select 
      period,
	  round(sum(total_revenue),2) as total_revenue
from revenue
group by period
order by period

```

# Summary

## 1.Monthly Orders — Pre-crisis vs Crisis

Insight:
Total orders dropped sharply after June 2025, following the food safety incident and delivery outage.

Pre-crisis total (Jan–May 2025): 96,321 orders

Crisis total (Jun–Sep 2025): 13,168 orders

Overall decline: ≈86%

While some improvement may have occurred toward the end of the crisis period, overall recovery remained limited

## 2. Top 5 City Groups — Highest % Decline in Orders

Insight:
All major city groups experienced a severe decline in total orders during the crisis period (June–September 2025) compared to the pre-crisis months (January–May 2025). The impact was widespread, with declines averaging around 86% across top markets.
<img width="698" height="298" alt="Screenshot_20251111_162125" src="https://github.com/user-attachments/assets/a3ecc743-d5e7-4b99-bc7a-a6e2e51a6ba1" />

## 3.Top 10 High-Volume Restaurants — Largest % Decline in Orders

Insight:
Among restaurants with at least 50 pre-crisis orders, the following top 10 high-volume partners experienced the steepest decline in order counts during the crisis period (June–September 2025).
<img width="967" height="458" alt="Screenshot_20251111_162606" src="https://github.com
  /user-attachments/assets/37a62001-926f-4a10-8f72-d11f4e639d43" />

## 4.Cancellation Analysis — Pre-crisis vs Crisis

Insight:
Cancellation rates increased significantly during the crisis period (June–September 2025) compared to the pre-crisis months (January–May 2025).

Most affected cities: Bengaluru, Chennai, and Kolkata

<img width="625" height="386" alt="Screenshot_20251111_163621" src="https://github.com/user-attachments/assets/4ef0d953-a493-4b51-bd47-b5e6042d4bc1" />

## 5.Revenue Impact — Pre-crisis vs Crisis

Insight:
Total revenue fell sharply during the crisis period (June–September 2025) compared to the pre-crisis months (January–May 2025).

Pre-crisis total revenue: ₹32,469,185.74

Crisis total revenue: ₹4,335,477.00

Overall revenue decline: ≈86.6%

<img width="428" height="242" alt="Screenshot_20251111_163815" src="https://github.com/user-attachments/assets/bd614697-572b-42ce-8fb5-3cbc02963c25" />

# Conclusion

The company now needs to focus on rebuilding trust, improving food safety checks, and restoring reliable delivery operations to recover customer confidence and sales.
During the crisis, QuickBite’s revenue dropped by nearly 87%.
The financial impact was severe for the company.
Recovery efforts showed  limited improvement.
QuickBite needs to rebuild customer trust and improve delivery to start earning well again.
