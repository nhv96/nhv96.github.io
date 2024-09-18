---
layout: post
title:  "Dependency Injection in Go"
date:   2024-09-18 21:59:30 +0700
category: design-pattern
---
# What is Dependency Injection?

Dependency Injection (DI) is a technique that classes or functions, should only receive other classes or functions that it required, without having to know or initializing them. This design pattern will separates the concerns of using other functions or services from actual implementing them.

This way, our code will be loosely decoupled to external code that using them or internal code that it needs, leads to a program that easy to maintain and extend it's functionalities.

Let's have a look at this code snippet.

```go
type BookingService struct {
    MongoDB
}

func (s *BookingService) RegisterBooking(ctx context.Context, b BookingInfo) Response {
    err := s.MongoDB.Insert(ctx, b)
    if err != nil {
        return Response{
            Code: 500,
            Msg: "Internal error",
            Data: err,
        }
    }

    return Response{
        Code: 200,
        Msg: "Succesfully recorded booking order",
        Data: nil,
    }
}

func NewBookingService() *BookingService {
    mgoDB := NewMongoDB()

    s := &BookingService{
        MongoDB: mgoDB,
    }

    return s
}
```

In the above snippet, we have a `BookingService`, it's job is to receive a booking order from a user, store that booking data into Mongo database, and then return a message to the caller indicating that the operation succeeded if there's no error.

Then we have a function to initialize the `BookingService`, along with creating a MongoDB object and assigns it to the struct.

In real world, this code should be able to run without any problem. But it has many limitations.

- What if we want to use a different kind of database object such as PostgreSQL?

  *We'll have to replace the **MongoDB** object in the `BookingService` struct with **PostgresDB**, rewrite the line initializing DB object in construction function, rewrite the line inserting data.*

- What if the creation of MongoDB object needs to take in host URL or usename and password?

  *We'll have to modify the construction code that creating MongoDB object and pass in new arguments.*

- This code has no unit test, how to implement one?

  *Well this is kinda problematic... we will have to instantiate a real database connection to use in construction function. But that will defeat the whole point of unit test, right?*

# Implementing Dependency Injection

Give the criterias above, let's re-design the `BookingService` and it's construction code to satisfy the needs.

As mentioned above, DI is a design pattern, an approach to solve our problem. DI can be implemented in any programming languages, and specifically in Go, we can use *interface* type to do that.


```go
type Database interface{
    Insert(ctx context.Context, data interface{}) error
}

type BookingService struct{
    DB Database
}

func (s *BookingService) RegisterBooking(ctx context.Context, b BookingInfo) Response {
    err := s.DB.Insert(ctx, b)
    if err != nil {
        return Response{
            Code: 500,
            Msg: "Internal error",
            Data: err,
        }
    }

    return Response{
        Code: 200,
        Msg: "Succesfully recorded booking order",
        Data: nil,
    }
}

func NewBookingService(db Database) *BookingService {
    s := &BookingService{
        DB: db,
    }

    return s
}
```

As you can see, we basically rewrite the whole `BookingService`, from how it's run to how it's constructed, but this effort come with great rewards.

From now on, whenever we want to use a different database for our service, we only need to pass in the implementation object that can satisfy the requirements of `Database` interface, which is, that object must implement method ***Insert(context.Context, interface{}) error***.

Some use cases:
```go
type mgodb struct {
    Host string
    Username string
    Password string
}

func (m *mgodb) Insert(ctx context.Context, data interface{}) error {
    // implement logic to insert document to mgodb
    return nil
}

func NewMongoDB(h, u, p string) *mgodb {
    return &mgodb{Host: h, Username: u, Password: p}
}

type postgresdb struct {}

func (p *postgresdb) Insert(ctx context.Context, data interface{}) error {
    // implement logic to insert data to postgresdb
    return nil
}

func NewPostgresDB() *postgresdb {
    return &postgresdb{}
}

func main() {
    // mongodb
    mgoDB := NewMongoDB("localhost:27017", "user_name", "p1234")

    // or if you want to use postgres?
    pgDB := NewPostgresDB()

    // now both mgoDB or pgDB are acceptable for the function to construct
    // a booking service
    s := NewBookingService(mgoDB)

    resp := s.RegisterBooking(context.Background(), BookingInfo{User: "Peter", Room: "501"})
    // handle errors...
}

```

Not only we allowed the service to freely use any kind of database, but now it also doesn't have to worry if the construction of MongoDB or PostgresDB needs to take in new arguments. Meaning, changes apply to external logic won't affect the internal logic of `BookingService`.

And here is the best part, we now can implement unit test to check if the `RegisterBooking` method is functioning exactly as we wanted.

```go
type mockDB struct{}

func (m *mockDB) Insert(ctx context.Context, data interface{}) error {
    // assert type to BookingInfo
    bi, ok := data.(BookingInfo)
    if !ok {
        return errors.New("failed to assert type")
    }

    // if user is Peter, return nil as we succeeded insert to DB
    if bi.User == "Peter" {
        return nil
    }

    // otherwise, return an error to simulate something wrong happened
    return errors.New("failed to insert")
}

func newMockDB() Database {
    // you can use any mocking framework in Go,
    // or for the simplicity in this article, we can implement it by ourself
    return &mockDB{}
}

func TestBookingService(t *testing.T) {
    // create a mock object that will simulate or track the logic and arguments it use
    mockDB := newMockDB()

    s := NewBookingService(mockDB)

    t.Run("happy case", func (t *testing.T)  {
        resp := s.RegisterBooking(context.Background, BookingInfo{User: "Peter", Room: "501"})

        if resp != nil {
            t.Fail()
        }
        // response must have code 200 and a message as expected
        if resp.Code != 200 || resp.Msg != "Succesfully recorded booking order" {
            t.Fail()
        }
    })

    t.Run("case DB must return error", func (t *testing.T) {
        resp := s.RegisterBooking(context.Background, BookingInfo{User: "Someone", Room: "501"})

        // we expected there will be a response with error here
        if resp == nil {
            t.Fail()
        }

        if resp.Code != 500 || resp.Msg != "failed to insert" {
            t.Fail()
        }
    })

}
```

# Conclusion

Dependency Injection is a handy design pattern that solves many problem in software development. It helps create software that easy to maintain, add functionalities, testing, debugging,... In Go, we can leverage *interface* to implement Dependency Injection any many other design patterns.