# Solving the SQL Murder Mystery https://mystery.knightlab.com/

Firstly I needed to isolate the crime report for the murder by filtering the date and the report type:
```
SELECT 
  * 
FROM 
  crime_scene_report 
WHERE 
  city = 'SQL City' 
  AND date = '20180115' 
  AND type = 'murder' 
LIMIT 
  10
```

This report revealed 'Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". 
I needed to use this to identify the witnesses in order to access their interviews:

```
SELECT 
  * 
FROM 
  person 
WHERE 
  address_street_name = 'Northwestern Dr' 
ORDER BY 
  address_number DESC 
LIMIT 
  1;
```
```
SELECT 
  * 
FROM 
  person 
WHERE 
  name LIKE 'Annabel%' 
  AND address_street_name = 'Franklin Ave';
```

This revealed the two witnesses: 'First Witness - Morty Schapiro, ID 14887, License ID 118009, 4919 Northwestern Dr SSN 111564949, Second Witness - Annabel Miller, ID 16371, License ID 490173, 103 Franklin Ave 318771143'
Now I needed to look at these witnesses' interviews in order to gain clues about who the murderer was. I used their person IDs to look up their interviews which revealed the following information: 'I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". 
I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.'
This allowed me to join the relevant tables and filter down to the suspect as follows:


```
SELECT 
  * 
FROM 
  get_fit_now_check_in c 
  INNER JOIN get_fit_now_member m ON c.membership_id = m.id 
  INNER JOIN person p ON m.person_id = p.id 
  INNER JOIN drivers_license dl ON p.license_id = dl.id 
WHERE 
  c.membership_id LIKE '48Z%' 
  AND dl.plate_number LIKE '%H42W%' 
  AND check_in_date = 20180109
```

I was able to identify the murderer as Jeremy Bowers and find his ID number. Using this I looked up the killer's interview:

```
SELECT * 
FROM interview
WHERE person_id = 67318
```
'I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.'
Using this information to filter and join the relevant tables I was able to find the mastermind behind the murders:

```
SELECT 
  * 
FROM 
  drivers_license dl 
  INNER JOIN person p ON dl.id = p.license_id 
  INNER JOIN facebook_event_checkin f ON f.person_id = p.id 
WHERE 
  car_make = 'Tesla' 
  AND car_model = 'Model S' 
  AND gender = 'female' 
  AND hair_color = 'red' 
  AND date >= 20170101 
  AND date < 20180101 
GROUP BY 
  f.person_id 
HAVING 
  COUNT(*) = 3 
LIMIT 
  10
```

This revealed the mastermind as: 'Miranda Priestly, 1883 Golden Ave, License ID 202298,  ID 99716.'
