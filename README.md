Set up data within Elasticsearch
Often times, the dataset is not optimal for running requests in its original state.

For example, the type of a field may not be recognized by Elasticsearch or the dataset may contain a value that was accidentally included in the wrong field and etc.

These are exact problems that I ran into while working with this dataset. The following are the requests that I sent to yield the results shared during the workshop.

Copy and paste these requests into the Kibana console(Dev Tools) and run these requests in the order shown below.

STEP 1: Create a new index(ecommerce_data) with the following mapping.

PUT ecommerce_data
{
  "mappings": {
    "properties": {
      "Country": {
        "type": "keyword"
      },
      "CustomerID": {
        "type": "long"
      },
      "Description": {
        "type": "text"
      },
      "InvoiceDate": {
        "type": "date",
        "format": "M/d/yyyy H:m"
      },
      "InvoiceNo": {
        "type": "keyword"
      },
      "Quantity": {
        "type": "long"
      },
      "StockCode": {
        "type": "keyword"
      },
      "UnitPrice": {
        "type": "double"
      }
    }
  }
}
STEP 2: Reindex the data from the original index(source) to the one you just created(destination).

POST _reindex
{
  "source": {
    "index": "name of your original index when you added the data to Elasticsearch"
  },
  "dest": {
    "index": "ecommerce_data"
  }
}
STEP 3: Remove the negative values from the field "UnitPrice".

When you explore the minimum unit price in this dataset, you will see that the minimum unit price value is -11062.06. To keep our data simple, I used the delete_by_query API to remove all unit prices less than 0.

POST ecommerce_data/_delete_by_query
{
  "query": {
    "range": {
      "UnitPrice": {
        "lte": 0
      }
    }
  }
}
STEP 4: Remove values greater than 500 from the field "UnitPrice".

When you explore the maximum unit price in this dataset, you will see that the maximum unit price value is 38,970. When the data is manually examined, the majority of the unit prices are less than 500. The max value of 38,970 would skew the average. To simplify our demo, I used the delete_by_query API to remove all unit prices greater than 500.

POST ecommerce_data/_delete_by_query
{
  "query": {
    "range": {
      "UnitPrice": {
        "gte": 500
      }
    }
  }
}
Review from previous workshops
There are two main ways to search in Elasticsearch:

Queriesretrieve documents that match the specified criteria.
Aggregations present the summary of your data as metrics, statistics, and other analytics.
image

image

Get information about documents in an index
The following query will retrieve information about documents in the ecommerce_data index. This query is a great way to explore the structure and content of your document.

Syntax:

GET Enter_name_of_the_index_here/_search
Example:

GET ecommerce_data/_search
Expected response from Elasticsearch:

Elasticsearch displays a number of hits(line 12) and a sample of 10 search results by default(lines 16+).

The first hit(a document) is shown on lines 17-31. The field "source"(line 22) lists all the fields(the content) of the document.

image

Aggregations Request
Syntax:

GET Enter_name_of_the_index_here/_search
{
  "aggs": {
    "Name your aggregations here": {
      "Specify the aggregation type here": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Metric Aggregations
Metricaggregations are used to compute numeric values based on your dataset. It can be used to calculate the values of sum,min, max, avg, unique count(cardinality) and etc.

Compute the sum of all unit prices in the index
Syntax:

GET Enter_name_of_the_index_here/_search
{
  "aggs": {
    "Name your aggregations here": {
      "sum": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected response from Elasticsearch:

By default, Elasticsearch returns top 10 hits(Lines 16+).

image

When you minimize hits(red box- line 10), you will see the aggregations results we named sum_unit_price(image below). It displays the sum of all unit prices present in our index.

image

If the purpose of running an aggregation is solely to get the aggregations results, you can add a size parameter and set it to 0 as shown below. This parameter prevents Elasticsearch from fetching the top 10 hits so that the aggregations results are shown at the top of the response.

Using a size parameter

Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected response from Elasticsearch:

We no longer need to minimize the hits to get access to the aggregations results! We will be setting the size parameter to 0 in all aggregations requests from this point on.

image

Compute the lowest(min) unit price of an item
Syntax:

GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "min": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "lowest_unit_price": {
      "min": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected response from Elasticsearch:

The lowest unit price of an item is 1.01. image

Compute the highest(max) unit price of an item
Syntax:

GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "max": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "highest_unit_price": {
      "max": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected response from Elasticsearch:

The highest unit price of an item is 498.79.

image

Compute the average unit price of items
Syntax:

GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "avg": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "average_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected response from Elasticsearch:

The average unit price of an item is ~4.39.

image

Stats Aggregation: Compute the count, min, max, avg, sum in one go
Syntax:

GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "stats": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "all_stats_unit_price": {
      "stats": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected Response from Elasticsearch:

The stats aggregation will yield the values of count(the number of unit prices aggregation was performed on), min, max, avg, and sum(sum of all unit prices in the index).

image

Cardinality Aggregation
The cardinality aggregation computes the count of unique values for a given field.

Syntax:

GET Enter_name_of_the_index_here
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "cardinality": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "number_unique_customers": {
      "cardinality": {
        "field": "CustomerID"
      }
    }
  }
}
Expected response from Elasticsearch:

Approximately, there are 4325 unique number of customers in our index.
image

Limiting the scope of an aggregation
In the previous examples, aggregations were performed on all documents in the ecommerce_data index. What if you want to run an aggregation on a subset of the documents?

For example, our index contains e-commerce data from multiple countries. Let's say you want to calculate the average unit price of items sold in Germany.

To limit the scope of the aggregation, you can add a query clause to the aggregations request. The query clause defines the subset of documents that aggregations should be performed on.

The combined query and aggregations look like the following.

Syntax:

GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "query": {
    "Enter match or match_phrase here": {
      "Enter the name of the field": "Enter the value you are looking for"
    }
  },
  "aggregations": {
    "Name your aggregations here": {
      "Specify aggregations type here": {
        "field": "Name the field you want to aggregate here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "query": {
    "match": {
      "Country": "Germany"
    }
  },
  "aggs": {
    "germany_average_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}
Expected response from Elasticsearch:

The average of unit price of items sold in Germany is ~4.58. image

The combination of query and aggregations request allowed us to perform aggregations on a subset of documents. What if we wanted to perform aggregations on several subsets of documents?

This is where Bucket Aggregations come into play!

Bucket Aggregations
When you want to aggregate on several subsets of documents, bucket aggregations will come in handy. Bucket aggregations group documents into several sets of documents called buckets. All documents in a bucket share a common criteria.

image

The following are different types of bucket aggregations.

Date Histogram Aggregation
Histogram Aggregation
Range Aggregation
Terms aggregation
1. Date Histogram Aggregation
When you are looking to group data by time interval, the date_histogram aggregation will prove very useful!

Our ecommerce_data index contains transaction data that has been collected over time(from the year 2010 to 2011).

If we are looking to get insights about transactions over time, our first instinct should be to run the date_histogram aggregation.

There are two ways to define a time interval with date_histogram aggregation. These are Fixed_interval and Calendar_interval.

Fixed_interval With the fixed_interval, the interval is always constant.

Example: Create a bucket for every 8 hour interval.

Syntax:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "date_histogram": {
        "field":"Name the field you want to aggregate on here",
        "fixed_interval": "Specify the interval here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_8_hrs": {
      "date_histogram": {
        "field": "InvoiceDate",
        "fixed_interval": "8h"
      }
    }
  }
}
Expected response from Elasticsearch:

Elasticsearch creates a bucket for every 8 hours("key_as_string") and shows the number of documents("doc_count") grouped into each bucket.

image

Another way we can define the time interval is through the calendar_interval.

Calendar_interval With the calendar_interval, the interval may vary.

For example, we could choose a time interval of day, month or year. But daylight savings can change the length of specific days, months can have different number of days, and leap seconds can be tacked onto a particular year.

So the time interval of day, month, or leap seconds could vary!

A scenario where you might use the calendar_interval is when you want to calculate the monthly revenue.

Ex. Split data into monthly buckets.

Syntax:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "date_histogram": {
        "field":"Name the field you want to aggregate on here",
        "calendar_interval": "Specify the interval here"
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_month": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M"
      }
    }
  }
}
Expected response from Elasticsearch:

Elasticsearch creates monthly buckets. Within each bucket, the date(monthly interval) is included in the field "key_as_string". The field "key" shows the same date represented as a timestamp. The field "doc_count" shows the number of documents that fall within the time interval.

image

Bucket sorting for date histogram aggregation

By default, the date_histogram aggregation sorts buckets based on the "key" values in ascending order.

To reverse this order, you can add an order parameter to the aggregations as shown below. Then, specify that you want to sort buckets based on the "_key" values in descending(desc) order.

Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_month": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M",
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}
Expected response from Elasticsearch:

You will see that buckets are now sorted to return the most recent interval first.

image

Histogram Aggregation
With the date_histogram aggregation, we were able to create buckets based on time intervals.

The histogram aggregation creates buckets based on any numerical interval.

Ex. Create a buckets based on price interval that increases in increments of 10.

Syntax:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "histogram": {
        "field":"Name the field you want to aggregate on here",
        "interval": Specify the interval here
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_price_interval": {
      "histogram": {
        "field": "UnitPrice",
        "interval": 10
      }
    }
  }
}
Expected response from Elasticsearch:

Elasticsearch returns a buckets array where each bucket represents a price interval("key"). Each interval increases in increments of 10 in unit price. It also includes the number of documents placed in each bucket("doc_count").

image

Bucket sorting for histogram aggregation

By default, the histogram aggregation sorts buckets based on the _key values in ascending order. To reverse this order, you can add an order parameter to the aggregation. Then, specify that you want to sort buckets based on the _key values in descending(desc) order!

Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_price_interval": {
      "histogram": {
        "field": "UnitPrice",
        "interval": 10,
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}
Expected response from Elasticsearch:

You will see that the buckets are now sorted to return the price intervals in descending order.

imageg)

Range Aggregation
The range aggregation is similar to the histogram aggregation in that it can create buckets based on any numerical interval. The difference is that the range aggregation allows you to define intervals of varying sizes so you can customize it to your use case.

For example, what if you wanted to know the number of transactions for items from varying price ranges(between 0 and $50, between $50-$200, and between $200 and up)?

Syntax:

GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
   "Name your aggregations here": {
      "range": {
        "field": "Name the field you want to aggregate on here",
        "ranges": [
          {
            "to": x
          },
          {
            "from": x,
            "to": y
          },
          {
            "from": z
          }
        ]
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_custom_price_ranges": {
      "range": {
        "field": "UnitPrice",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 200
          },
          {
            "from": 200
          }
        ]
      }
    }
  }
}
Expected response from Elasticsearch:

Elasticsearch returns a buckets array where each bucket represents a customized price interval("key"). It also includes the number of documents("doc_count") placed in each bucket.

image

Bucket sorting for range aggregation

The range aggregation is sorted based on the input ranges you specify and it cannot be sorted any other way!

Terms Aggregation
The terms aggregation creates a new bucket for every unique term it encounters for the specified field. It is often used to find the most frequently found terms in a document.

For example, let's say you want to identify 5 customers with the highest number of transactions(documents).

Syntax:

GET Enter_name_of_the_index_here/_search
{
  "aggs": {
    "Name your aggregations here": {
      "terms": {
        "field": "Name the field you want to aggregate on here",
        "size": State how many top results you want returned here
      }
    }
  }
}
Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "top_5_customers": {
      "terms": {
        "field": "CustomerID",
        "size": 5
      }
    }
  }
}
Expected response from Elasticsearch:

Elasticsearch will return 5 customer IDs("key") with the highest number of transactions("doc_count").
image

Bucket sorting for terms aggregation

By default, the terms aggregation sorts buckets based on the "doc_count" values in descending order. To reverse this order, you can add an order parameter to the aggregation. Then, specify that you want to sort buckets based on the _count values in ascending(asc) order!

Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "5_customers_with_lowest_number_of_transactions": {
      "terms": {
        "field": "CustomerID",
        "size": 5,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}
Expected response from Elasticsearch:

You will see that the buckets are now sorted in ascending order of "doc_count", showing buckets with the lowest "doc_count" first.

image

Combined Aggregations
So far, we have ran metric aggregations or bucket aggregations to answer simple questions.

There will be times when we will ask more complex questions that require running combinations of these aggregations.

For example, let's say we wanted to know the sum of revenue per day.

To get the answer, we need to first split our data into daily buckets(date_histogram aggregation).

image

Within each bucket, we need to perform metric aggregations to calculate the daily revenue.

image

Calculate the daily revenue
The combined aggregations request looks like the following:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        }
      }
    }
  }
}
Expected Response from Elasticsearch:

Elasticsearch returns an array of daily buckets.

Within each bucket, it shows the number of documents("doc_count") within each bucket as well as the revenue generated from each day("daily_revenue").

image

Calculating multiple metrics per bucket
You can also calculate multiple metrics per bucket.

For example, let's say you wanted to calculate the daily revenue and the number of unique customers per day in one go. To do this, you can add multiple metric aggregations per bucket as shown below!

Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "number_of_unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}
Expected Response from Elasticsearch:

Elasticsearch returns an array of daily buckets.

Within each bucket, you will see that the "number_of_unique_customers_per_day" and the "daily_revenue" have been calculated for each day!

image

Sorting by metric value of a sub-aggregation

You do not always need to sort by time interval, numerical interval, or by doc_count! You can also sort by metric value of sub-aggregations.

Let's take a look at the request below. Within the sub-aggregation, metric values "daily_revenue" and "number_of_unique_customers_per_day" are calculated.

Let's say you wanted to find which day had the highest daily revenue to date!

All you need to do is to add the "order" parameter( and sort buckets based on the metric value of "daily_revenue" in descending("desc") order!

Example:

GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day",
        "order": {
          "daily_revenue": "desc"
        }
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "number_of_unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}
Expected response from Elasticsearch:

You will see that the response is no longer sorted by date. The buckets are now sorted to return the highest daily revenue first!

image

Q and A from the live workshop
