# MongoDB Notes

> ### Disclaimer: All notes and example queries are directly from MongoDB University Free Courses available on their website. Check them out.

1. **Collection** - an organized store of documents in MongoDB usually with common fields between documents. There can be many collections per database and many documents per collection.

2. **Document** - a way to organize and store data as a set of field-value pairs.

3. **Field** - a unique identifier for a datapoint.

4. **Value** - data related to a given identifier.

5. **Replica Set** - a few connected machines that store the same data to ensure that if something happens to one of the machines the data will remain intact. Comes from the word replicate - to copy something.

6. **Instance** - a single machine locally or in the cloud, running a certain software, in our case it is the MongoDB database.

7. **Cluster** - group of servers that store your data.

8. **SRV connection string** - a specific format used to establish a connection between your application and a MongoDB instance.

9. **Export**:

   ```js
   mongodump --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" // BSON Export

   mongoexport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --collection=sales --out=sales.json // JSON Export

   mongorestore --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --drop dump // BSON Import

   mongoimport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --drop sales.json // JSON Import [--drop is for dropping last version]
   ```

10. **Namespace** - The concatenation of the database name and collection name is called a namespace.

11. **Connecting to the cluster through mongo shell**

    ```cmd
    mongo mongodb+srv://<username>:<password>@<CLUSTER_NAME.ID>.mongodb.net/admin
    ```

12. **MongoDB Operations**

    ```js

    show dbs // lists databases in cluster

    use sample_training // selects the db named "sample_training"

    show collections // lists the collections inside the selected db

    db.zips.find({"state": "NY"}) // db here represents the selected db. zips is the collection being selected inside the db. find(<query:object>) is the command to filter the collection documents based on query

    db.zips.find({"state": "NY"}).count() // count() finds the number of documents that gets filtered by find().

    db.zips.find({"state": "NY", "city": "ALBANY"})

    db.zips.find({"state": "NY", "city": "ALBANY"}).pretty() // pretty prints the results in the shell

    it // iterates through the cursor results

    db.zips.findOne() // get a random document from the collection

    db.zips.insert(<new_object_with_data>) // insert one object

    db.zips.insert([<new_object_1>, <new_object_2> ...]) // insert many objects

    db.inspections.insert([ { "test": 1 }, { "test": 2 }, { "test": 3 } ]) // insert many objects example 1

    db.inspections.insert([{ "_id": 1, "test": 1 },{ "_id": 1, "test": 2 },{ "_id": 3, "test": 3 }],{ "ordered": false }) // insert many example 2. "ordered" false signifies skipping errors and inserting later objects anyways.

    db.zips.updateMany({ "city": "HUDSON" }, { "$inc": { "pop": 10 } }) // Update all documents in the zips collection where the city field is equal to "HUDSON" by adding 10 to the current value of the "pop" field.

    db.zips.updateOne({ "zip": "12534" }, { "$set": { "pop": 17630 } }) // Update a single document in the zips collection where the zip field is equal to "12534" by setting the value of the "pop" field to 17630.

    db.grades.updateOne({ "student_id": 250, "class_id": 339 },{ "$push": { "scores": { "type": "extra credit", "score": 100 }}}) // Update one document in the grades collection where the student_id is ``250`` *, and the class_id field is 339 , by adding a document element to the "scores" array.

    db.inspections.deleteMany({ "test": 1 }) // Delete all the documents that have test field equal to 1.

    db.inspections.deleteMany({ "test": 1 }) // Delete one document that has test field equal to 3.

    db.inspection.drop() // Drop the inspection collection

    /**
     * Operators: $eq (equal to), $ne (not equal to), $lt (less than), $gt (greater than), $gte, $lte
     * $eq is default
     */

    db.trips.find({ "tripduration": { "$lte" : 70 },"usertype": "Customer" }).pretty()

    /** Logical Operators
     * $and: Matches all of the specified query clauses
     * $or: At least one of the query clauses is matched
     * $nor: Fail to match both given clauses
     * $not: Negates the query requirement
     *
     * $and is default
    */

    db.routes.find({ "$and": [
            { "$or" :[
                        { "dst_airport": "KZN" },
                        { "src_airport": "KZN" }
                     ]
            },
            { "$or" :[
                        { "airplane": "CR2" },
                        { "airplane": "A81" }
                     ]
            }
        ]
    }).pretty()

    // The only time we use explicit $and is when we need to include the same operator more than once in a clause. Here $or is that operator.

    // Without $and in above case, we will get flights where src_airport is KZN or dst_airport is KZN or airplane is CR2 or airplane is A81

    db.zips.find({"pop": {"$lt": 5000}}).count()
    db.zips.find({"pop": {"$gt": 1000000}}).count()
    db.companies.find(
        {
            "$or": [
                {"founded_year": 2004, "category_code": "web"},
                {"founded_year": 2004, "category_code": "social"},
                {"founded_month": 10, "category_code": "web"},
                {"founded_month": 10, "category_code": "social"}
            ],
        }
    ).count()

    // $expr operator -> Expressive Query Operator. Allows the use of aggregation operators within the query. Here, $gt is aggregation operator.
    db.trips.find({ "$expr": { "$and": [ { "$gt": [ "$tripduration", 1200 ]},
                         { "$eq": [ "$end station id", "$start station id" ]}
                       ]}}).count() // $ denotes an operator and signifies that we are looking at the value of the field rather than field name

    /**
     * Array Operators: $size, $all
     */
    db.listingsAndReviews.find({ "amenities": {
                                  "$size": 20,
                                  "$all": [ "Internet", "Wifi",  "Kitchen",
                                           "Heating", "Family/kid friendly",
                                           "Washer", "Dryer", "Essentials",
                                           "Shampoo", "Hangers",
                                           "Hair dryer", "Iron",
                                           "Laptop friendly workspace" ]
                                         }
                            }).pretty() // Find all documents with exactly 20 amenities which include all the amenities listed in the query array:

    /**
     * Projection: To project only the values of fields specified
     */
    db.listingsAndReviews.find({ "amenities": "Wifi" },
                           { "price": 1, "address": 1, "_id": 0 }).pretty() // only price and address fields will be shown in the result. Also, don't use inclusion (1) and exclusion(0) simultaneously unless it's for _id field.

    db.grades.find({ "class_id": 431 },
               { "scores": { "$elemMatch": { "score": { "$gt": 85 } } }
             }).pretty() // Find all documents where the student in class 431 received a grade higher than 85 for any type of assignment:

    db.trips.findOne({ "start station location.type": "Point" })

    db.companies.find({ "relationships.0.person.last_name": "Zuckerberg" },
                    { "name": 1 }).pretty()

    db.companies.find({ "relationships.0.person.first_name": "Mark",
                        "relationships.0.title": { "$regex": "CEO" } },
                    { "name": 1 }).count()


    db.companies.find({ "relationships.0.person.first_name": "Mark",
                        "relationships.0.title": {"$regex": "CEO" } },
                    { "name": 1 }).pretty()

    db.companies.find({ "relationships":
                        { "$elemMatch": { "is_past": true,
                                            "person.first_name": "Mark" } } },
                    { "name": 1 }).pretty()

    db.companies.find({ "relationships":
                        { "$elemMatch": { "is_past": true,
                                            "person.first_name": "Mark" } } },
                    { "name": 1 }).count()
    ```

    _mongo shell_ is a fully functional javascript interpreter. It helps interact with mongo instance without the use of GUI

13. **Aggregation Framework Command:**

    ```js
    db.listingsAndReviews
      .aggregate([
        { $match: { amenities: "Wifi" } },
        { $project: { price: 1, address: 1, _id: 0 } },
      ])
      .pretty(); // Using the aggregation framework find all documents that have Wifi as one of the amenities``*. Only include* ``price and address in the resulting cursor.

    db.listingsAndReviews.aggregate([
      { $project: { address: 1, _id: 0 } },
      { $group: { _id: "$address.country", count: { $sum: 1 } } },
    ]); // Project only the address field value for each document, then group all documents into one document per address.country value, and count one for each document in each group.

    // Any find() query can be translated into an aggregation pipeline equivalent, but not every aggregation pipeline can be translated into a find() query.

    // The aggregation framework allows us to compute and reshape data via using stages like $group, $sum, and others.

    /** Cursor Methods: pretty(), count(), sort(), limit() - A cursor method is not applied to the data stored in the database. It is instead applied to the resultset that lives in the cursor.
     *
     * Order matters here for cursor methods. sort().limit() -> adds limit to the sorted resultset. limit().sort() -> limits the resultset and then applies the sort operation.
     * */
    db.zips.find().sort({ pop: 1 }).limit(1); // pop: 1 -> decreasing

    db.zips.find({ pop: 0 }).count(); // pop: 0 -> No sorting

    db.zips.find().sort({ pop: -1 }).limit(1); // pop: -1 -> increasing

    db.zips.find().sort({ pop: -1 }).limit(10); // limit: 10 -> 10 top results based on pop modification

    db.zips.find().sort({ pop: 1, city: -1 }); // city: -1 -> decreasing [Z->A]

    db.companies
      .find({ founded_year: { $ne: null } }, { name: 1, founded_year: 1 })
      .limit(5)
      .sort({ founded_year: 1 }); // While the limit() and sort() methods are not listed in the correct order, MongoDB flips their order when executing the query, delivering the results that the question prompt is looking for.
    ```
14. **Indexing

    ```js
    /**
     * Indexing is the data structure that is used to make our query faster. Since memory consumption and time to sort the data increases with the amount of data, we need it.
     */
    db.trips.createIndex({ "birth year": 1 });

    db.trips.createIndex({ "start station id": 1, "birth year": 1 });
    ```

15. **Data modeling:** a way to organize fields in a document to support your application performance and querying capabilities.

16. **Upsert [Update or Insert?]
    ```js
    db.iot.updateOne(
      { sensor: r.sensor, date: r.date, valcount: { $lt: 48 } },
      {
        $push: { readings: { v: r.value, t: r.time } },
        $inc: { valcount: 1, total: r.value },
      },
      { upsert: true }
    );
    ```
