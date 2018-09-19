# MYSQL

##Practice of using MYSQL

#Importing database
use sakila;

#1a. Display the first and last names of all actors from the table actor.
SELECT first_name, last_name from sakila.actor;

#1b. Display the first and last name of each actor in a single column in upper case letters. Name the column Actor Name.
SELECT concat(first_name,last_name) from actor;

#2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?
SELECT actor_id, first_name, last_name from actor
WHERE first_name = "JOE";

#2b. Find all actors whose last name contain the letters GEN
SELECT * from actor
WHERE last_name like "%GEN%";

#2c. Find all actors whose last names contain the letters `LI`. This time, order the rows by last name and first name, in that order:
SELECT * from actor
WHERE last_name like "%LI%"
ORDER BY last_name, first_name ASC;

#2d. Using `IN`, display the `country_id` and `country` columns of the following countries: Afghanistan, Bangladesh, and China:
SELECT country_id, country from country
WHERE country in ("Afghanistan","Bangladesh","China");

#3a. Add a middle_name column to the table actor. Position it between first_name and last_name. Hint: you will need to specify the data type.
ALTER TABLE actor
ADD middle_name char(10);
ALTER TABLE actor ADD last_name_new varchar(45);
ALTER TABLE actor ADD last_update_new varchar(45);

UPDATE actor SET last_name_new = last_name, last_update_new = last_update;

ALTER TABLE actor DROP COLUMN last_name;
ALTER TABLE actor DROP COLUMN last_update;
ALTER TABLE actor CHANGE COLUMN last_name_new last_name VARCHAR(45);
ALTER TABLE actor CHANGE COLUMN last_update_new last_update VARCHAR(45);

SELECT * FROM actor;

#3b. You realize that some of these actors have tremendously long last names. Change the data type of the middle_name column to blobs
ALTER TABLE actor MODIFY COLUMN middle_name BLOB;

#3c. Now delete the middle_name column.
ALTER TABLE actor DROP COLUMN middle_name;

#4a. List the last names of actors, as well as how many actors have that last name.
SELECT last_name,count(last_name) FROM actor GROUP BY last_name;

#4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors
SELECT last_name,count(last_name) FROM actor GROUP BY last_name HAVING count(last_name) >= 2;

#4c. Oh, no! The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.
UPDATE ACTOR 
SET first_name = "HARPO", last_name = "WILLIAMS" WHERE first_name = "GROUCHO" AND last_name = "WILLIAMS";

#5a. You cannot locate the schema of the address table. Which query would you use to re-create it?
SELECT * from address;

#6a. Use JOIN to display the first and last names, as well as the address, of each staff member. Use the tables staff and address:
SELECT first_name, last_name, address from staff INNER JOIN address ON address.address_id = staff.address_id;

#6b. Use JOIN to display the total amount rung up by each staff member in August of 2005. Use tables staff and payment.
SELECT staff.first_name, SUM(amount) from payment INNER JOIN staff ON staff.staff_id = payment.staff_id
GROUP BY staff.staff_id;

#6c. List each film and the number of actors who are listed for that film. Use tables film_actor and film. Use inner join.
SELECT film.title, COUNT(actor_id) actor_count from film INNER JOIN film_actor ON film.film_id = film_actor.film_id
GROUP BY film_actor.film_id;

#6d. How many copies of the film Hunchback Impossible exist in the inventory system?
SELECT film.title, COUNT(inventory_id) inventory_count from film INNER JOIN inventory ON film.film_id = inventory.film_id
WHERE film.title = "HUNCHBACK IMPOSSIBLE"
GROUP BY inventory.film_id;

#6e. Using the tables payment and customer and the JOIN command, list the total paid by each customer. List the customers alphabetically by last name:
SELECT customer.last_name , sum(payment.amount) from customer INNER JOIN payment ON customer.customer_id = payment.customer_id
GROUP BY customer.customer_id;

#7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. 
#	 As an unintended consequence, films starting with the letters K and Q have also soared in popularity. 
#	 Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.
SELECT * from film WHERE REGEXP_LIKE (title, "^[KQ]");

#7b. Use subqueries to display all actors who appear in the film Alone Trip.
SELECT first_name, last_name from actor WHERE actor_id in (SELECT actor_id from film_actor WHERE(film_id = (SELECT film_id from film WHERE title = "ALONE TRIP")));

#7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. 
#	 Use joins to retrieve this information.
SELECT * from customer INNER JOIN address on customer.address_id = address.address_id 
INNER JOIN city on address.city_id = city.city_id INNER JOIN country ON city.country_id = country.country_id WHERE country = "CANADA";

#7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as family films.
SELECT * from film INNER JOIN film_category on film.film_id = film_category.film_id INNER JOIN category on film_category.category_id = category.category_id WHERE name = "Family";

#7e. Display the most frequently rented movies in descending order.
SELECT COUNT(film.film_id) as rental_count , film.title from rental INNER JOIN inventory on rental.inventory_id = inventory.inventory_id
INNER JOIN film on film.film_id = inventory.film_id GROUP BY film.film_id ORDER BY rental_count DESC;

#7f. Write a query to display how much business, in dollars, each store brought in.
SELECT SUM(amount) as amounts_sold, address.district as store_district from staff INNER JOIN store on store.store_id = staff.store_id INNER JOIN address on address.address_id = store.address_id 
INNER JOIN payment on staff.staff_id = payment.staff_id GROUP BY payment.staff_id;

#7g. Write a query to display for each store its store ID, city, and country.
SELECT store_id, city, country from store INNER JOIN address on address.address_id = store.address_id INNER JOIN city on city.city_id = address.city_id
INNER JOIN country on country.country_id = city.country_id;

#7h. List the top five genres in gross revenue in descending order. 
#	 (Hint: you may need to use the following tables: category, film_category, inventory, payment, and rental.)
SELECT SUM(amount) as sales, category.name from category INNER JOIN film_category on film_category.category_id = category.category_id INNER JOIN inventory on inventory.film_id = film_category.film_id
INNER JOIN rental on rental.inventory_id = inventory.inventory_id INNER JOIN payment on payment.rental_id = rental.rental_id GROUP BY category.category_id; 

#8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. 
#	 Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.
CREATE VIEW top_five_genre as  SELECT SUM(amount) as sales, category.name from category INNER JOIN film_category on film_category.category_id = category.category_id INNER JOIN inventory on inventory.film_id = film_category.film_id
INNER JOIN rental on rental.inventory_id = inventory.inventory_id INNER JOIN payment on payment.rental_id = rental.rental_id GROUP BY category.category_id
ORDER BY sales DESC LIMIT 5; 

#8b. How would you display the view that you created in 8a?

SELECT * from top_five_genre;

#8c. You find that you no longer need the view top_five_genres. Write a query to delete it.

DROP VIEW top_five_genre;
