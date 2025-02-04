---
layout: post
title: Advanced Custom Validations in Rails Models
subtitle: Create powerful and reusable custom validations to enhance data integrity in your Rails applications.
categories: Ruby on Rails
tags: ["Rails", "Custom Validations", "ActiveRecord", "Best Practices", "Data Integrity"]
excerpt: Learn how to build advanced custom validations in Ruby on Rails models to enforce complex business rules and ensure data integrity.
---

#### **Introduction**
ActiveRecord's built-in validations (`validates_presence_of`, `validates_uniqueness_of`, etc.) cover most use cases, but sometimes, you need **custom validations** to enforce complex business rules.

**Example Use Cases for Custom Validations:**  
✅ Ensuring email domains belong to a specific company  
✅ Validating complex password policies  
✅ Restricting booking times to business hours  
✅ Checking if a user’s input conflicts with existing records

In this guide, we’ll explore **advanced custom validations** in Rails, including:  
✔️ Writing reusable **validation methods**  
✔️ Creating **fully customized validator classes**  
✔️ Handling **asynchronous validations**

By the end, you'll be able to write **scalable, efficient, and reusable** validation logic in Rails 🚀

---

#### **1. Using `validate` for Custom Model-Level Validations**
The simplest way to add a custom validation is by using the `validate` method inside the model.

##### **Example: Ensuring a Username Is Alphanumeric**
```ruby
class User < ApplicationRecord
validate :username_must_be_alphanumeric

private

def username_must_be_alphanumeric
unless username =~ /\A[a-zA-Z0-9]+\z/
errors.add(:username, "can only contain letters and numbers")
end
end
end
```  
✅ **This approach is ideal for simple, one-off validations.**

---

#### **2. Creating a Reusable Custom Validator Class**
For **complex validations** that multiple models need, creating a separate validator class is best.

##### **Example: Ensuring Email Belongs to an Approved Domain**
```ruby
class EmailDomainValidator < ActiveModel::EachValidator
APPROVED_DOMAINS = ["example.com", "company.org"]

def validate_each(record, attribute, value)
domain = value.split("@").last
unless APPROVED_DOMAINS.include?(domain)
record.errors.add(attribute, "must belong to an approved domain: #{APPROVED_DOMAINS.join(', ')}")
end
end
end
```

Now, apply this validator to multiple models:

```ruby
class User < ApplicationRecord
validates :email, presence: true, email_domain: true
end

class Admin < ApplicationRecord
validates :email, email_domain: true
end
```  
✅ **This makes the validation reusable and keeps models cleaner.**

---

#### **3. Conditional Validations: Apply Rules Dynamically**
Sometimes, you need to **validate fields only under certain conditions**.

##### **Example: Enforcing Password Strength for New Users but Not Updates**
```ruby
class User < ApplicationRecord
validates :password, presence: true, length: { minimum: 8 }, if: :new_record?
validate :password_complexity, if: :password_present?

private

def password_present?
password.present?
end

def password_complexity
unless password.match?(/\A(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*])/)
errors.add(:password, "must include at least one uppercase letter, one digit, and one special character")
end
end
end
```  
✅ **This prevents unnecessary validation on updates.**

---

#### **4. Cross-Field Validations: Comparing Multiple Attributes**
When one field’s validity **depends on another**, you need cross-field validation.

##### **Example: Ensuring `end_date` Is After `start_date`**
```ruby
class Event < ApplicationRecord
validate :end_date_after_start_date

private

def end_date_after_start_date
if end_date.present? && start_date.present? && end_date <= start_date
errors.add(:end_date, "must be after the start date")
end
end
end
```  
✅ **This ensures logical consistency between fields.**

---

#### **5. Checking Uniqueness Across Multiple Fields**
Rails' `validates_uniqueness_of` only works for a **single attribute**. For more complex checks, use a custom validation.

##### **Example: Unique Combination of `category` and `name` in Products**
```ruby
class Product < ApplicationRecord
validate :unique_category_and_name

private

def unique_category_and_name
if Product.where(category: category, name: name).exists?
errors.add(:base, "A product with this name already exists in the category")
end
end
end
```  
✅ **This ensures category + name is unique.**

---

#### **6. Asynchronous Validations with ActiveJob**
If a validation requires **slow external API calls**, move it to a **background job** to avoid slowing down requests.

##### **Example: Validating Email Against an External API**
```ruby
class EmailVerificationJob < ApplicationJob
queue_as :default

def perform(user_id)
user = User.find(user_id)
unless ExternalEmailService.valid?(user.email)
user.errors.add(:email, "is invalid")
end
end
end
```

Trigger this job **after saving** instead of during validation:  
```ruby
class User < ApplicationRecord
after_create :verify_email_async

private

def verify_email_async
EmailVerificationJob.perform_later(id)
end
end
```  
✅ **This ensures non-blocking validation while keeping the user experience smooth.**

---

#### **7. Skipping Validations When Necessary**
In some cases, you **may want to skip validations**—for example, during bulk imports.

Use **`update_attribute`** (skips validations but triggers callbacks):  
```ruby
user.update_attribute(:email, "new@example.com")
```

Or use **`update_columns`** (skips both validations and callbacks):  
```ruby
user.update_columns(email: "new@example.com")
```

✅ **Use these methods carefully to avoid data inconsistencies.**

---

#### **Conclusion**
Custom validations in Rails **enhance data integrity and enforce business rules**. In this guide, we explored:  
✔️ **Basic custom validations with `validate`**  
✔️ **Reusable custom validators with `ActiveModel::EachValidator`**  
✔️ **Conditional validations and cross-field checks**  
✔️ **Handling uniqueness across multiple columns**  
✔️ **Running validations asynchronously with background jobs**

With these techniques, you can **build robust Rails applications** that ensure high-quality data integrity while maintaining **performance and scalability**.

💡 **What’s your favorite custom validation trick? Share in the comments below!** 🚀  
