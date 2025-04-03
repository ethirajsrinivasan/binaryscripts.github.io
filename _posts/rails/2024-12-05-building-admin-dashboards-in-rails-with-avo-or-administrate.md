---
layout: post  
title: Building Admin Dashboards in Rails with Avo or Administrate  
subtitle: Create powerful admin panels in Rails using Avo and Administrate  
categories: RubyOnRails  
tags: ["Rails", "Admin Dashboard", "Avo", "Administrate", "ActiveAdmin", "CRUD", "Scalability"]  
excerpt: Learn how to build admin dashboards in Ruby on Rails using Avo and Administrate, two powerful gems that simplify CRUD operations and data management.  
---


Admin dashboards are essential for managing application data, providing non-technical users with an intuitive interface for CRUD operations.

🚀 **In this guide, we will explore:**  
✔️ **Why you need an admin dashboard**  
✔️ **How Avo and Administrate work**  
✔️ **Step-by-step setup of each**  
✔️ **Comparison and choosing the right tool**

---

#### **1. Why Use an Admin Dashboard in Rails?**
Instead of manually interacting with the database using `rails console` or `ActiveRecord queries`, an **admin panel** allows:

✅ **Easier Data Management** – Create, read, update, and delete records with a UI.  
✅ **User-Friendly Controls** – Non-developers can manage data efficiently.  
✅ **Access Control** – Restrict who can modify critical data.  
✅ **Performance Monitoring** – View logs, analytics, and background jobs.

While **ActiveAdmin** has been a popular choice, **Avo and Administrate** provide modern, customizable solutions.

---

#### **2. Setting Up an Admin Panel with Avo**
[Avo](https://avohq.io) is a premium, modern admin panel that focuses on **developer experience, performance, and extensibility**.

##### **Step 1: Install Avo**
Add Avo to your `Gemfile`:  
```rb  
gem "avo"  
```  
Run:  
```sh  
bundle install  
rails generate avo:install  
rails db:migrate  
```

##### **Step 2: Define Your Resources**
Create an admin panel for the `User` model:  
```sh  
rails generate avo:resource User  
```

Modify the generated file (`app/avo/resources/user_resource.rb`):  
```rb  
class UserResource < Avo::BaseResource  
self.title = :email  
field :id, as: :id  
field :email, as: :text  
field :created_at, as: :date_time  
end  
```

##### **Step 3: Start the Avo Dashboard**
```sh  
rails server  
```  
Visit **`http://localhost:3000/admin`** to access your panel! 🚀

---

#### **3. Setting Up an Admin Panel with Administrate**
[Administrate](https://github.com/thoughtbot/administrate) is an open-source alternative that **auto-generates dashboards** for your models.

##### **Step 1: Install Administrate**
```sh  
bundle add administrate  
rails generate administrate:install  
```

##### **Step 2: Generate a Dashboard**
```sh  
rails generate administrate:dashboard User  
```

##### **Step 3: Customize the Dashboard**
Modify `app/dashboards/user_dashboard.rb`:  
```rb  
class UserDashboard < Administrate::BaseDashboard  
ATTRIBUTE_TYPES = {  
id: Field::Number,  
email: Field::String,  
created_at: Field::DateTime  
}.freeze  
end  
```

##### **Step 4: Start the Server**
```sh  
rails server  
```  
Visit **`http://localhost:3000/admin`** 🎉

---

#### **4. Avo vs. Administrate: Choosing the Right Tool**
| Feature         | Avo | Administrate |  
|----------------|-----|-------------|  
| **Cost**      | Paid  | Free (Open Source) |  
| **UI Customization** | High | Moderate |  
| **Code Generation** | Yes | Yes |  
| **Performance** | Faster | Slower with large data |  
| **Access Control** | Built-in | Requires customization |  

📌 **Choose Avo** if you need a sleek UI and out-of-the-box features.  
📌 **Choose Administrate** if you prefer open-source, lightweight solutions.

---

#### **Conclusion**
Admin dashboards **simplify data management** in Rails applications.

💡 **Recap:**  
✔️ Avo provides a **modern, premium** experience with great customization.  
✔️ Administrate is **open-source** and offers a simple setup.  
✔️ Both allow you to **quickly build CRUD interfaces**.

🚀 **Next Steps:** Try integrating **Pundit or CanCanCan** for role-based access control!  
