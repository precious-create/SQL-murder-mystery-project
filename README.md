# SQL-murder-mystery-project
A SQL project to solve a murder crime and find the culprit through investigation
# -- Details
-You vaguely remember the crime was a murder that occured sometime january 15th 2018 and that it took place in SQL City
# Solution Process
# step 1
-- my first step is to retrieve the crime scene reports from the 15th of january 2018 in SQL city, so that i can have the description of the report to help investigation.
select * from crime_scene_report

select * from crime_scene_report
where date = 20180115 and city = 'SQL City'

# step 2
-- After retrieving exact report from the 15th that has the 'murder' type, i take notes of the descriptions given in the report. 
-- the description says there was a security footage shows that there were 2 witnesses.
--the first witness lives at the last house on 'Northwestern Dr'. 
--the second witness, named Annabel lives somewhere on 'Franklin Avenue'
-- my next step is to find the full details of the 2 witnesses 

/* witness 1 - he/she lives at the last house on Northwestern Dr */

select *  from person

select * from person
where address_street_name  = 'Northwestern Dr'
order by address_number desc

# step 3
-- using the query above, i am able to find the first, based on the partial details we were given on his drive and his house being the last on the street.
--witness 1 is Mr Morty Schapiro, with ID 14887 and has a car with license ID 118009, he lives on 4919 Northwestern drive and has SSN 111564949.

select address_street_name, max(address_number)
from person
group by address_street_name
having address_street_name = 'Northwestern Dr'

select * from person
where address_number = 4919

/* witness 2 */

select * from person 
where name like '%Annabel%'
and address_street_name = 'Franklin Ave'

# step 4
--using the query above, i am able to find the second witness, using the partial details we were given on name like annabel and street name Franklin Ave
--the second witness is Annabel Miller with ID 16371, with license ID 490173 who lives on Franklin avenue with SSN 318771143 */

# step 5
-- my next step is to get their interview that was given to the police, this would help me understand what they saw at the crime scene. 
-- to do this i use their id number to filter the interview table. 

select * from interview
where person_id = 14887

-- Witness 1 Morty shapiro said in his interview - I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z".
--Only gold members have those bags. The man got into a car with a plate that included "H42W". 

select * from interview
where person_id = 16371

-- witness 2 Annabel said in her interview - I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. */

# step 6
-- my next step is to use the partial clues from these interviews to filter out the suspects.
--Morty's clue gives us a man with a Get fit now gym bag and membership number that started with '48z' and a car plate number like 'H42W'
-- Annabel's cluegives us a date the suspect would have been at the gym on the 9th of january


select * from get_fit_now_member
where ID like '48Z%'
and membership_status = 'gold'

-- the query above gives us 2 suspects
--suspect 1 - Joe Germuska has id 48Z7A and person id 28819, membership start date 20160305 and has gold status
--suspect 2- Jeremy Bowers has id 48Z55 and person id 67318, membership start date 20160101 and has gold status 
-- my next step is to then see which of the suspects was at the gym on the 9th of january 

select * from get_fit_now_check_in
where check_in_date = 20180109
and membership_id in ('48Z7A', '48Z55')

# step 7
-- this query above shows that both suspects were at the gym on the 9th of february 
-- since this does not give a clear answer, my next step is to use the partial car plate number that witness Morty gave in his interview, this would show which of the 2 suspects owns the car. 
-- to do this i will join the drivers license table which has details such as plate number, model and id which is the primary key on the persons table which has information on the name and license id.

select dl.age, dl.height, dl.gender, dl.plate_number, dl.car_make, dl.car_model, p.name, p.ssn, p.address_street_name, p.id 
from drivers_license AS dl
left join person AS p
on dl.id = p.license_id
where plate_number like '%H42W%' 

-- the query above gives me 3 returns, one of which has the name of one of the suspects from our ongoing investigation. 
-- The Murderer is suspect 2- Jeremy Bowers, the partial car plate number matches his and removes the other suspect from the list, leaving just Jeremy Bowers.

select * from person
where name = 'Jeremy Bowers'

--The Killers id is 67318 

select * from interview
where person_id = 67318

# step 8
-- my next step is to find out who hired the killer. The killer said-  I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") 
--or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. 

select * from drivers_license
where height between 65 and 67
and hair_color = 'red'
and gender = 'female'
and car_make = 'Tesla'
and car_model = 'Model S'

create table suspect as 
select * from drivers_license
where height between 65 and 67
and hair_color = 'red'
and gender = 'female'
and car_make = 'Tesla'
and car_model = 'Model S'

select * from suspect 

# step 9
-- the query above gives us the suspects ids. suspect id's 202298 , 291182,  918773 

select * from facebook_event_checkin
where event_name = 'SQL Symphony Concert'
and date between 20171201 and 20171231

select * from person

select s.id, s.age, s.height,
	   p.id as person_id, p.address_street_name, p.ssn, p.name
	   from suspect as s
	   right join person as p 
	   on s.id = p.license_id
	   
-- the query above gives us the person ids of the suspect. person id of the three suspect are 99716, 90700, 78881

select * from facebook_event_checkin
where event_name = 'SQL Symphony Concert'
and date between 20171201 and 20171231
and person_id in (99716, 90700, 78881)

-- the culprit has person id 99716

select * from person
where id = 99716

# final answer
-- The person who hired the killer is Miranda Priestly 
