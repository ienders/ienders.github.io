---
layout: post
title:  "Detecting Unused Database Tables and Fields in Rails"
date:   2013-07-02 19:42:00
categories: coding
---

I was dealing with an overdesigned database schema today that I had very little memory of and wanted to do some housekeeping. In order to remove unused tables and fields, I decided to do a thorough scan.

For a table to be unused, it should have no records (ie: it stores no data). For a field in to be unused it should not hold any significant data. It can either all be filled with NULL values or every single value in every single record is the same. Homogeniety in all records on that field indicates that it could be extracted to your business logic and out of your datastore. (Of course, it is entirely possible that every record having the same value in a field is coincidence, so use your judgement.)

Here's the snippet of Rails code I wrote to do the scan:

{% highlight ruby %}
connection = ActiveRecord::Base.connection
connection.tables.collect do |t|
  count = connection.select_all("SELECT count(1) as count FROM #{t}", "Count").first['count']
  
  puts "TABLE UNUSED #{t}" if count.to_i == 0

  columns = connection.columns(t).collect(&:name).reject {|x| x == 'id' }
  columns.each do |column|
    values = connection.select_all("SELECT DISTINCT(#{column}) AS val FROM #{t} LIMIT 2", "Distinct Check")
    if values.size == 1
      if values.first['val'].nil?
        puts "COLUMN UNUSED #{t}:#{column}"
      else
        puts "COLUMN SINGLE VALUE #{t}:#{column} -- #{values.first['val']}"
      end
    end
  end
end
{% endhighlight %}

Feel free to extrapolate from this to get a database migration to clean up YOUR schema!