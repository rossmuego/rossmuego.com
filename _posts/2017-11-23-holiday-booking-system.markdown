---
layout: post
title: "Holiday Booking System (Post WIP)"
date: "2017-11-23 18:16:00 +0000"
tags: C#
      SQL
published: true

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

If you wish to see all the code from this project, checkout the [solution][solution-link] on my [GitHub][github-link].

# Adding Bookings

The fundamental part of the booking system is its ability to add bookings. Each booking must be assigned to a single customer, with a customer having potentially many bookings. A booking also has a number of guests assigned to it with 6 being the maximum. It also provides options for breakfast (+£5 per day), evening meals (+£10 per night) and optional car higher (+£50 per day selected).

In order to create bookings, there needed to be a uniqe ID generated for that booking, in order to do that I created a Singleton class names RefGeneratorSingleton and used that to generate a unique booking ID.

```cs
public class RefGeneratorSingleton
{
    private RefGeneratorSingleton() { }

    private static RefGeneratorSingleton generator;

    public static RefGeneratorSingleton Generator
    {
        get
        {
            if(generator == null)
            {
                generator = new RefGeneratorSingleton();
            }
            return generator;
        }
    }

    private int bookingRef = baseBookRef();

    public int generateBookingRef()
    {
        bookingRef++;

        return bookingRef;
    }

    private static int baseBookRef()
    {
        int highbook = 0;

        SqlConnection conn = new SqlConnection(@"Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\NapierHolidaysDB.mdf;Integrated Security=True;Connect Timeout=30");
        conn.Open();

        SqlCommand command = new SqlCommand("SELECT booking_id FROM Bookings ORDER BY booking_id DESC", conn);
        highbook = Convert.ToInt32(command.ExecuteScalar());

        conn.Close();

        return highbook;
    }
}
```

# Conflict Checking

So another fundamental part of the task was making sure that bookings are not made for chalets that are already occupied during the selected days of a holiday. In order to do this I created 2 tables in my database. One held the number of chalets the holiday location had (in out example this was from 1-10), and then another tables named ChaletAvali which held what bookings were in what chalets. I then ran the below SQL statement which showed all the chalets that were free within the specified dates.

```sql
SELECT chalet_id
FROM Chalets WHERE
chalet_id NOT IN
( SELECT chalet_id
  FROM ChaletAvali
  WHERE ((arrival BETWEEN @arrival AND @depart)
  OR (depart BETWEEN @arrival AND @depart)
  OR (@arrival >= arrival AND @depart <= depart))
)
```

# Retrieving a Booking

In order to allow a user to retrieve a booking, a number of actions must be performed. Since the booking, customer and guest are all stored in different tables, we must perform a couple of statements and handle a few different objects.

I call the getBooking method in my Facade from my presentation layer, simply passing a booking ID as a parameter. This is what that method looks like in my Facade;

```cs
public ArrayList searchBooking(int bookingref)
{
    ArrayList booking = new ArrayList();

    string[] bookingArray = _database.getBooking(bookingref);
    Booking found = new Booking();
    found.ArrivalDate = Convert.ToDateTime(bookingArray[0]);
    found.DepartureDate = Convert.ToDateTime(bookingArray[1]);
    found.Chalet = Convert.ToInt32(bookingArray[2]);
    found.CustomerID = Convert.ToInt32(bookingArray[3]);
    found.Breakfast = Convert.ToBoolean(bookingArray[4]);
    found.Evening = Convert.ToBoolean(bookingArray[5]);
    found.Car = Convert.ToInt32(bookingArray[6]);
    found.TotalGuests = Convert.ToInt32(bookingArray[7]);
    found.BookingRef = Convert.ToInt32(bookingArray[8]);

    Customer foundCustomer = SearchCustomer(found.CustomerID);

    booking.Add(foundCustomer);

    booking.Add(found);

    List<Guest> guestList = new List<Guest>();

    ArrayList guest = _database.getGuest(bookingref);

    foreach(string[] i in guest)
    {
        Guest foundGuest = new Guest();
        foundGuest.GuestID = Convert.ToInt32(i[0]);
        foundGuest.Name = i[1];
        foundGuest.Age = Convert.ToInt32(i[2]);
        foundGuest.PassportNumber = i[3];
        guestList.Add(foundGuest);
    }

    booking.Add(guestList);

    Car carHire = new Car();

    ArrayList carHireList = _database.getCarHire(bookingref);

    foreach(string[] x in carHireList)
    {
        carHire.Name = x[0];
        carHire.Start = Convert.ToDateTime(x[1]);
        carHire.End = Convert.ToDateTime(x[2]);
        booking.Add(carHire);
    }

    return booking;
}
```
This code can be broken down into a number of components. The first thing it does is create an ArrayList named booking. This list will house all the components of our booking and return it to the presentation layer where it will later be handled.

The first database method is then called, that being the getBooking method. This simply runs an SQL statement selecting a row where the bookingID is equal to that provided. It then handles the SQL results and adds them all to a string array as shown below:

```cs
using (SqlDataReader reader = command.ExecuteReader())
{
    if (reader.Read())
    {
        foundbooking[0] = reader["arrivalDate"].ToString();
        foundbooking[1] = reader["departureDate"].ToString();
        foundbooking[2] = reader["chalet_id"].ToString();
        foundbooking[3] = reader["customer_id"].ToString();
        foundbooking[4] = reader["breakfast"].ToString();
        foundbooking[5] = reader["evening"].ToString();
        foundbooking[6] = reader["car_days"].ToString();
        foundbooking[7] = reader["total_guests"].ToString();
        foundbooking[8] = reader["booking_id"].ToString();
    }
}
```
This returned string array called foundbooking is then converted into a Booking object as seen above using the different positions in the array to set the various properties.

The same is done for customer, with the method 'SearchCustomer' also being in the Facade and creating the Customer object before returning it and adding it to the list we created at the start along with the booking object we just made.

For guests it is slightly different since there are potentially a number of guests for each booking. The method getGuest is invoked, which in the data layer adds a number of string arrays to an ArrayList and returns that. This way, we can loop through the ArratList using foreach, then for each string array within that list assign it to a guest object and add that guest to a List<Guest> which is then added to our booking ArrayList.



[solution-link]:  www.
[github-link]:  www.github.com/rossmuego
