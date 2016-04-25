---
layout: post
title: SQL Analysis with the Chinook DB
excerpt: "Answering some questions with SQL"
categories: code-posts
comments: true
share: true
---

# ChinookDB Analyses

Schema:

![ChinookDB](http://lh4.ggpht.com/_oKo6zFhdD98/SWFPtyfHJFI/AAAAAAAAAMc/GdrlzeBNsZM/s800/ChinookDatabaseSchema1.1.png)

# Q & A style

### Who's the composer of the track 'Cochise'?

{% highlight sql %}
SELECT Track.Composer FROM Track WHERE Track.Name = 'Cochise';
{% endhighlight%}

*Audioslave/Chris Cornell*

### What is the name of the genre for the track 'Cochise'?

{% highlight sql %}
SELECT Genre.Name FROM Track 
INNER JOIN Genre ON Track.GenreId = Genre.GenreId 
WHERE Track.Name = 'Cochise';
{% endhighlight%}

*Rock*


### What's the name of the genre with the 3rd most tracks? How many tracks are there total in that genre? 

{% highlight sql %}
SELECT Genre.Name, COUNT(Track.TrackId) FROM Track 
INNER JOIN Genre 
ON Track.GenreId = Genre.GenreId 
GROUP BY Genre.GenreId 
ORDER BY COUNT(Track.TrackId) DESC 
LIMIT 2,1;
{% endhighlight %}

*Metal 374*


### How many tracks does the Artist 'Metallica' have? 

{% highlight sql %}
SELECT art.name, COUNT(t.TrackId) FROM Artist art
INNER JOIN Album alb 
ON art.ArtistId = alb.ArtistId 
INNER JOIN Track t ON t.AlbumId = alb.AlbumId 
WHERE art.Name = 'Metallica';
{% endhighlight %}
*Metallica 112*


### What countries buy the most music?

{% highlight sql %}
SELECT BillingCountry, COUNT(BillingCountry) 
FROM Invoice 
GROUP BY BillingCountry 
ORDER BY Count(BillingCountry) DESC
LIMIT 0, 1;
{% endhighlight %}

*USA 91*

### What are the top 5 genres in the US? 

{% highlight sql %}
SELECT Genre.Name, COUNT(Track.TrackId) FROM Track
INNER JOIN Genre
ON Track.GenreId = Genre.GenreId
INNER JOIN InvoiceLine 
ON Track.TrackId = InvoiceLine.TrackId
INNER JOIN Invoice
ON InvoiceLine.InvoiceId = Invoice.InvoiceId
WHERE Invoice.BillingCountry = 'USA'
GROUP BY Genre.GenreId
ORDER BY COUNT(Track.TrackId) DESC 
LIMIT 0, 5;
{% endhighlight %}

*Rock 157, Latin 91, Metal 64, Alternative & Punk 50, Jazz 22*

{% highlight sql %}
{% endhighlight %}