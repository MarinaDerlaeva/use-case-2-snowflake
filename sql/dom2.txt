-----DOM 2----
 -- USE DATABASE ACCOUNTADMIN_ZOO_DB
 -- USE SCHEMA PUBLIC;


CREATE DATABASE ACCOUNTADMIN_ZOO_DB;
CREATE TABLE ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data (
    data VARIANT
);



 

SELECT * FROM zoo1_data LIMIT 1;

--  1. Профил на зоопарка: име и локация
-- sql

SELECT 
    data:zooName::STRING AS zoo_name,
    data:location::STRING AS location
FROM zoo1_data;

-- 2. Име и вид на директора (вложен обект)
SELECT 
    data:director.name::STRING AS director_name,
    data:director.species::STRING AS director_species
FROM zoo1_data;

SELECT 
    creature.value:name::STRING AS name,
    creature.value:species::STRING AS species
FROM zoo1_data,
LATERAL FLATTEN(input => data:creatures) AS creature;

-- 4. Същества от планетата 'Xylar'
SELECT 
    creature.value:name::STRING AS name
FROM zoo1_data,
LATERAL FLATTEN(input => data:creatures) AS creature
WHERE creature.value:origin::STRING = 'Xylar';

-- 5. Хабитати с размер > 2000 кв.м.
SELECT
  h.value:id::STRING AS habitat_id,
  h.value:name::STRING AS habitat_name,
  h.value:environmentType::STRING AS environment_type,
  h.value:sizeSqMeters::NUMBER AS size_sqm
FROM zoo1_data,
  LATERAL FLATTEN(input => data:habitats) h
WHERE TRY_CAST(h.value:sizeSqMeters AS NUMBER) > 2000;

SELECT data FROM zoo1_data LIMIT 1;



SELECT
    c.value:name::string AS creature_name
FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
LATERAL FLATTEN(input => data:creatures) c
WHERE ARRAY_CONTAINS(c.value:specialAbilities, 'Camouflage');


SELECT
    c.value:name::string AS creature_name,
    c.value:healthStatus.status::string AS health_status
FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
LATERAL FLATTEN(input => data:creatures) c
WHERE c.value:healthStatus.status::string != 'Excellent';

--6---
SELECT
  habitat.value:id,
  habitat.value:name,
  habitat.value:environmentType,
  habitat.value:features
FROM zoo1_data,
LATERAL FLATTEN(input => data:habitats) habitat,
LATERAL FLATTEN(input => habitat.value:features) feature
WHERE feature.value = 'Echo Chambers';



SELECT
    c.value:habitatId::string AS habitat_id,
    COUNT(*) AS creature_count
FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
LATERAL FLATTEN(input => data:creatures) c
GROUP BY c.value:habitatId::string;



SELECT DISTINCT
    f.value::string AS feature
FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
LATERAL FLATTEN(input => data:habitats) h,
LATERAL FLATTEN(input => h.value:features) f;


SELECT
    e.value:name::string AS event_name,
    e.value:type::string AS event_type,
    TO_TIMESTAMP(e.value:scheduledTime::string) AS scheduled_time
FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
LATERAL FLATTEN(input => data:upcomingEvents) e;




WITH
  creatures AS (
    SELECT
      c.value:id::string AS creature_id,
      c.value:name::string AS creature_name,
      c.value:habitatId::string AS habitat_id
    FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
    LATERAL FLATTEN(input => data:creatures) c
  ),
  habitats AS (
    SELECT
      h.value:id::string AS habitat_id,
      h.value:environmentType::string AS environment_type
    FROM ACCOUNTADMIN_ZOO_DB.PUBLIC.zoo1_data,
    LATERAL FLATTEN(input => data:habitats) h
  )
SELECT
  cr.creature_name,
  hb.environment_type
FROM creatures cr
JOIN habitats hb ON cr.habitat_id = hb.habitat_id;
