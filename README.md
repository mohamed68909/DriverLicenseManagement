# Driver License Management System 🚗📋

A C# and SQL Server-based desktop application designed to help manage driver licenses, applicants, and driving tests efficiently.

## 🧠 Project Overview

DVLD (Driver and Vehicle License Department) is a Windows Forms desktop application that provides:

- Management of person records.
- Issuing, renewing, and canceling driver licenses.
- Organizing theoretical and practical driving tests.
- Tracking license types and their requirements.
- Integration with a SQL Server database.

## 🏗️ Project Structure

The project follows a 3-tier architecture and consists of:

1. **DVLD**  
   The User Interface (UI) built using Windows Forms where users interact with the system.

2. **DVLD_Buisness**  
   The Business Logic Layer containing core functionalities such as validation rules, license operations, and permissions logic.

3. **DVLD_DataAccess**  
   The Data Access Layer (DAL) that manages the database connection and executes SQL queries using ADO.NET.

## 🧰 Technologies Used

- C#
- .NET Framework
- Windows Forms
- ADO.NET
- SQL Server

## ⚙️ How to Run the Project

1. Open the solution in Visual Studio.
2. Make sure SQL Server is installed and the database is set up correctly.
3. Update the connection string in the configuration file to match your environment.
4. Build and run the application.


