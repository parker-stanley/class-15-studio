# class-15-studio

# setting up the tables

create table book (
book_id int auto_increment primary key,
author_id int,
title varchar(255),
isbn int,
available bool,
genre_id int 
);

create table author (
author_id int auto_increment primary key,
first_name varchar(255),
last_name varchar(255),
birthday date,
deathday date
);

create table patron (
patron_id int auto_increment primary key,
first_name varchar(255),
last_name varchar(255),
loan_id int
);

create table reference_books (
reference_id int auto_increment primary key,
edition int,
book_id int,
foreign key (book_id) references book(book_id)
	on update set null
    on delete set null
);

INSERT INTO reference_books(edition, book_id)
VALUE (5,32);

create table genre (
genre_id int primary key,
genres varchar(100)
);

create table loan (
loan_id int auto_increment primary key,
patron_id int,
date_out date,
date_in date,
book_id int,
foreign key (book_id) references book(book_id)
	on update set null
    on delete set null
);

# 18.5.2 warm up queries

select title, isbn from book
where genre_id = 3;

select title, isbn from book
inner join genre on genre.genre_id = book.genre_id
where genres = "mystery";

select title, isbn
from book, genre
where book.genre_id = genre.genre_id
AND genres = 'mystery';

SELECT title, isbn
FROM book
WHERE genre_id IN (SELECT genre_id FROM genre WHERE genres = "Mystery");

select title, first_name, last_name 
from book b
inner join author a on b.author_id = a.author_id
where deathday is null;

select title, first_name, last_name 
from book, author
where book.author_id = author.author_id 
AND deathday is null;

-- select title, first_name, last_name 
-- from book
-- where author_id IN (select author_id from author where deathday is null);

# 18.5.3 loan out a book

update book
set available = false
where book_id = 3;

update book
set available = false
where title = 'The Golden Compass';

insert into loan (patron_id, date_out, book_id)
values (9, CURDATE(), 3);

-- better version
insert into loan (patron_id, date_out, book_id)
values ((select patron_id from patron where first_name="frank" AND last_name="jelk"), CURDATE(), (select book_id from book where title = 'The Golden Compass'));

update patron 
set loan_id = 4
WHERE patron_id = 9;

-- this one will execute for ALL matches of patron_id
update patron 
set loan_id = (select loan_id from loan where loan.patron_id = patron.patron_id)
WHERE patron_id = (select patron_id from loan where patron.patron_id = loan.patron_id);

-- this is a good one randy wrote using variables
SELECT @pid := patron_id FROM patron WHERE first_name = "Frank" AND last_name = "Jelk";
UPDATE patron
SET loan_id = (SELECT loan_id FROM loan WHERE loan.patron_id = @pid)
WHERE patron_id = @pid;

# 18.5.4 check a book back in
 
update book
set available = true
where title = 'The Golden Compass'; 

SELECT @bid := book_id FROM book WHERE title = "the golden compass";
update book
set available = true
where book_id = @bid;

update loan
set date_in = CURDATE()
where loan_id = (select loan_id from patron where first_name = "Frank" AND last_name = "Jelk");

SELECT @pid := patron_id FROM patron WHERE first_name = "Frank" AND last_name = "Jelk";
UPDATE patron
SET loan_id = null
WHERE patron_id = @pid;

-- better version that targets the specific loan (in the event there were multiple users with the same name)
-- SELECT @pid := patron_id FROM patron WHERE first_name = "Frank" AND last_name = "Jelk"
-- SELECT @lid := loan_id FROM loan WHERE patron_id = @pid;
-- UPDATE patron
-- SET loan_id = null
-- WHERE loan_id = @lid; 

# 18.5.5 wrap up query

select first_name, last_name, genres
from (
	select first_name, last_name, book_id, date_in
    from patron 
    inner join loan 
    on loan.patron_id = patron.patron_id
    ) AS patron_loan
inner join (
	select genres, book_id
    from book 
    inner join genre 
    on book.genre_id = genre.genre_id
    ) AS book_genre
on book_genre.book_id = patron_loan.book_id
where patron_loan.date_in is null;

-- another version
SELECT first_name, last_name, genres
FROM patron
JOIN loan USING (patron_id)
JOIN book USING (book_id)
JOIN genre USING (genre_id)
WHERE date_in IS NULL;

# bonus missions

select genre_id, COUNT(*)
from book 
group by genre.id;

-- book solution from LaunchCode, this is meant to replace the code when a book gets "checked in"
-- essentially preventing a book with the genre_id of 25 from every being made available
update book
set available = CASE
	WHEN genre_id = 25 THEN available
    ELSE FALSE
	END
WHERE book_id = 10;

update book
set available = false
WHERE genre_id = 25;

update book 
set available = false
WHERE genre_id = (select genre_id from genre where genres = "Reference");
