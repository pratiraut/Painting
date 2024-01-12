# Painting

Dive into the fascinating world of art with our SQL data analysis project, "Exploring Artistry through Data," featuring an extensive dataset comprising eight tables. Each table serves as a unique portal, offering insights into different facets of famous paintings and their illustrious creators. Through the lens of structured data analysis, this comprehensive collection, unraveling relationships and patterns that enrich our understanding of artistic evolution.This data-driven journey, where the synergy of SQL and art unveils the hidden stories within these tables, painting a vivid picture of the intricate connections between iconic masterpieces and the brilliant minds behind them.


### Dataset: [link](https://www.kaggle.com/datasets/mexwell/famous-paintings?resource=download&select=work.csv)



### function used:
1. Joins
2. Windows Function
3. Common Table Expression (CTE)
4. Delete 

## Problem Statement:
 1. Top 10 most famous painting subject.
 2. Top 5 most popular museum and artist.
 3. Which the country and the city with most no of museums.
 4. Which museum has the most no of most popular painting style.
 5. Which artist has the most no of Portraits paintings outside USA.
 6. Which country has the 5th highest no of paintings.


## Process:
- Importing a Data
  
    Eight CSV files were successfully imported into a PostgreSQL database utilizing Python. This strategic approach to data management resulted in a significant reduction in the importing time, highlighting the effectiveness of the optimization process.
  
- Understand the Data

  ```sql
  SELECT * FROM artist;
  SELECT * FROM canvas_size;
  SELECT * FROM image_link;
  SELECT * FROM museum_hours;
  SELECT * FROM museum;
  SELECT * FROM product_size;
  SELECT * FROM subject;
  SELECT * FROM works;
  ```
  

- Exploratory Data Analysis

  All the paintings which are not displayed on any museums?
  
  ```sql
  select * from works where museum_id is null;
  ```

  Are there museuems without any paintings?

  ```sql
   select *
    from museum m
    where not exists (select 1 from works w
                       where w.museum_id=m.museum_id)
  ```

  How many paintings have an asking price of more than their regular price?

  ```sql
   select * from product_size
	where sale_price > regular_price;
  ```

  Identify the paintings whose asking price is less than 50% of its regular price

  ```sql
   select * 
    from product_size
    where sale_price < (regular_price*0.5);
  ```
  Which canva size costs the most?

  ```sql
  select cs.labels as canva, ps.sale_price
	from (select *
		  , rank() over(order by sale_price desc) as rnk 
		  from product_size) ps
	join canvas_size cs on cs.size_id::text=ps.size_id
	where ps.rnk=1;
  ```
  
- Data Cleaning

  Removing duplicate records: Duplicates can skew analytical results. 

  ```sql
  delete from works 
    where ctid not in (select min(ctid)
                        from works
		        group by work_id );
  ```

  ```sql
  delete from product_size 
	where ctid not in (select min(ctid)
		            from product_size
                            group by work_id, size_id );
  ```

  ```sql
  delete from subject 
	where ctid not in (select min(ctid)
		            from subject
		            group by work_id, subject );
  ```

  ```sql
  delete from image_link 
	where ctid not in (select min(ctid)
		            from image_link
		            group by work_id );
  ```
  


  
  
- Data Retrieval

   Data Retrieval is the process of identifying and extracting data from a database, based on a query provided by the user or application.
  
   1. Top 10 most famous painting subject.
     
   ```sql
   select * 
	from (select s.subject,count(1) as no_of_paintings
		     ,rank() over(order by count(1) desc) as ranking
	       from works w
		join subject s on s.work_id=w.work_id
		group by s.subject ) x
	where ranking <= 10;
  ```

   ![Screenshot (261)](https://github.com/pratiraut/Painting/assets/146583441/066cb4fa-7596-463a-91b5-a0071253d0a9)

  2. How many museums are open every single day?
  ```sql
   select count(1)
	from (select museum_id, count(1)
		  from museum_hours
		  group by museum_id
		  having count(1) = 7) x;
  ```

  ![Screenshot (266)](https://github.com/pratiraut/Painting/assets/146583441/f42a1818-e2a6-4ab3-8026-638a0d43416a)


 
  3. Top 5 most popular museum? (Popularity is defined based on most no of paintings in a museum)
 
  ```sql
    select m.name as museum, m.city, m.country, x.no_of_painintgs
	from (	select m.museum_id, count(1) as no_of_painintgs
			, rank() over(order by count(1) desc) as rnk
			from works w
			join museum m on m.museum_id=w.museum_id
			group by m.museum_id) x
	join museum m on m.museum_id=x.museum_id
	where x.rnk<=5;
  ```

  ![Screenshot (262)](https://github.com/pratiraut/Painting/assets/146583441/9e98a75d-2eac-4a38-af32-0920dd89a677)


  4. Top 5 most popular artist? (Popularity is defined based on most no of paintings done by an artist)


  ```sql
   select a.full_name as artist, a.nationality, x.no_of_painintgs
	from (	select a.artist_id, count(1) as no_of_painintgs
			, rank() over(order by count(1) desc) as rnk
			from works w
			join artist a on a.artist_id=w.artist_id
			group by a.artist_id) x
	join artist a on a.artist_id=x.artist_id
	where x.rnk<=5;
  ```

  ![Screenshot (263)](https://github.com/pratiraut/Painting/assets/146583441/fe706233-792a-4e67-be37-0f32b82f21c2)


  5. Which museum has the most no of most popular painting style?

   
  ```sql
   with pop_style as 
		    (select style
			    ,rank() over(order by count(1) desc) as rnk
			from works
			group by style),
	     cte as
		    (select w.museum_id,m.name as museum_name,ps.style, count(1) as no_of_paintings
			    ,rank() over(order by count(1) desc) as rnk
			from works w
			join museum m on m.museum_id=w.museum_id
			join pop_style ps on ps.style = w.style
			where w.museum_id is not null
			and ps.rnk=1
			group by w.museum_id, m.name,ps.style)
	select museum_name,style,no_of_paintings
	from cte 
	where rnk=1;
  ```

  ![Screenshot (267)](https://github.com/pratiraut/Painting/assets/146583441/8b74a127-1abd-4a12-9316-a9002e7737e5)

  6. Which  country and  city with most no of museums.

 
   
  ```sql
    with cte_country as 
			(select country, count(1)
			, rank() over(order by count(1) desc) as rnk
			from museum
			group by country),
	    cte_city as
			(select city, count(1)
			, rank() over(order by count(1) desc) as rnk
			from museum
			group by city)
	select string_agg(distinct country.country,', '), string_agg(city.city,', ')
	from cte_country country
	cross join cte_city city
	where country.rnk = 1
	and city.rnk = 1;
  ```

  
  ![Screenshot (268)](https://github.com/pratiraut/Painting/assets/146583441/2418d834-8762-43f6-abf1-eb7bd2fc7043)

  7. Which country has the 5th highest no of paintings.


  ```sql
    with cte as 
		(select m.country, count(1) as no_of_Paintings
		, rank() over(order by count(1) desc) as rnk
		from works w
		join museum m on m.museum_id=w.museum_id
		group by m.country)
	select country, no_of_Paintings
	from cte 
	where rnk=5;
  ```

  ![Screenshot (269)](https://github.com/pratiraut/Painting/assets/146583441/84dacc00-90d4-4375-ab68-25bebae618ca)

  8. Which artist has the most no  Portraits paintings outside the USA.

  ```sql
    select full_name as artist_name, nationality, no_of_paintings
	from (
		select a.full_name, a.nationality
		,count(1) as no_of_paintings
		,rank() over(order by count(1) desc) as rnk
		from works w
		join artist a on a.artist_id=w.artist_id
		join subject s on s.work_id=w.work_id
		join museum m on m.museum_id=w.museum_id
		where s.subject='Portraits'
		and m.country != 'USA'
		group by a.full_name, a.nationality) x
	where rnk=1;
  ```

  ![Screenshot (270)](https://github.com/pratiraut/Painting/assets/146583441/e23d1ce8-1eeb-4784-beab-4140504687c8)




## Finding:

1. 58 paintings have asking prices that are less than 50% of their regular prices.
2. The canvas size with the highest cost is the "48" x 96"" (122 cm x 244 cm)" variant, priced at $1115.
3. Eighteen museums remain open every day without exception.
4. The Metropolitan Museum of Art boasts the highest number of paintings in the most popular style, with 244 artworks dedicated to Impressionism.
5. The country with the highest number of museums is the USA, and among the cities contributing to this count are London, Washington, New York, and Paris.
6. Jan Willem Pieneman and Vincent Van Gogh, both Dutch artists, have the highest number of portrait paintings, each with 14 artworks.






