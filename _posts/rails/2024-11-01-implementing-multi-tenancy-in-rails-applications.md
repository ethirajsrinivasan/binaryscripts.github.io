---
layout: post
title: Implementing Multi-Tenancy in Rails Applications for Scalable SaaS
subtitle: A deep dive into multi-tenant architectures in Ruby on Rails and their practical implementation
categories: Rails
tags: ["Rails", "Multi-Tenancy", "SaaS", "PostgreSQL", "Software Architecture"]
excerpt: Learn how to implement multi-tenancy in Rails applications using schema-based and row-based strategies for scalable SaaS platforms.
---

#### **Introduction**
Multi-tenancy is a crucial architectural pattern for SaaS applications where multiple users or organizations (tenants) share the same Rails application while maintaining data isolation. Implementing it correctly ensures **scalability, security, and maintainability**.

This guide explores **schema-based** and **row-based multi-tenancy** in Rails, their trade-offs, and best practices.

---

#### **Understanding Multi-Tenancy in Rails**
Multi-tenancy allows a single application to serve multiple tenants while ensuring data segregation. There are two primary approaches:

1. **Schema-Based Multi-Tenancy** – Each tenant has a separate schema in the database.
2. **Row-Based Multi-Tenancy** – A single database is shared, and tenant data is separated by a `tenant_id` column.

| Approach | Pros | Cons |
|----------|------|------|
| **Schema-Based** | Strong data isolation, better performance for large tenants | Complex migrations, requires PostgreSQL |
| **Row-Based** | Easier maintenance, works with all databases | Risk of tenant data leaks, performance issues at scale |

---

#### **Schema-Based Multi-Tenancy with PostgreSQL**
This approach requires PostgreSQL since it supports **schema switching** dynamically.

##### **Step 1: Install the Apartment Gem**
Apartment is a popular gem for managing multi-tenant schemas in Rails.

```ruby
gem 'apartment'
```
Then, install the gem:

```sh
bundle install
```

##### **Step 2: Configure Apartment**
Create an initializer: `config/initializers/apartment.rb`

```ruby
Apartment.configure do |config|
config.excluded_models = %w[User]
config.tenant_names = -> { Tenant.pluck(:subdomain) }
end
```

This ensures shared models like `User` remain outside the tenant-specific schemas.

##### **Step 3: Switching Tenant Schemas Dynamically**
Modify `application_controller.rb`:

```ruby
class ApplicationController < ActionController::Base
before_action :switch_tenant

def switch_tenant
subdomain = request.subdomain
Apartment::Tenant.switch!(subdomain) if Tenant.exists?(subdomain: subdomain)
end
end
```

##### **Step 4: Creating a New Tenant**
When a new tenant signs up, create a schema dynamically:

```ruby
Tenant.create!(subdomain: "customer1")
Apartment::Tenant.create("customer1")
```

---

#### **Row-Based Multi-Tenancy: Shared Database Approach**
Row-based multi-tenancy stores all data in a single database but differentiates records using a `tenant_id` column.

##### **Step 1: Add a `tenant_id` Column**
Run a migration:

```sh
rails generate migration AddTenantIdToUsers tenant_id:integer
rails db:migrate
```

##### **Step 2: Use `acts_as_tenant` for Scoping**
Install the `acts_as_tenant` gem:

```ruby
gem 'acts_as_tenant'
```

Run:

```sh
bundle install
```

Modify the model:

```ruby
class User < ApplicationRecord
acts_as_tenant(:tenant)
end
```

##### **Step 3: Automatically Scope Queries**
Modify `application_controller.rb`:

```ruby
class ApplicationController < ActionController::Base
set_current_tenant_through_filter
before_action :set_tenant

def set_tenant
current_tenant = Tenant.find_by(subdomain: request.subdomain)
set_current_tenant(current_tenant)
end
end
```

Now, every query automatically filters by the `tenant_id`.

---

#### **Performance Considerations**
- **Schema-based multi-tenancy** is better for large SaaS applications but requires PostgreSQL.
- **Row-based multi-tenancy** is easier to manage but may have **query performance** issues at scale.
- Use **database indexing** to optimize queries in row-based multi-tenancy.
- Implement **caching mechanisms** for frequent queries to improve performance.

---

#### **Security Best Practices**
- **Ensure tenant isolation**: Prevent cross-tenant data access using row-level security.
- **Use database-level constraints**: Enforce foreign key constraints to maintain data integrity.
- **Validate subdomains**: In schema-based multi-tenancy, validate tenant subdomains before switching schemas.

---

#### **Testing Multi-Tenancy in Rails**
For **schema-based multi-tenancy**:

```ruby
Apartment::Tenant.switch!('customer1')
puts User.all # Returns users only from customer1 schema
```

For **row-based multi-tenancy**:

```ruby
Tenant.current = Tenant.find_by(subdomain: 'customer1')
puts User.all # Automatically filters by tenant_id
```

---

#### **Conclusion**
Multi-tenancy is a **critical design choice** for SaaS applications.
- Use **schema-based multi-tenancy** if you need strong **data isolation**.
- Use **row-based multi-tenancy** if you prefer **simpler maintenance** and database compatibility.

By leveraging **Apartment** for schema-based and **acts_as_tenant** for row-based multi-tenancy, you can **scale your Rails applications effectively**. 🚀
