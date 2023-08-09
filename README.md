# Querys curso SQL
# USo de select
SELECT * from platzi.alumnos

SELECT nombre, apellido, colegiatura,
CASE
	WHEN colegiatura > 3000 THEN 'CARO'
	WHEN colegiatura > 3000 THEN 'CARO'
END AS Costo
FROM platzi.alumnos

# Uso de FROM, JOIN y WHERE
SELECT *
FROM platzi.alumnos AS ta
	JOIN platzi.carreras AS tc
	ON ta.carrera_id = tc.id
WHERE nombre LIKE '%as%'

# Top 10 carreras con mayor cantidad de alumnos
SELECT carrera, count(alumnos.id) as "Total de alumnos", vigente
FROM platzi.alumnos 
	JOIN platzi.carreras as carr
		ON platzi.alumnos.carrera_id = carr.id
WHERE vigente = true
GROUP BY carrera, vigente
ORDER BY "Total de alumnos" DESC
LIMIT 10

# Primer ejercicio
--Fetch es muy similar a limit, solo que fetch esta en la norma ANSI
--Se usa el ONLY para que solo te traiga un registro
SELECT *
FROM platzi.alumnos
FETCH FIRST 5 ROWS ONLY;--Aqui se modifica cuantos registros quieres
--Tambien se puede usar "LIMIT 1"

# Ejemplo con windows function
--ROW number dice "traeme cual es el numero de registro independientemente de el criterio que te ponga"
--Over vacio dice que de toda la tabla
--As es un alias
SELECT *
FROM (
	SELECT ROW_NUMBER() OVER() AS row_id, *
	FROM platzi.alumnos
)AS alumnos_with_row_num
WHERE row_id = 5;
--Ejercicio el primero
SELECT *
FROM (
	SELECT ROW_NUMBER() OVER() AS row_id, *
	FROM platzi.alumnos
)AS alumnos_with_row_num
WHERE row_id < 6;
# otra opcion --> WHERE row_id BETWEEN 1 and 5

# Segundo Ejercicio-> El segundo mas alto
# distintct solo trae una vez
# Dentro del subquery lo que hace es comparar dos copias de la misma columna
# Para cada valor, obtiene los valores que son mayores o igual a él
# Y finalmente obtiene la cuenta de ellos.
# colegiatura = [2000, 4800, 1000, 5000, 3000] 
# count = [4, 2, 5, 1, 3]
SELECT DISTINCT colegiatura 
FROM platzi.alumnos AS a1
WHERE 2 = (
	SELECT COUNT (DISTINCT colegiatura)
	FROM platzi.alumnos a2
	WHERE a1.colegiatura <= a2.colegiatura
);
# Esto se traduce a un ordenamiento con indexación y es por eso que elegir el 2 nos dará el segundo valor de colegiatura más alto. 
# Esta segunda forma es mas eficiente
SELECT DISTINCT colegiatura, tutor_id
FROM platzi.alumnos 
WHERE tutor_id = 20
ORDER BY colegiatura DESC
LIMIT 1 OFFSET 1;

# Tercer forma
# Se usa la query de arriba en una subquery solo que al final a esta se le hace un JOIN con los datos de los alunos que pagan esa colegiatura
SELECT *
FROM platzi.alumnos AS datos_alumnos
INNER JOIN (
	SELECT DISTINCT colegiatura
	FROM platzi.alumnos
	WHERE tutor_id = 20
	ORDER BY colegiatura DESC
	LIMIT 1 OFFSET 1
) AS segunda_mayor_colegiatura
ON datos_alumnos.colegiatura = segunda_mayor_colegiatura.colegiatura;

# En esta query se hace la subquery en el where
SELECT *
FROM platzi.alumnos AS datos_alumnos
WHERE colegiatura = (
	SELECT DISTINCT colegiatura
	FROM platzi.alumnos
	WHERE tutor_id = 20
	ORDER BY colegiatura DESC
	LIMIT 1 OFFSET 1
);

# Tarea: traer la segunda mitad de la tabla de alumnos, usar subquerys y limit
SELECT *
FROM platzi.alumnos
LIMIT 500 OFFSET 500;
# Otra solucion, aqui incluso no se usa el limit
SELECT * FROM platzi.alumnos OFFSET ( SELECT (COUNT(id)/2) FROM platzi.alumnos )
# Solucion del profe
SELECT ROW_NUMBER() OVER() AS row_id, *
FROM platzi.alumnos
OFFSET(
	SELECT COUNT(*)/2
	FROM platzi.alumnos
);

# Como hacer select en arrays o un grupo en especifico
# IN se usa para hacer selct en forma de arreglo 
SELECT *
FROM (
	SELECT ROW_NUMBER() OVER() AS row_id, *
	FROM platzi.alumnos
) AS alumnos_with_row_num
WHERE row_id IN (1,5,10,15,20);
--mas simplificado, con el subquery se tiene un resultado mas nutrido
--Y se hacen consultas mas especificas
SELECT *
FROM platzi.alumnos
WHERE id IN (
	SELECT id
	FROM platzi.alumnos
	WHERE tutor_id = 30
		AND carrera_id = 31
);
# Reto: Traer lista de alumnos que no se encuentren en las lista creadas
# Para el reto se puede usar "NOT" con cualquiera de los WHERE, "tutor_id <> 30"
SELECT *
FROM platzi.alumnos
WHERE id IN (
	SELECT id
	FROM platzi.alumnos
	WHERE NOT tutor_id = 30
);

# Ejercicios con tiempos y horas
# Con el extract se extrae el año de incorporacion
SELECT EXTRACT(YEAR FROM fecha_incorporacion) AS anio_incorporacion
FROM platzi.alumnos;
# Otra forma
SELECT DATE_PART('YEAR', fecha_incorporacion) AS anio_incorporacion
FROM platzi.alumnos;
# Extraccion del mes
SELECT DATE_PART('YEAR', fecha_incorporacion) AS anio_incorporacion,
		DATE_PART('MONTH', fecha_incorporacion) AS mes_incorporacion,
		DATE_PART('DAY', fecha_incorporacion) AS dia_incorporacion,
		DATE_PART('HOUR', fecha_incorporacion) AS hour_incorporacion,
		DATE_PART('MINUTE', fecha_incorporacion) AS minute_incorporacion,
		DATE_PART('SECOND', fecha_incorporacion) AS second_incorporacion
FROM platzi.alumnos;

# Usar date para hacer filtros en WHERE
SELECT *
FROM platzi.alumnos
WHERE (EXTRACT(YEAR FROM fecha_incorporacion)) =2019;
# TAmbien se puede usar DATE_PART aqui

# Con un subquery
SELECT *
FROM (
	SELECT *,
		DATE_PART('YEAR', fecha_incorporacion) AS anio_incorporacion
	FROM platzi.alumnos
) AS alumnos_con_anio
WHERE anio_incorporacion = 2020;

# Reto: Añadir mas campos, explorar alumnos incorporados en mayo de 2018
SELECT *
FROM (
	SELECT *,
		DATE_PART('YEAR', fecha_incorporacion) AS anio_incorporacion,
		DATE_PART('MONTH', fecha_incorporacion) AS month_incorporacion
	FROM platzi.alumnos
) AS alumnos_con_anio
WHERE anio_incorporacion = 2018 AND month_incorporacion = 5;

# Se añade un registro para ver duplicados
insert into platzi.alumnos (id, nombre, apellido, email, colegiatura, fecha_incorporacion, carrera_id, tutor_id) values (1001, 'Pamelina', null, 'pmylchreestrr@salon.com', 4800, '2020-04-26 10:18:51', 12, 16);

# Como encontrar registros duplicados en caso de que tengan el mismo ID
SELECT *
FROM platzi.alumnos AS ou
WHERE(
	SELECT COUNT(*)
	FROM platzi.alumnos AS inr
	WHERE ou.id = inr.id
) > 1;

# Primero se seleccionan todos los campos y se convierten a texto, los dos puntos son como hacer un cast
# Luego se agrupan todos en una sola columna
# Luego con having se pide el conteo de los registros duplicados segun el ID
SELECT (platzi.alumnos.*)::text,COUNT(*)
FROM platzi.alumnos
GROUP BY platzi.alumnos.*
HAVING COUNT(*) > 1

# Aqui se repetira el proceso solo que aqui se especificaran los campos
SELECT (
	platzi.alumnos.nombre,
	platzi.alumnos.apellido,
	platzi.alumnos.email,
	platzi.alumnos.colegiatura,
	platzi.alumnos.fecha_incorporacion,
	platzi.alumnos.carrera_id,
	platzi.alumnos.tutor_id
	)::text,COUNT(*)
FROM platzi.alumnos
GROUP BY platzi.alumnos.nombre,
	platzi.alumnos.apellido,
	platzi.alumnos.email,
	platzi.alumnos.colegiatura,
	platzi.alumnos.fecha_incorporacion,
	platzi.alumnos.carrera_id,
	platzi.alumnos.tutor_id
HAVING COUNT(*) > 1

# Otra forma con subquerys
# La particion es una seccion de datos que obedece ciertos criterios
# Con order by se pone en orden ascendente
# COn esta query se seleccionan los datos del registro duplicado y se muestran en pantalla
# Reto: Borrar datos repetidos con esta query
SELECT *
FROM (
	SELECT id,
	ROW_NUMBER() OVER(
		PARTITION BY
			nombre,
			apellido,
			email,
			colegiatura,
			fecha_incorporacion,
			carrera_id,
			tutor_id
		ORDER BY id ASC
	) AS row,
	--Aqui el asterisco hace referencia a todos los otros campos
	*
	FROM platzi.alumnos
) AS duplicados
WHERE duplicados.row > 1

# Reto: Borrar datos repetidos en la query de arriba
# Con IN se inidica que se borre lo que esta dentro
DELETE FROM platzi.alumnos
WHERE id IN (
	SELECT id
	FROM (
		SELECT id,
		ROW_NUMBER() OVER(
			PARTITION BY
				nombre,
				apellido,
				email,
				colegiatura,
				fecha_incorporacion,
				carrera_id,
				tutor_id
			ORDER BY id ASC
		) AS row
		FROM platzi.alumnos
	) AS duplicados
	WHERE duplicados.row > 1
);

# Selectores de rango
SELECT * 
FROM platzi.alumnos
WHERE tutor_id IN (1,2,3,4);

SELECT *
FROM platzi.alumnos
WHERE tutor_id >=1
	AND tutor_id<=10
# Tambien se puede usar el Between para definir el rango

# la funcion genera un rango de enteros de 10 a 20
SELECT int4range(10,20) @>3;

# Aqui se pregunta si hay valores que se solapen
SELECT numrange(11.1, 22.2) && numrange(20.0,30.0);

# Con upper se pregunta el valor mas alto de lo que se da en el rango
SELECT UPPER(int8range(15,25));

# Aqui se hace intereseccion entre ambos rangos, es decir, te dice el rango de valores que se tienen en comun
SELECT int4range(10,20) * int4range(15,25);

# Aqui se pregunta si el rango esta vacio 
SELECT ISEMPTY(numrange(1,5));

# Aqui se usa un rango para definir los ids que se quieren ver
SELECT * 
FROM platzi.alumnos
WHERE int4range(10,20) @> tutor_id;
# int4range: Que trae un rango de enteros.
# int8range: Es un rango de enteros grandes.
# numrange: Es un rango numérico.
# tsrange: Es un rango del tipo timestamp pero sin la zona horaria.
# tstzrange: Es un rango del tipo timestamp con la zona horaria
# daterange: Es un rango del tipo fecha.

# Reto: INtereseccion de valores entre ids de tutores y ids de carrera
SELECT tutor_id
FROM platzi.alumnos 
GROUP BY tutor_id

SELECT carrera_id
FROM platzi.alumnos 
GROUP BY carrera_id

# En esta solucion se obtienen los minimos y maximos de los ids de ambas columnas y luego se hace la intersección
SELECT INT4RANGE(MIN(tutor_id), MAX(tutor_id)) * INT4RANGE(MIN(carrera_id), MAX(carrera_id))
    FROM platzi.alumnos;

# Formas de sacar los maximos en una tabla
SELECT fecha_incorporacion
FROM platzi.alumnos
ORDER BY fecha_incorporacion DESC
LIMIT 1;

# Con esta query vemos las carreras y la fecha mas reciente en que se incorporo un alumno
SELECT carrera_id,MAX(fecha_incorporacion)
FROM platzi.alumnos
GROUP BY carrera_id
ORDER BY carrera_id;

# REto: Sacar el minimo nombre alfabeticamente que existe en la tabla, minimo de toda la tabla y por ID de tutor
SELECT MIN(nombre)
FROM platzi.alumnos;

# Segunda parte del reto, aqui obtienes los nombres ordenados alfabéticamente de los tutores agrupaados
SELECT tutor_id,MIN(nombre)
FROM platzi.alumnos
GROUP BY tutor_id
ORDER BY tutor_id;

# SELF JOIN_> hacer join con la propia tabla
# CONCAT sirve para concatenar columnas
SELECT CONCAT(a.nombre, ' ', a.apellido) AS alumno,
		CONCAT(t.nombre, ' ',t.apellido) AS tutor
FROM platzi.alumnos AS a
	INNER JOIN platzi.alumnos AS t ON a.tutor_id = t.id;

# Se hace otra query con group by y count, aqui se puede ver cuantos alumnos tiene cada tutor
# Aqui solo se usa un self join sin necesidad de usar otra tabla
SELECT CONCAT(t.nombre, ' ',t.apellido) AS tutor,
	COUNT(*) AS alumnos_por_tutor
FROM platzi.alumnos AS a
	INNER JOIN platzi.alumnos AS t ON a.tutor_id = t.id
	GROUP BY tutor
	ORDER BY alumnos_por_tutor DESC
	LIMIT 10;

# Reto: obtener promedio de alumnos por tutor
# Aquí se obtiene cuantos alumnos tiene cada tutor en promedio
SELECT AVG(alumnos_por_tutor) AS promedio_alumnos
FROM(
	SELECT CONCAT(t.nombre, ' ',t.apellido) AS tutor,
	COUNT(*) AS alumnos_por_tutor
	FROM platzi.alumnos AS a
	INNER JOIN platzi.alumnos AS t ON a.tutor_id = t.id
	GROUP BY tutor
	ORDER BY alumnos_por_tutor DESC
) AS promedio

# Experimento-- 
# promedio de tutores por carrera--
SELECT AVG(dc.tutor_por_carrera)
FROM(
# seleciona cuantas tutorias realiza en una carrera--
SELECT 	CONCAT(t.nombre,' ',	t.apellido) AS tutor,
		COUNT(*) AS tutor_por_carrera,
		a.carrera_id
FROM platzi.alumnos AS a
	INNER JOIN platzi.alumnos AS t ON a.tutor_id=t.id 
			AND a.carrera_id=t.tutor_id
GROUP BY tutor, a.carrera_id	
ORDER BY tutor_por_carrera DESC) AS dc

# Promedio de alumnos por tutor
SELECT AVG(alumnos_por_tutor) AS promedio_alumnos
FROM(
	SELECT CONCAT(t.nombre, ' ',t.apellido) AS tutor,
	COUNT(*) AS alumnos_por_tutor
	FROM platzi.alumnos AS a
	INNER JOIN platzi.alumnos AS t ON a.tutor_id = t.id
	GROUP BY tutor
	ORDER BY alumnos_por_tutor DESC
) AS promedio

# Resolviendo Diferencias
SELECT carrera_id, COUNT(*) AS cuenta
FROM platzi.alumnos
GROUP BY carrera_id
ORDER BY cuenta DESC;

# Aqui se hace una diferencia, se traen los alumnos de las carrera que no existen en la tabla de carreras
# Esto es un LEFT JOIN exclusive
# RETO: left join sin excluir gente
SELECT a.nombre,
	a.apellido,
	a.carrera_id,
	c.id,
	c.carrera
FROM platzi.alumnos AS a
	LEFT JOIN platzi.carreras AS c
	ON a.carrera_id = c.id
# Se puede comentar el where para no excluir a los nulos
WHERE c.id IS NULL
ORDER BY a.carrera_id;

# Todas las uniones
# Full outer join
SELECT a.nombre,
	a.apellido,
	a.carrera_id,
	c.id,
	c.carrera
FROM platzi.alumnos AS a
	FULL OUTER JOIN platzi.carreras AS c
	ON a.carrera_id = c.id
ORDER BY a.carrera_id;

# LEFT JOIN, RIGHT JOIN, INNER JOIN
SELECT a.nombre,
	a.apellido,
	a.carrera_id,
	c.id,
	c.carrera
FROM platzi.alumnos AS a
	INNER JOIN platzi.carreras AS c
	ON a.carrera_id = c.id
# Se usa el where para obtener una right exclusive join o un LEFT exclusive
# WHERE a.id IS NULL
ORDER BY c.id DESC;

# Exclusive full outer JOIN
SELECT a.nombre,
	a.apellido,
	a.carrera_id,
	c.id,
	c.carrera
FROM platzi.alumnos AS a
	FULL OUTER JOIN platzi.carreras AS c
	ON a.carrera_id = c.id
WHERE a.id IS NULL
	OR c.id IS NULL
ORDER BY a.carrera_id DESC, c.id DESC;

# Query trayendo una lista de los alumnos (customers) con su respectivo tutor_id (company) 
# de la carrera (location) que tuviera más alumnos (customers).
SELECT CONCAT(a.nombre, ' ', a.apellido),
		a.tutor_id
FROM platzi.alumnos AS a
	JOIN platzi.carreras AS c
	ON c.id = a.carrera_id
WHERE c.id = (
	SELECT c.id
	FROM	platzi.carreras AS c
		INNER JOIN platzi.alumnos AS a
		ON a.carrera_id = c.id
	GROUP BY c.id
	ORDER BY COUNT(*) DESC LIMIT 1);
	
# Triangulando
# Left padding, alcochonamiento a la izquierda
SELECT lpad('sql',15,'*');

SELECT lpad('*',id, '*'),carrera_id
FROM platzi.alumnos
WHERE id <10
ORDER BY carrera_id;

# LPAD ayuda a tener cadenas de la longitud que se necesite en caso de que un pipeline lo requiera
# Aqui se usar ROW id para dar forma a triángulo
# COn cast se converte el row a entero
# Reto:Usar Rpad
SELECT lpad('*',CAST(row_id AS int), '*') 
FROM(
	SELECT ROW_NUMBER() OVER(ORDER BY carrera_id) AS row_id, *
	FROM platzi.alumnos
) AS alumnos_with_row_id
WHERE row_id <= 5
ORDER BY carrera_id;

SELECT rpad('3', 3, '0');

# Generando Rangos, se debe tener cuidado con el overlapping,
# En el tercer parámetro se pone de cuanto en cuanto vaya avanzando
SELECT *
FROM generate_series(5,1,-2);

# Aquí se van sumando 7 días a la fecha actual, por defecto se suman días con el +
SELECT current_date + s.a AS dates
FROM generate_series(0,14,7) AS s(a)

# Se genera una tabla dinámica que va aumentando de 10 horas en 10 horas
SELECT *
FROM generate_series('2020-09-01 00:00:00'::"timestamp",
					'2020-09-04 12:00:00','10 hours');
					
# Hacer un inner join con un generate series
SELECT a.id,
	a.nombre,
	a.apellido,
	a.carrera_id,
	s.a
FROM platzi.alumnos AS a
	INNER JOIN generate_series(0,10) AS s(a)
	ON s.a = a.carrera_id
ORDER BY a.carrera_id;

# Reto: generar triángulo con rango generando
# Create a triangle with series
SELECT	lpad('(ツ)', s.a, '(ツ)')
FROM platzi.alumnos AS a
	INNER JOIN generate_series(3,30,3) AS s(a)
	ON s.a = a.id
ORDER BY s.a;
# Otra solución
SELECT lpad('\', id, '/') 
FROM generate_series(0,50) as id
# Otra
SELECT rpad('Te Amo <',s.a,'3')
FROM generate_series(1,15) AS s(a)
# Solucion del profe
SELECT lpad('*',CAST(ordinality AS int), '*')
FROM generate_series(10,2,-2) WITH ordinality;

# Regularizando Expresiones
SELECT email
FROM platzi.alumnos
WHERE email ~*'[A-Z0-9._%+-]+@google[A-Z0-9.-]+\.[A-Z]{2,4}';

# Bases de datos distribuidas: Bases de datos en diferentes partes pero conectadas en una red informática
# Ventajes-> DEsarollo modular: Puedes separar a conveniencia para tener mayor confiabilidad, mejor rendimiento
# y una mayor rapidez de respuesta
# DEsventajas: Es mas dificil el manejo de seguridad al tner distintos puntos, puede ser mas complejo
# la integración puede ser mas compleja y mas costosa
# Hay homógenes y heterógeneas: 
# * OS
# * Sistemas de base de datos
# * Modelo de datos

# Arquitecturas:
# * Cliente-Servidor: Bases de datos clientes o esclavos que piden a la bd prinicipal
# * Par a par o peer 2 peer: todos se hablan como iguales
# * Multi manejador de base de datos: 

# Estrategias de diseño: 
# * Top down: Se planea muy bien, se va configurando de arriba hacia abajo de acuerdo a las necesidades
# * Botton up: Ya se tiene algo existente, por lo cual se empieza a construir desde ahí

# Carácterísticas del almacenamiento distribuido
# * Fragmentaicón: Horizontal:sharding, ej. rows de méxico a méxico y rows de colombia a colombia;
# vertical:cuando manejas algo columnar; mixta:
# * Replicación: Completa, parcial o sin replcación
# * Distribución: Como se transfieren los datos de un lado a otro
# Puedes ser Centralizada, Particionada o Replicada

# **************************--------**********************
# SHARDING
# Se puede analizar la analogía de la pizza, no puedes acabarte una pizza solo
# COn sharding se parte la pizza y llamas amigos para comerse la pizza, se divide en shards
# Partes rows o tuplas y las partes en diversas peticiones
# Desventajas: Los joins entre chards se pueden volver muy complejos, Es dificil cambiar la estructura de un shard
# por lo cual se tiene baja elasticidad (se puede hacer subsharding pero puede ser muy complejo)
# En sharding se elimina el uso de las Pk ya que el rodenamiento esta definido por el criterio con el que se estructuro
# Se recomienda para datos estáticos que tienen mucho tráfico

# **************************--------**********************
# WINDOW FUNCTION
# Relación entre un row y un window frame (sección de la tabla que te interesa medir)
# Ayuda a evitar usar self joins
# En esta query se obtiene el promedio de costo de colegiatura por carrera
# Sin la particion se obtiene un único promedio de la colegiatura
SELECT *,
	AVG(colegiatura) OVER (PARTITION BY carrera_id)
FROM platzi.alumnos

# Order by dentro de la WF se toma agregación los campos que se encuentran igual o que esten arriba al actual
# Es decir, primero separa los registros con colegiatura de 2000 y los sumas y ahi entrega su suma
# Al momento que se cambia de colegiatura, ej, 2500, ahora sumara todas las que tienen 2500
# Al agregar la particion se va haciendo la suma conforme se va cambiando de colegiatura
SELECT *,
# PARTITION BY carrera_id 
	SUM(colegiatura) OVER (PARTITION BY carrera_id ORDER BY colegiatura)
FROM platzi.alumnos

# Con ranking se obtiene que lugar ocupa algo en nuestra tabla
# En este ejemplo se muestra en la columna rank cual es el numero de colegiatura mas alta
# Con el Order by se ordena primero por carrera y luego por el ranking
# Las window function corren al final de todo, es decir, si se tiene un where, leera el where sin tomar en cuenta la window function
# Se deben usar subquerys para poder usar el where
SELECT *
FROM(
	SELECT * ,
	RANK() OVER (PARTITION BY carrera_id ORDER BY colegiatura DESC) AS brand_rank
	FROM platzi.alumnos
) AS rank_colegiaturas_por_carrera	
WHERE brand_rank < 3
ORDER BY brand_rank;

# Window Functions: Particiones y agregaciones
# Con el row number se obtienen un indice alterno al id
SELECT ROW_NUMBER() OVER(ORDER BY fecha_incorporacion) AS row_id, *
FROM platzi.alumnos;

# Con first value se asigna el primer valor de colegiatura a todas las rows
# En este caso el valor va cambiando de acuerdo a carrera_id
SELECT FIRST_VALUE(colegiatura) OVER(PARTITION BY carrera_id) AS primera_colegiatura, *
FROM platzi.alumnos;

# Last value es similas a first value solo que aqui se asigna el último valor a todas las rows con el mismo colegiatura_id
# Si usas NTH_VALUE se puede especificar que numero quieres -> NTH_VALUE(colegiatura, 3)
SELECT LAST_VALUE(colegiatura) OVER(PARTITION BY carrera_id) AS ultima_colegiatura, *
FROM platzi.alumnos;

# Rango
# Con dense rank se busca eliminar los gaps, que son los errores que provoqcan que se impriman los rangos en desorden
# Se tiene un rango mas condensado o denso
SELECT *,
	DENSE_RANK() OVER (PARTITION BY carrera_id ORDER BY colegiatura DESC) AS colegiatura_rank
FROM platzi.alumnos
ORDER BY carrera_id, colegiatura_rank;

# Precent rank categoriza con porcentajes (rank-1) / (total rows -1)
SELECT *,
	PERCENT_RANK() OVER (PARTITION BY carrera_id ORDER BY colegiatura DESC) AS colegiatura_rank
FROM platzi.alumnos
ORDER BY carrera_id, colegiatura_rank;

# El futuro de SQL
# Cada vez esta mas y mas presente