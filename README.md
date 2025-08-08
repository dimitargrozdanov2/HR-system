# HR system
This HR system is used to manage:

• Employees

• Salaries

• Vacations

• Payments

• Bonuses

• Promotions


# System Requirements
• Web-based application.

• Performs CRUD operations on the employees.

• Manages salaries:

• Allows the manager to ask for an employee’s salary
change.

• Allows the HR manager to approve or reject a request.

• Manages vacation days.

• Manages promotions & bonuses.

• Uses an external payment system.

# System usage
• 10 concurrent users.

• 250 employees. Will rise to 500 in 5 years.

• Payment processor -  Legacy system written in C++. Hosted in the company’s server farm and works only with a single file that is sent once a month.

# Data volume
Each employee holds around 1MB of data. Every hired person has 10 documents - contracts, reviews, etc.
Each document is around 5MB. A total of 51MB per employee.
51 x 500 = 25.5GB 

# SLA 
The Service Level Agreement says it is not critical to have downtime.

# Architecture diagrams

## Logical Diagram
<img width="2848" height="3272" alt="image" src="https://github.com/user-attachments/assets/9f7f4304-2d92-4a9f-818a-81da9095ec40" />

## Technical Diagram
<img width="708" height="408" alt="image" src="https://github.com/user-attachments/assets/205fda07-b399-43a3-bdc5-81b81300427c" />

# Tech stack
Technology stack would be .NET and SQL Server and React.

# Services and module architecture explanation
## Logging Service
Logging service is a critical service that will output errors from all services to a database.

Logging will be a background services that runs on a schedule. Messages will be processed and validated from an asynchronous queue.  
Then they will be recorded in a database. 

For this service a classic 3-layer architecture will be used:
Queue Polling -> Business Logic - Data Access 

A good practice is to have 3 replicas of this critical service, however, as it is a demo project, I would go for 2 instances using Active-Passive architectural pattern.

A Polly library is also used for exceptions with a maximum retry count of 3.

Dependancy Injection is used to promote loose coupling and as EF Core is used as an ORM. EF already uses Repository and UOW patterns, so those will not be implemented.

## View Service
View service serves static files such as HTML, CSS and JS and has no business logic.

To tackle redundancy, a load balancer is introduced to balance two view service instances.

## Employees Service
Employees service perform CRUD operations on all employees. 

The architercture is again following the classic 3-layer archicture: 
Service Interface -> Business Logic -> Data store 

The following APIs are described, along with their status code and a short description:
• GET /api/v1/employees/{id}  200, 401, 404 -> all information about an employee along with documents 

• GET /api/v1/employees?name=..&birthplace=.. 200, 400, 401 -> employee object with information and number of documents stored 

• POST /api/v1/employees  / 201, 400, 401 -> employee object with information that has been created

• PUT /api/v1/employees/{id}  200, 400, 401, 404 -> employee updated object 

• DELETE /api/v1/employees/{id}  200, 400, 401, 404 -> bool which indicates success or fail 

## Documents Service
Document service supports adding a document, removing a document, getting a particular document and getting a list of documents for an employee.

Initially it was a separate service, but as it was linked to an employee it will be based in the Employees Service.  The separation of concerns pattern is followed, splitting the controllers. 

The following APIs are described, along with their status code and a short description:

• GET /api/v1/employees/{id}/documents 200,401, 404

• GET /api/v1/employees/{id}/documents?employeename=..&filename=.. 200,400, 401

• POST /api/v1/employees/{id}/documents 201, 401 -> create document in azure or amazon 

• DELETE api/v1/employees/{id}/documents - 200, 400, 401, 404  -> bool which indicates success or fail 

There were 2 options of storing the files:

* Option 1 - relational database and store them in a specialized column for blobs.

* Option 2 - cloud storage (Azure's Storage or AWS S3)

I have decided on Option 2 due to its great scale and easy to execute even though it may be costly.

All exceptions will be logged and passed to the Logging service:

* Service layer exception will be retried a maximum number of 3 times.
  
* Business layer will be wrapped in a transaction and rollbacked

* Data exceptions will log the exception and the query.

As before a redundancy will be achived by 2 pods with one load balancer. 

## Salary Service
Salary service can create or delete a request to change an employee's salary by his direct manager. The request can be approved or rejected by the HR manager.

Two roles should be created - one for a manager, and another for a HR manager.

Salary service will have the following functionality:

* Get all salary requests

* Create a salary request

* Remove a salary request

* Approve a salary request

* Reject a salary request

The following APIs are described, along with their status code and a short description:

* GET api/v1/salaryRequests - 200, 401 get all salary requests

* POST /api/v1/salaryRequests 201, 400, 401  -> create a salary request

* DELETE /api/v1/salaryRequests/{id} 200, 400, 401, 404 -> remove a salary request(soft delete)

* PUT /api/v1/salaryRequests/{id}?approve=yes 200, 400, 401, 404  -> one salary request for both approve and reject. The bool "approved" will be nullable in the database and it will be used in get all.

HR request will be kept in a separate table. Obviously salaries, would have an updated by column which would list the id of the manager.

View Service -> BL layer -> Data store 

For redundancy again 2 pods with one load balancer.

## Vacation Service
Vacation service allows employees to manage their vacation days and allow HR representative to set available vacation days for employees.

* Get all vacations
  
* Create a vacation

* Remove a vacation

* Set available days for vacation

* Remove available days for vacation

The following APIs are described, along with their status code and a short description:

* GET /api/v1/employees/{id}/vacations - 200, 401 get all vacations

* POST /api/v1/employees/{id}/vacations 201, 400, 401  -> create a vacation

* DELETE /api/v1/employees/{id}/vacations 200, 400, 401, 404 -> remove a vacation(soft delete)

* POST /api/v1/vacations 200, 400, 401, 404  -> create a vacation request that takes parameters - from date to date

* DELETE /api/v1/vacations/{id} 200,400, 401, 404 -> delete a vacation request (soft delete)

View Service -> BL layer -> Data store 

For redundancy again 2 pods with one load balancer.

## External Payment Service
External Payment Service queries the database for a salary data once a month and sends it to the external payment system.

Background scheduled job with Quartz -> BL -> Data store

We will use the same Active - Passive architectural pattern as in the Logging Service.

We don't need any caching and the exception handling strategy should be the same as above.

## Asynchronous Queue
The role of the queue is to process errors, validate them and pass them to the Logging service.

There are two options as technologies:
• RabbitMQ

• Apache Kafka

Rabbit is a general purpose message-broker engine that is easy to setup and easy to use. Apache kafka is a stream processing platform that is perfect for high load.
We will choose RabbitMQ.

For redundancy we can mirror and enable more than one queue simultaneously. As it is complex to setup and our queue does not serve a crucial data role, in offline scenarions,
we can log that the queue is offline, as we will not have the full log information. 

An optional thing to do is to implement a Dead Letter Queue.

# Infrastructure 

• CI/CD - Jenkins

• Containerization - Docker

• Orchestration - Kubernetes

• Environment infrastructure - data centers







