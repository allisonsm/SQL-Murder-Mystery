# SQL-Murder-Mystery
This is a breakdown of the queries I used to solve the [SQL Murder Mystery](https://mystery.knightlab.com/) game!

### SPOILER ALERT
Do not read any further if you want to figure out the solution on your own! I utilized comments on the queries page to keep track of my clues, so it is not spoiler free!

### Table of Contents

**Step 1: Retrieving crime scene report:**

    Using the preset query I discovered the database has the tables: 
        - crime_scene_report
        - drivers_license
        - facebook_event_checkin
        - interview
        - get_fit_now_member
        - get_fit_now_checkin
        - solution
        - income
        - person
First I looked at the crime scene table:


```
SELECT * 
FROM crime_scene_report 
LIMIT 20;
```

Then I looked for the report using the only information we know about the murder: that it took place on January 15th, 2018 in SQL City

```
SELECT *
FROM crime_scene_report
WHERE date = 20180115 OR date = 20181501;
```

After determining the date format, I narrowed it down:

```
SELECT *
FROM crime_scene_report
WHERE date = 20180115 AND city = "SQL City" AND type = "murder";
```

** Step 2: Tracking down witness statements**
     - Witness #1: lives at last house on "Northwestern Dr"
     - Witness #2: Annabel, lives somewhere on "Franklin Ave"

```
SELECT *
FROM person
LIMIT 10;

SELECT *
FROM interview
LIMIT 10;

SELECT *
FROM person
WHERE address_street_name = "Northwestern Dr"
ORDER BY address_number DESC;

SELECT *
FROM interview
WHERE person_id = 14887;
```

**Step 3: Search for Witness #1's partial sighting of Gym Bag # and License Plate #**
     - Gym bag, supposedly a Gold Member,  started with "48Z"
     - License plate included "H42W"

```
SELECT *
FROM get_fit_now_member
LIMIT 10;    

SELECT *
FROM get_fit_now_member
WHERE membership_status = "gold" AND id LIKE "48Z%";
```

--

```
SELECT *
FROM drivers_license
LIMIT 20; 

SELECT *
FROM drivers_license
WHERE plate_number LIKE "%H42W%";
```


**Step 4: Search for Witness #2, Annabel's statement**

```
SELECT *
FROM person
WHERE address_street_name = "Franklin Ave" AND name LIKE "Annabel%";
```

```
SELECT *
FROM interview
WHERE person_id = 16371;
```

She recognizes the killer from seeing them at the gym on January 9th

```
SELECT *
FROM get_fit_now_check_in
LIMIT 10;

SELECT *
FROM get_fit_now_check_in
WHERE check_in_date = 20180109;

SELECT *
FROM get_fit_now_member AS m
LEFT JOIN get_fit_now_check_in AS c
ON m.id = c.membership_id
WHERE m.membership_status = "gold" AND c.check_in_date = 20180109 AND id LIKE "48Z%";
```

Two people fit this citerea:
    - Joe Germuska, ID #48Z7A, Person_ID #28819
    - Jeremy Bowers, ID #48Z55, Person_ID #67318
    Either could have been seen by Annabel at the crime scene and the gym on Jan. 9th.
    They are both Gold Members at the gym with member ID's that match Witness #1's criteria.
    
**Step 5: Find out which potential suspect has a license plate matching Witness #1's criteria**

```
SELECT *
FROM person
WHERE id = 28819 OR id =67318;
```

Joe's license_id # is 173289, Jeremy's is 423327

```
SELECT *
FROM drivers_license
WHERE id = 173289 OR id = 423327
AND plate_number LIKE "%H42W%";
```

**Step 6: Jeremy appears to be our suspect. Let's see if he has a police interview on record:**

```
SELECT *
FROM interview
WHERE person_id = 67318;
```

Jeremy says he was hired by a woman who:
    - Is about 5'5" (65"), or 5'7" (67")
    - Has red hair
    - Drives a Tesla Model S
    - Attended SQL Symphony Concert three times in December 2017
    
**Step 6: Find the woman who hired Jeremy**

```
SELECT *
FROM drivers_license
WHERE hair_color = "red" 
AND gender = "female"
AND car_make = "Tesla"
AND car_model = "Model S"
AND height BETWEEN 65 AND 67;
```

Three license ID's match the description:
    - 202298
    - 291182
    - 918773 

```
SELECT *
FROM person
WHERE license_id = 202298
OR license_id = 291182
OR license_id = 918773;
```

Our three suspects for the hit are:
    - Red Korb, # 78881
    - Regina George, # 90700
    - Miranda Priestly, # 99716
Let's find who checked in a SQL Symphony multiple times in December 2017

```
SELECT person_id, event_name, COUNT(DISTINCT date) as attendances
FROM facebook_event_checkin
WHERE date BETWEEN 20171201 AND 20171231
AND event_name = "SQL Symphony Concert"
AND person_id = 78881 OR person_id = 90700 or person_id = 99716
GROUP BY person_id;
```

Miranda Priestly fits our killer's criteria, having checked in on Facebook at the SQL Symphony three times
