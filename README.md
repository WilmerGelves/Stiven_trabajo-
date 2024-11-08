# Consultas refuerzo Stiven (Base de datos Sakila) #

## Consultas básicas ##

1. Listar todos los clientes
    ```
    SELECT store_id,first_name,last_name,email,address_id,active,create_date,last_update 
    FROM customer
    ORDER BY first_name;
    ```
    

2. Obtener todas las películas con una duración menor a 90 minutos
    ```
     SELECT title
    FROM film
    WHERE film.length > 90;
    ```
4. Películas estrenadas en 2005
   ```
     SELECT title
    FROM film
    WHERE film.release_year = 2005;
    ```
   

6. Buscar clientes con el nombre "Michael"
    ```SELECT first_name,last_name
    FROM customer
    WHERE first_name = 'Michael';
    ```
7. Películas que no están en alquiler
    ```
    SELECT title 
    FROM film
    WHERE film_id NOT IN (
        SELECT film_id
        FROM rental
    );
    ```

## Consultas Multitabla (Con JOINs) ##
1. Obtener los clientes con el total de pagos realizados
    ```
    SELECT  c.first_name AS Nombre, c.last_name AS Apellido, SUM(p.amount) AS Total_pago
    FROM customer c
    JOIN payment p ON c.customer_id = p.customer_id
    GROUP BY c.customer_id;
    ```
2. Películas y su categoría correspondiente
    ```
    SELECT f.title AS titulo, c.name AS genero
    FROM film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id;
    ```
3. Clientes que han alquilado películas de la categoría 'Action'
    ```
    SELECT DISTINCT cl.first_name nombre, cl.last_name apellido
    from customer cl
    JOIN rental r ON cl.customer_id = r.customer_id
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film_category fc ON i.film_id= fc.film_id
    JOIN category c ON fc.category_id =  c.category_id
    WHERE c.name = 'Action' ORDER BY cl.first_name ASC, cl.last_name ASC;
    ```
4. ¿Cómo obtener una lista de los clientes junto con las películas que han alquilado?
    ```
    SELECT s.first_name, s.last_name, COUNT(r.rental_id) AS total_rentals
    FROM staff s
    JOIN rental r ON s.staff_id = r.staff_id
    GROUP BY s.staff_id;
    ```
5. ¿Cómo obtener la lista de categorías junto con el número de películas que pertenecen a cada una?
    ```
    SELECT c.name AS category_name, COUNT(f.film_id) AS num_films
    FROM category c
    JOIN film_category fc ON c.category_id = fc.category_id
    JOIN film f ON fc.film_id = f.film_id
    GROUP BY c.category_id;
    ```
## Subconsultas ##
1. ¿Cómo obtener los clientes que no han realizado pagos?
    ```
    SELECT first_name, last_name
    FROM customer
    WHERE customer_id NOT IN (
    SELECT customer_id
    FROM payment
    );
    ```
2. ¿Cómo obtener las películas que tienen una duración mayor a la media de todas las películas?
    ```
    SELECT title
    FROM film
    WHERE length > (
    SELECT AVG(length)
    FROM film
    );
    ```
3. ¿Cómo obtener los clientes que han alquilado una película específica?
    ```
    SELECT first_name, last_name
    FROM customer
    WHERE customer_id IN (
        SELECT customer_id
        FROM rental
        WHERE inventory_id IN (
            SELECT inventory_id
            FROM inventory
            WHERE film_id = 1
        )
    );
    ```
4. ¿Cómo obtener los empleados que han gestionado más de 50 alquileres?
    ```
    SELECT first_name, last_name
    FROM staff
    WHERE staff_id IN (
    SELECT staff_id
    FROM rental
    GROUP BY staff_id
    HAVING COUNT(rental_id) > 50
    );
    ```
5. ¿Cómo obtener las películas cuyo precio de alquiler es mayor que el precio promedio de alquiler de todas las películas?
    ```
    SELECT title
    FROM film
    WHERE rental_rate > (
    SELECT AVG(rental_rate)
    FROM film
    );
    ```

## Procesos de almacenado ##

1. Crear un procedimiento que reciba un film_id y devuelva el título, la descripción y la duración de la película.

```
DELIMITER $$

CREATE PROCEDURE GetFilmDetails(IN filmId INT)
BEGIN
    SELECT title, description, length
    FROM film
    WHERE film_id = filmId;
END $$

DELIMITER ;
```
2. Crear un procedimiento que agregue un nuevo cliente a la base de datos, usando varios parámetros de entrada.
```
DELIMITER $$

CREATE PROCEDURE AddCustomer(
    IN firstName VARCHAR(45),
    IN lastName VARCHAR(45),
    IN email VARCHAR(100),
    IN addressId INT
)
BEGIN
    INSERT INTO customer (first_name, last_name, email, address_id)
    VALUES (firstName, lastName, email, addressId);
END $$

DELIMITER ;
```
3. Procedimiento para obtener los detalles de una película
```
DELIMITER $$

CREATE PROCEDURE GetFilmDetails(IN filmId INT)
BEGIN
    SELECT title, description, length
    FROM film
    WHERE film_id = filmId;
END $$

DELIMITER ;
```
4. Procedimiento para actualizar el precio de alquiler de una película
```
DELIMITER $$

CREATE PROCEDURE UpdateRentalRate(IN filmId INT, IN newRate DECIMAL(4,2))
BEGIN
    UPDATE film
    SET rental_rate = newRate
    WHERE film_id = filmId;
END $$

DELIMITER ;
```
5. Procedimiento para obtener el total de pagos de un cliente

```
DELIMITER $$

CREATE PROCEDURE GetCustomerPayments(IN customerId INT)
BEGIN
    SELECT SUM(amount) AS total_payments
    FROM payment
    WHERE customer_id = customerId;
END $$

DELIMITER ;
```

