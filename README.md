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

# Architecture diagram
<img width="2804" height="3000" alt="image" src="https://github.com/user-attachments/assets/54c73f2f-ac1c-470e-abf6-5d77ae297c19" />

# Tech stack
Technology stack would be .NET and SQL Server and React.

# Services and module architecture explanation
## Logging Service
Logging service is a critical service that will output errors from all services to a database.

Logging will be a background services that runs on a schedule. Messages will be processed and validated from an asynchronous queue.  
Then they will be recorded in a database. The messages importance will be from Low to Critical.

For this service a classic 3-layer architecture will be used:
UI Layer -> Business Logic - Data Access 

A good practice is to have 3 replicas of this critical service, however, as it is a demo project, I would go for 2 instances using Active-Passive architectural pattern.

A Polly library is also used for exceptions with a maximum retry count of 3.

Dependancy Injection is used to promote loose coupling and as EF Core is used as an ORM. EF already uses Repository and UOW patterns, so those will not be implemented.

## View Service
View service serves static files such as HTML, CSS and JS and has no business logic.

To tackle redundancy, a load balancer is introduced to balance two view service instances.



