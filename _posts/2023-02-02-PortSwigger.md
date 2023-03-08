---
title: Port Swigger Academy Notes
date: 2023-02-02 21:19
categories: [Port Swigger]
tags: [Educational]
---

# SQL Injection

## What is SQL Injection

SQL injection is a vulnerability which allows an attacker to interfere with the queries that an application makes to its database. An attacker can exploit this vulnerability to obtain, modify, or delete sensitive information, and in some cases it can be used to gain access to the back-end infrastructure.

## Common SQL Injection Attacks

### Retreiving Hidden Data

Consider the URL: 

`https://insecure-website(.)com/products?category=Gifts`

which causes the application to make the SQL query 

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

This query will return all the products in the Gift category that are released. Now consider that the application is not properly sanitizing user input, and an attacker sends this request to the server.

`https://insecure-website.com/products?category=Gifts'--`

Which changes the query to

`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`

The double dash sequence is a comment indicator in SQL, and means that the rest of the query will not get interpreted, and effectively removes the `AND released = 1`, showing the attacker both the unreleased and released products.

Furthermore, an attacker could send

`https://insecure-website.com/products?category=Gifts'+OR+1=1--` 

Which modifies the query to look like

`SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`

This query will return items where the category = Gifts, or 1=1, which is always true. Meaning the query will return ALL items.

### Subverting Application Logic

Consider now an application that allows users to login with their username/password combination using the following SQL query:

`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`

Here an attacker could login as any user (given they know the username) by submitting `administrator'--` into the username field, and leaving the password field blank. The query then looks like

`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`

Which would successfuly log the attacker in as "administrator", assuming that is a valid username.

### Retrieving Data from Other Database Tables

In cases where the results of an SQL query are returned within the application's response, an attacker can use the UNION operator to retrieve data from other tables within the databse. For example, if an application uses the following query containing the user input "Gifts"

`SELECT name, description FROM products WHERE category = 'Gifts'`

an attacker could submit the input

`' UNION SELECT username, password FROM users--`

which will cause the application to return all usernames and passwords, as well as descriptions of the gift products.   

### Examing the Database

After determining that an application is vulnerable to SQL injection, it is useful to know more information about what kind of database is running. This heavily depends on the database type. You can also determine what database tables exist on most databases using

`SELECT * FROM information_schema.tables`

### Blind SQL Injection

Alot of SQL vulnerabilities in the wild will be blind, meaning the server will not return the results of the SQL query or any associated errors. Depending on the nature of the vulnerability, you can discover these by changing the logic of the query to trigger a detectable difference in the applications response (causing it to error out), or conditionally triggering a time delay.

## Detecting SQL Injection

To test for SQL injection manually, every single field that accepts user input can be tested. This can involve:

- Submitting a single quote, and looking for errors. (the single quote might break the syntax of the SQL query)
- Submitting some SQL specific syntax that evaluates to the original value of the entry point, and to a different value, and looking for systematic differences in the resulting application responses.
- Submitting Boolean conditions such as `OR 1=1` or `OR 1=2` and looking for differences in the application's responses.
- Submitting payloads that are designed to trigger time delays, these are different depending on which database the application is using (enumerate, enumerate, enumerate)


