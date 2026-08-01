# Employee Lifecycle & Asset Management Portal

A Salesforce Administrator project developed as part of the **Salesforce Admin Internship Assignment**. This application automates employee lifecycle management by managing employees, company assets, onboarding approvals, and leave requests using Salesforce declarative features.

---

## 📌 Project Overview

The Employee Lifecycle & Asset Management Portal replaces manual employee management processes with a centralized Salesforce application. It enables HR and IT teams to efficiently manage employees, track company assets, automate onboarding approvals, and monitor leave requests.

The project is built entirely using Salesforce's declarative tools without Apex.

---

## 🚀 Features

### 👨‍💼 Employee Management

* Manage employee records
* Store department, designation, manager, and joining date
* Track employment status
* Maintain employee contact information

### 💻 Asset Management

* Manage company assets
* Track laptops, desktops, mobile devices, and accessories
* Assign assets to employees
* Monitor asset availability and status

### ✅ Employee Onboarding

* Create onboarding requests
* Approval Process for onboarding
* Automatic employee status updates
* Record Triggered Flow automation
* Screen Flow for onboarding form

### 📝 Leave Management

* Create leave requests
* Track leave status
* Store leave remarks and request dates
* Leave approval workflow

---

# Salesforce Features Used

* Custom Objects
* Custom Fields
* Lookup Relationships
* Lightning App
* Page Layouts
* Validation Rules
* Formula Fields
* Approval Process
* Record Triggered Flow
* Screen Flow
* Permission Sets
* Sharing Settings
* Reports
* Dashboards
* List Views
* Data Import Wizard

---

# Custom Objects

| Object             | Description                     |
| ------------------ | ------------------------------- |
| Employee           | Stores employee information     |
| Asset              | Stores company asset details    |
| Onboarding Request | Tracks onboarding approvals     |
| Leave Request      | Manages employee leave requests |

---

# Object Relationships

```text
Employee
   │
   ├──────────────► Asset
   │
   ├──────────────► Onboarding Request
   │
   └──────────────► Leave Request
```

---

# Automation Implemented

## Approval Process

Employee Onboarding Approval

Workflow:

```
Submit Request
        │
        ▼
Manager Approval
        │
 ┌──────┴──────┐
 │             │
 ▼             ▼
Approved    Rejected
 │             │
 ▼             ▼
Employee      Status Updated
Activated     as Rejected
```

---

## Record Triggered Flow

**Flow Name**

```
Activate Employee after Onboarding Approval
```

Purpose

* Triggered when an onboarding request is approved.
* Updates the related employee's employment status to **Active**.

---

## Screen Flow

**Flow Name**

```
Employee Onboarding Form
```

Purpose

* Collect employee onboarding details.
* Automatically create an onboarding request.

---

# Validation Rules

### Employee

* Joining Date cannot be in the future.

### Asset

* Assigned Employee is mandatory when Asset Status is Assigned.

### Leave Request

* End Date must be greater than or equal to Start Date.

---

# Reports

* Employees by Department
* Active vs Inactive Employees
* Assets by Status
* Pending Onboarding Requests
* Leave Requests Summary

---

# Dashboard Components

* Employees by Department
* Active Employees
* Asset Status Distribution
* Pending Approvals
* Leave Request Summary

---

# Permission Sets

### HR Manager

Permissions:

* Employee
* Asset
* Onboarding Request
* Leave Request

Object permissions include:

* Read
* Create
* Edit

---

# Sample Data

| Data                | Records |
| ------------------- | ------- |
| Employees           | 30      |
| Assets              | 20      |
| Onboarding Requests | 10      |
| Leave Requests      | 5       |

---

# Project Structure

```
Employee Lifecycle & Asset Management Portal
│
├── Employee Management
├── Asset Management
├── Onboarding Request
├── Leave Request
├── Approval Process
├── Record Triggered Flow
├── Screen Flow
├── Reports
├── Dashboards
└── Permission Sets
```

---

# Technologies Used

* Salesforce Lightning Experience
* Salesforce Flow Builder
* Approval Process
* Reports & Dashboards
* Data Import Wizard
* Permission Sets
* Validation Rules

---

# Learning Outcomes

Through this project, the following Salesforce Administrator concepts were implemented:

* Salesforce Data Model
* Custom Objects and Fields
* Object Relationships
* Declarative Automation
* Approval Processes
* Flow Builder
* Validation Rules
* Security and Permissions
* Reporting and Analytics
* Lightning Experience Configuration

---

# Future Enhancements

* Email Notifications
* Asset Return Workflow
* Employee Offboarding Module
* Experience Cloud Portal
* Chatter Integration
* Mobile Optimization

---

# Author

**Kedar Naygaonkar**

Salesforce Administrator Internship Project

---

# License

This project was developed for educational and internship purposes as part of the Salesforce Administrator Internship Assignment.
