# AirBnB MongoDB Analysis

[original data set](https://insideairbnb.com/get-the-data/)
My data set includes AirBnB listings data from Sevilla, Andalucía, Spain.
The original data set comes in csv format.
Below are some set up codes necessary to perform this analysis.

```
ssh yz6350@i6.cims.nyu.edu
module load mongodb-4.0
export LC_ALL=en_US.UTF-8
mongoimport --headerline --type=csv --db=yz6350 --collection=listings --host=class-mongodb.cims.nyu.edu --file=listings.csv --username=yz6350 --password=bV4xdU2L
mongosh yz6350 -host class-mongodb.cims.nyu.edu -u yz6350 -p
```

## Data analysis in MongoDB

### 1.
Display two listings from the dataset
```
db.listings.find().limit(2)
```

### 2.
Display ten listings from the dataset using a different approach
```
db.listings.find().limit(10).pretty()
```

### 3.
First I find all host_id that are superhost
```
db.listings.find({ host_is_superhost: "t" }, { host_id: 1, host_is_superhost: 1}).limit(5).pretty()
```
I picked 504269030 and 14958822, and then I display the fields as requested while referring to their individual host_id.
```
db.listings.find(
  { 
    host_id: { $in: [504269030, 14958822] }
  },
  {
    host_id:1,
    name: 1, 
    price: 1, 
    neighbourhood: 1,
    host_name: 1,
    host_is_superhost: 1
  }
).pretty()
```

### 4.
Using the distinct function, I can display all unique host_name easily.
```
db.listings.distinct("host_name")
```

### 5.
Using $gt: 2, I find all places in Seville, Andalusia, Spain that have more than 2 beds.
```
db.listings.find(
  {
    beds: { $gt: 2 },
    neighbourhood: "Seville, Andalusia, Spain"
  },
  {
    name: 1,
    beds: 1,
    review_scores_rating: 1,
    price: 1
  }
).sort({ review_scores_rating: -1 })
```

### 6.
I used aggregate function here to find the number of listings per host, displaying their host_id.
```
db.listings.aggregate([
  { $group: { _id: "$host_id", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
])
```

### 7.
Again, using the aggregate function, I displayed average review_scores_rating per neighborhood, and only showing those that are 4 or above (sorted in descending order).
```
db.listings.aggregate([
  { $match: { review_scores_rating: { $gte: 4 } } },
  { $group: { _id: "$neighbourhood", avg_rating: { $avg: "$review_scores_rating" } } },
  { $sort: { avg_rating: -1 } }
])
```
