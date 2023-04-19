# Platzi_cert
SELECT * FROM pasajero
JOIN viaje ON (viaje.id_pasajero=pasajero.id);

SELECT * FROM pasajero
LEFT JOIN viaje ON (viaje.id_pasajero=pasajero.id)
WHERE viaje.id IS NULL;

INSERT INTO public.estacion(
	id, nombre, direccion)
	VALUES (1, 'Nombre', 'Dire')
	on conflict (id) do UPDATE SET nombre='Nombre', direccion='Dire';
INSERT INTO public.estacion(
	nombre, direccion)
	VALUES ( 'RET', 'RETDRI')
	RETURNING *;

SELECT nombre
	FROM PUBLIC.pasajero
	WHERE nombre LIKE 'o%';	
-- sin importar mayusculas ni minusculas
SELECT nombre
	FROM PUBLIC.pasajero
	WHERE nombre ILIKE 'o%';

SELECT *
	FROM PUBLIC.tren
	WHERE modelo IS NULL;	
-- Funciones avanzadas Coalesce
SELECT id , COALESCE (nombre, 'No Aplica') AS nombre, direccion_residencia, fecha_nacimiento
	FROM public.pasajero WHERE id=1;
-- Funciones avanzadas prevenir errores NUllIF
SELECT NULLIF (0,0);
-- Encontrar el dato mayor
SELECT GREATEST (0,1,2,4,1,7,2,4);
-- Encontrar el dato menor
SELECT LEAST (0,1,2,4,1,7,2,4);
-- Bloques anónimos
SELECT *,
CASE
WHEN fecha_nacimiento> '2022-12-03' THEN
'NIÑO'
ELSE
'MAYOR'
END AS tipo FROM public.pasajero ORDER BY tipo;	
-- view volátil
select * from rango_view;
-- view materializada
SELECT * FROM PUBLIC.viaje WHERE inicio > '2023-01-01';
REFRESH MATERIALIZED VIEW  despues_noche_mview;
SELECT * FROM despues_noche_mview;
DELETE FROM public.viaje WHERE id= 10;
-- pl/pgsql Bloques de Código
DO $$ 
  DECLARE
  	rec record ;
	contador integer:=0;
  BEGIN
  	FOR rec  in SELECT * FROM public.pasajero LOOP 
		contador:= contador + 1;
  		RAISE NOTICE 'Un pasajero se llama %', rec.nombre;
	END LOOP;	
	RAISE NOTICE 'Conteo es %', contador;
  END
$$;
-- crear la función
DROP FUNCTION plsec();
CREATE  OR REPLACE FUNCTION plsec()
  RETURNS integer
AS $$ 
DECLARE
  	rec record ;
	contador integer:=0;
BEGIN
  	FOR rec  in SELECT * FROM public.pasajero LOOP 
		contador:= contador + 1;
  		RAISE NOTICE 'Un pasajero se llama %', rec.nombre;
	END LOOP;	
	RAISE NOTICE 'Conteo es %', contador;
	RETURN contador;
END
$$
LANGUAGE PLPGSQL;
-- crear la función con pgadmin
SELECT pl_sec();
-- FUNCTION: public.pl_sec()

-- DROP FUNCTION public.pl_sec();

CREATE FUNCTION public.pl_sec()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
DECLARE
  	rec record ;
	contador integer:=0;
BEGIN
  	FOR rec  in SELECT * FROM public.pasajero LOOP 
		contador:= contador + 1;
  	
	END LOOP;	
	INSERT INTO public.cont_pasajero(total,tiempo)
	VALUES (contador,now());
	RETURN NEW;
END
$BODY$;

ALTER FUNCTION public.pl_sec()
    OWNER TO postgres;

-- Crear trigger

CREATE TRIGGER mytrigger
AFTER INSERT
ON public.pasajero
FOR EACH ROW
EXECUTE PROCEDURE pl_sec();
--conexion con base datos remotos
CREATE EXTENSION dblink;
SELECT * FROM public.pasajero
JOIN
dblink ('dbname=remota
	  port= 5432
	  host=127.0.0.1 
	  user= usuario_consulta
	  password = 123',
	  'SELECT id, fecha FROM vip')
	  AS datos_remotos(id integer, fecha date)
 
-- ON (pasajero.id=datos_remotos.id);
USING (id);

--commit y roolback
BEGIN;

	
INSERT INTO public.tren(
	 modelo, capacidad)
	VALUES ('Modelo Trns' , 123);	
INSERT INTO public.estacion(
	 nombre, direccion)
	VALUES ('Estacion Transac', 'Dirsasdad');

COMMIT; 
--ROLLBACK
-- funciones adicionales de extensiones fuzzystrmatch
CREATE EXTENSION fuzzystrmatch;
SELECT levenshtein('oswaldo','osvaldo');

SELECT difference('oswaldo','osvaldo');
-- similitud en la pronunciación inglesa
SELECT difference('beard','bird');
