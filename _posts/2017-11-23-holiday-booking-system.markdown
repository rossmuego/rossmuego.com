---
layout: post
title: "Holiday Booking System"
date: "2017-11-23 18:16:00 +0000"
tags: C#
      SQL
published: false

---
The largest project that I have undertaken yet, a booking system for a holiday resort. There were a number of different features requested be built into this system along with a number of basic tasks it must perform. This included:

* Adding/Removing/Altering Customers
* Adding/Removing/Altering Guests
* Adding/Removing/Altering Bookings
* Checking what chalets are available to book
* Creating an invoice for a booking

I incorporated a database into my solution in order to store bookings, customers and guests as well as keeping track of car hire and chalet availability. The database was stored locally in the solution for ease of demonstration/marking (as well as cost), however it could easily be connected to an online database.

This entire solution was programmed in C# with the database queries being carried out using SQL. I learnt a large number of things about the C# language during this process as well as how to implement an SQL database into my program and how to handle the data provided by it.

I also incorporated 2 popular design patterns into my solution; Facade
and Singleton.
# Database Structure

In total there are 6 tables being used in the solution. Below is an ER diagram detailing the relationship between them along with their columns.

![Database]({{ "/images/posts/holiday-system/"}})


# Adding Bookings

The fundamental part of the booking system is its ability to add bookings. Each booking must be assigned to a single customer, with a customer having potentially many bookings. A booking also has a number of guests assigned to it with 6 being the maximum. It also provides options for breakfast (+£5 per day), evening meals (+£10 per night) and optional car higher (+£50 per day selected).
