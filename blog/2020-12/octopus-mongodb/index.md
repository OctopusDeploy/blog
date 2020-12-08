---
title: Deploying to MongoDB with Octopus Deploy and Liquibase
description: Use Octopus Deploy to deploy to MongoDB using Liquibase
author: shawn.sesna@octopus.com
visibility: private
published: 2021-06-29
metaImage: 
bannerImage: 
tags:
 
---

When you think of databases, what immediately comes to mind are massive datastores with 1:N relationships between tables and complex T-SQL statements to bring related information together.  This, however, is the only type of database technology that exists.  Though not a new concept, NoSQL databases have been gaining popularity in recent years.  One of the more recognizable names in the NoSQL world is MongoDB.  In this post, I'll demonstrate how to deploy to a MongoDB database using Octopus Deploy and Liquibase.

## Liquibase
I [previously](https://octopus.com/blog/octopus-oracle-liquibase) demonstrated how to deploy to Oracle using the Liquabase product.  However, our friends over at Liquibase do not operate strictly in the relational database space, they also have solutions for deploying to NoSQL as well, including MongoDB.

### Change log
As you might expect, the change log for MongoDB differs quite significantly than its relational counter parts.  For example, where you would create a Table in a relational database, you create a Collection in MongoDB:

```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.6.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <changeSet id="1" author="alex">
            <ext:createCollection collectionName="createCollectionWithValidatorAndOptionsTest">
                <ext:options>
                    {
                    validator: {
                        $jsonSchema: {
                            bsonType: "object",
                            required: ["name", "address"],
                            properties: {
                                name: {
                                    bsonType: "string",
                                    description: "The Name"
                                },
                                address: {
                                    bsonType: "string",
                                    description: "The Address"
                                }
                            }
                        }
                    },
                    validationAction: "warn",
                    validationLevel: "strict"
                    }
                </ext:options>
            </ext:createCollection>
        <ext:createCollection collectionName="createCollectionWithEmptyValidatorTest">
            <ext:options>
            </ext:options>
        </ext:createCollection>
        <ext:createCollection collectionName="createCollectionWithNoValidator"/>
    </changeSet>
</databaseChangeLog>
```

Similarly, inserting data into a Collection is very different as well:

```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.6.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <changeSet id="1" author="alex">
        <ext:insertMany collectionName="insertManyTest1">
            <ext:documents>
                [
                { id: 2 },
                { id: 3,
                  address: { nr: 1, ap: 5}
                }
                ]
            </ext:documents>
        </ext:insertMany>
    </changeSet>
</databaseChangeLog>
```

:::hint
More examples of MongoDB operations with Liquibase can be found [here](https://github.com/liquibase/liquibase-mongodb/tree/main/src/test/resources/liquibase/ext).
:::

For this post, my dbchangelog.xml file consisted of the following which is a copy of the [AirBNB example from MongoDB](https://docs.atlas.mongodb.com/sample-data/sample-airbnb):
<details>
	<summary>dbchangelog.xml</summary>
	

```xml
<?xml version="1.1" encoding="UTF-8" standalone="no"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.6.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <changeSet id="1" author="Shawn.Sesna">
            <ext:createCollection collectionName="Bookings">
            </ext:createCollection>
    </changeSet>
    <changeSet id="2" author="Shawn.Sesna">
        <ext:insertMany collectionName="Listings">
            <ext:documents>
                [
                    {
                    "_id": "10006546",
                    "listing_url": "https://www.airbnb.com/rooms/10006546",
                    "name": "Ribeira Charming Duplex",
                    "summary": "Fantastic duplex apartment with three bedrooms, located in the historic area of Porto, Ribeira (Cube)...",
                    "interaction": "Cot - 10 € / night Dog - € 7,5 / night",
                    "house_rules": "Make the house your home...",
                    "property_type": "House",
                    "room_type": "Entire home/apt",
                    "bed_type": "Real Bed",
                    "minimum_nights": "2",
                    "maximum_nights": "30",
                    "cancellation_policy": "moderate",
                    "last_scraped": {
                        "$date": {
                        "$numberLong": "1550293200000"
                        }
                    },
                    "calendar_last_scraped": {
                        "$date": {
                        "$numberLong": "1550293200000"
                        }
                    },
                    "first_review": {
                        "$date": {
                        "$numberLong": "1451797200000"
                        }
                    },
                    "last_review": {
                        "$date": {
                        "$numberLong": "1547960400000"
                        }
                    },
                    "accommodates": {
                        "$numberInt": "8"
                    },
                    "bedrooms": {
                        "$numberInt": "3"
                    },
                    "beds": {
                        "$numberInt": "5"
                    },
                    "number_of_reviews": {
                        "$numberInt": "51"
                    },
                    "bathrooms": {
                        "$numberDecimal": "1.0"
                    },
                    "amenities": [
                        "TV",
                        "Cable TV",
                        "Wifi",
                        "Kitchen",
                        "Paid parking off premises",
                        "Smoking allowed",
                        "Pets allowed",
                        "Buzzer/wireless intercom",
                        "Heating",
                        "Family/kid friendly",
                        "Washer",
                        "First aid kit",
                        "Fire extinguisher",
                        "Essentials",
                        "Hangers",
                        "Hair dryer",
                        "Iron",
                        "Pack ’n Play/travel crib",
                        "Room-darkening shades",
                        "Hot water",
                        "Bed linens",
                        "Extra pillows and blankets",
                        "Microwave",
                        "Coffee maker",
                        "Refrigerator",
                        "Dishwasher",
                        "Dishes and silverware",
                        "Cooking basics",
                        "Oven",
                        "Stove",
                        "Cleaning before checkout",
                        "Waterfront"
                    ],
                    "price": {
                        "$numberDecimal": "80.00"
                    },
                    "security_deposit": {
                        "$numberDecimal": "200.00"
                    },
                    "cleaning_fee": {
                        "$numberDecimal": "35.00"
                    },
                    "extra_people": {
                        "$numberDecimal": "15.00"
                    },
                    "guests_included": {
                        "$numberDecimal": "6"
                    },
                    "images": {
                        "thumbnail_url": "",
                        "medium_url": "",
                        "picture_url": "https://a0.muscache.com/im/pictures/e83e702f-ef49-40fb-8fa0-6512d7e26e9b.jpg?aki_policy=large",
                        "xl_picture_url": ""
                    },
                    "host": {
                        "host_id": "51399391",
                        "host_url": "https://www.airbnb.com/users/show/51399391",
                        "host_name": "Ana Gonçalo",
                        "host_location": "Porto, Porto District, Portugal",
                        "host_about": "Gostamos de passear, de viajar, de conhecer pessoas e locais novos, gostamos de desporto e animais! Vivemos na cidade mais linda do mundo!!!",
                        "host_response_time": "within an hour",
                        "host_thumbnail_url": "https://a0.muscache.com/im/pictures/fab79f25-2e10-4f0f-9711-663cb69dc7d8.jpg?aki_policy=profile_small",
                        "host_picture_url": "https://a0.muscache.com/im/pictures/fab79f25-2e10-4f0f-9711-663cb69dc7d8.jpg?aki_policy=profile_x_medium",
                        "host_neighbourhood": "",
                        "host_response_rate": {
                        "$numberInt": "100"
                        },
                        "host_is_superhost": false,
                        "host_has_profile_pic": true,
                        "host_identity_verified": true,
                        "host_listings_count": {
                        "$numberInt": "3"
                        },
                        "host_total_listings_count": {
                        "$numberInt": "3"
                        },
                        "host_verifications": [
                        "email",
                        "phone",
                        "reviews",
                        "jumio",
                        "offline_government_id",
                        "government_id"
                        ]
                    },
                    "address": {
                        "street": "Porto, Porto, Portugal",
                        "suburb": "",
                        "government_area": "Cedofeita, Ildefonso, Sé, Miragaia, Nicolau, Vitória",
                        "market": "Porto",
                        "country": "Portugal",
                        "country_code": "PT",
                        "location": {
                        "type": "Point",
                        "coordinates": [
                            {
                            "$numberDouble": "-8.61308"
                            },
                            {
                            "$numberDouble": "41.1413"
                            }
                        ],
                        "is_location_exact": false
                        }
                    },
                    "availability": {
                        "availability_30": {
                        "$numberInt": "28"
                        },
                        "availability_60": {
                        "$numberInt": "47"
                        },
                        "availability_90": {
                        "$numberInt": "74"
                        },
                        "availability_365": {
                        "$numberInt": "239"
                        }
                    },
                    "review_scores": {
                        "review_scores_accuracy": {
                        "$numberInt": "9"
                        },
                        "review_scores_cleanliness": {
                        "$numberInt": "9"
                        },
                        "review_scores_checkin": {
                        "$numberInt": "10"
                        },
                        "review_scores_communication": {
                        "$numberInt": "10"
                        },
                        "review_scores_location": {
                        "$numberInt": "10"
                        },
                        "review_scores_value": {
                        "$numberInt": "9"
                        },
                        "review_scores_rating": {
                        "$numberInt": "89"
                        }
                    },
                    "reviews": [
                        {
                        "_id": "362865132",
                        "date": {
                            "$date": {
                            "$numberLong": "1545886800000"
                            }
                        },
                        "listing_id": "10006546",
                        "reviewer_id": "208880077",
                        "reviewer_name": "Thomas",
                        "comments": "Very helpful hosts. Cooked traditional..."
                        },
                        {
                        "_id": "364728730",
                        "date": {
                            "$date": {
                            "$numberLong": "1546232400000"
                            }
                        },
                        "listing_id": "10006546",
                        "reviewer_id": "91827533",
                        "reviewer_name": "Mr",
                        "comments": "Ana Goncalo were great on communication..."
                        },
                        {  - Server Name: Name or IP address of the MongoDB server
  - Server Port: Port MongoDB is listening on

                        "_id": "403055315",
                        "date": {
                            "$date": {
                            "$numberLong": "1547960400000"
                            }
                        },
                        "listing_id": "10006546",
                        "reviewer_id": "15138940",
                        "reviewer_name": "Milo",
                        "comments": "The house was extremely well located..."
                        }
                    ]
                    }                
                ]
            </ext:documents>
        </ext:insertMany>
    </changeSet>
</databaseChangeLog>
```
</details>


## Octopus Deploy
Deploying to MongoDB is very similar to the Oracle method I demonstrated previously.  If you've read the Oracle post, you'll note that the Liquibase template has been updated to include MongoDB as a `Database type`.

My deployment project consists of the following steps:

- Create MongoDB Database
  - Server Name: Name or IP address of the MongoDB server
  - Server Port: Port MongoDB is listening on
  - Database Name: Name of the database to create 
  - Initial Collection: Name of a collection to create
  - Username: Username of an account that can create databases and collections
  - Password: Password for the username
- Create MongoDB User
  - Server Name: Name or IP address of the MongoDB server
  - Server Port: Port MongoDB is listening on
  - Admin Username: Username of an account that can create users.
  - Admin Password: Password for the Admin Username
  - Username: Username for the account to create
  - Password: Password for the User to create
- Add or update roles for user
  - Server Name: Name or IP address of the MongoDB server
  - Server Port: Port MongoDB is listening on
  - Database Name: Name of the database to grant roles to
  - Admin Username: Username of an account that can create users.
  - Admin Password: Password for the Admin Username
  - Roles: Comma-delimited list of Roles to assign
- Liquibase - Apply changeset
  - Pro license key: Empty
  - Database type: MongoDB
  - Change Log file name: dbchangelog.xml
  - Server Name: Name or IP address of the MongoDB server
  - Server Port: Port MongoDB is listening on
  - Database name: Name of the database to apply updates to
  - Username: Username with rights to update database
  - Password: Password for the user account
  - Connection query string parameters: ?authSource=admin
  - Database driver path: Empty
  - Executable file path: Empty
  - Report only?: Unchecked (not supported for NoSQL databases)
  - Download Liquibase: Checked
  - Liquibase Version: Empty
  - Changeset package: Package containing dbchangelog.xml

![](InsertMeHere.png)

Unlike the Oracle post, MongoDB requires the `Connection query string parameters` to be set to `?authSource=admin`.  The value of `admin` can of course be changed to whatever database is the [authorization database](https://docs.mongodb.com/manual/reference/connection-string/).

Using MongoDB Compass, we can verify that our database, collection, and data have been added.

