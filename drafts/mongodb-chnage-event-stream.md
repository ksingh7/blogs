![](https://raw.githubusercontent.com/ksingh7/blogs/main/posts/assets/mongodb-change-streams.png)

## What are Change Streams ?
Change streams is a near real-time ordered flow of information (stream) about any change to item in a database, table/collection, or row of a table / document in a collection. For example, whenever any update (Insert, Update or Delete) occurs in a specific collection/table, the database triggers a change event with all the data which has been modified.

## MongoDB Change Streams
MongoDB change streams provide a high-level API that can notify an application of changes to a MongoDB database, collection, or cluster, without using polling(which would come with much higher overhead). Characteristics of MongoDB Change Streams are:
- Filterable
	- Applications can filter changes to receive only those change notifications they need.
- Resumable 
	- Change streams are resumable because each response comes with a resume token. Using the token, an application can start the stream where it left off (if it ever disconnects).
- Ordered
	- Change notifications occur in the same order that the database was updated.
- Durable
	- Change streams only include majority-committed changes. This is so every change seen by listening applications is durable in failure scenarios, such as electing a new primary.
- Secure 
	- Only users with rights to read a collection can create a change stream on that collection.
- Easy to use
	- The syntax of the change streams API uses the existing MongoDB drivers and query language.

## Experimenting with MongoDB Change Stream using Golang

### Prerequisites

- MongoDB Atlas Cluster, get it for free at https://www.mongodb.com/cloud/atlas

### Getting Started with MongoDB Streams : Golang Implementation
```
# export MongoDB URI

export MONGODB_URI="mongodb+srv://admin:xxxxx@cluster0.ii90w.mongodb.net/myFirstDatabase?retryWrites=true&w=majority"

git clone https://github.com/ksingh7/mongodb-change-events-go.git
cd mongodb-change-events-go
go mod tidy
go run main.go
```

### Code Walkthrough

> `main.go` file already has required guidelines in the form of comments. However, in this section I will explain sections that I think are crucial
- Declaring struct returned by MongoDB Stream API 
```go
type DbEvent struct {
	DocumentKey     documentKey     `bson:"documentKey"`
    OperationType    string                   `bson:"operationType"`
}
type documentKey struct {
	ID      primitive.ObjectID      `bson:"_id"`
}
```
- Declaring a struct that resembles to the collection
```go
type result struct {
    ID               primitive.ObjectID       `bson:"_id"`
    UserID        string                            `bson:"userID"`
    DeviceType string                            `bson:"deviceType"`
    GameState   string                            `bson:"gameState"`
}
```
- Connect to MongoDB
```go
    client, err := mongo.Connect(context.TODO(), options.Client().ApplyURI(os.Getenv("MONGODB_URI")))
    if err != nil {
        panic(err)
    }
```
- Set DB and Collection names
```go
    database := client.Database("summit-demo")
    collection := database.Collection("bike-factory")
```
- Create a change stream
```go
	changeStream, err := collection.Watch(context.TODO(), mongo.Pipeline{})
	if err != nil {
		panic(err)
	}
```
- Iterate over the change stream
```go
	for changeStream.Next(context.TODO()) {
		change := changeStream.Current
		fmt.Printf("%+v\n", change)
	}
```
- Detect change type (Insert or Update) and accordingly fetch the document
```go
        // Print out the document that was inserted or updated
        if DbEvent.OperationType == "insert" ||  DbEvent.OperationType == "update" {
            // Find the mongodb document based on the objectID
            var result result
            err  := collection.FindOne(context.TODO(), DbEvent.DocumentKey).Decode(&result)
            if err != nil {
                log.Fatal(err)
            }
            // Convert changd MongoDB document from BSON to JSON
            data, writeErr := bson.MarshalExtJSON(result, false, false)
            if writeErr != nil {
                log.Fatal(writeErr)
            }
            // Print the changed document in JSON format
            fmt.Println(string(data))
            fmt.Println("")
        }
```
- Close the change stream
```go
	if err := changeStream.Close(context.TODO()); err != nil {
		panic(err)
	}
```
### Bonus : Function to Insert records to MongoDB collection
```go
func insertRecord(collection *mongo.Collection) {
        // pre-populated values for DeviceType and GameState    
        DeviceType := make([]string, 0)
        DeviceType = append(DeviceType, "mobile","laptop","karan-board","tablet","desktop","smart-watch")
        GameState := make([]string, 0)
        GameState = append(GameState, "playing","paused","stopped","finished","failed")

        // insert new records to MongoDB every 5 seconds
        for {
            item := result{
                ID: primitive.NewObjectID(),
                UserID: strconv.Itoa(rand.Intn(10000)),
                DeviceType: DeviceType[rand.Intn(len(DeviceType))],
                GameState: GameState[rand.Intn(len(GameState))],
            }
            _, err := collection.InsertOne(context.TODO(), item)
            if err != nil {
                log.Fatal(err)
            }
    
            time.Sleep(5 * time.Second)
        }
    }
```

### Summary
Hope this post gives you a better understanding of MongoDB Change Streams and how to use them in your application.

![](https://raw.githubusercontent.com/ksingh7/blogs/main/posts/assets/thats-all-folks.gif)