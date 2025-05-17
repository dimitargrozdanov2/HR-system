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
