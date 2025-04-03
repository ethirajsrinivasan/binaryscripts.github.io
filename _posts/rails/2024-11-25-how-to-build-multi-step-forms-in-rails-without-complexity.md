---
layout: post  
title: How to Build Multi-Step Forms in Rails Without Complexity  
subtitle: Create user-friendly multi-step forms in Rails without unnecessary complexity  
categories: RubyOnRails  
tags: ["Rails", "Multi-Step Forms", "Wizards", "Form Handling", "UX"]  
excerpt: Learn how to build simple and scalable multi-step forms in Rails, improving user experience and data handling while keeping the implementation clean and efficient.  
---


Multi-step forms (also known as **form wizards**) enhance **user experience** by breaking down long forms into smaller, manageable steps. This approach reduces **friction** and increases **completion rates**, especially for forms that collect a large amount of data.

In this guide, we will:

✔️ **Understand multi-step form architecture in Rails**  
✔️ **Implement step-by-step navigation using sessions or database storage**  
✔️ **Validate data at each step**  
✔️ **Improve UX with progress indicators**

---

#### **1. Choosing the Right Approach for Multi-Step Forms**
There are **two common approaches** to handling multi-step forms in Rails:

1️⃣ **Session-Based Approach** – Store form data in sessions until submission.  
2️⃣ **Database-Persisted Approach** – Save form progress to the database at each step.

🚀 **Use Sessions for simple forms** with temporary data.  
🚀 **Use Database Persistence when handling large or important data** (e.g., onboarding forms).

---

#### **2. Setting Up a Multi-Step Form in Rails**
We'll create a **user registration wizard** with **three steps**:

✔️ Step 1: Personal Information  
✔️ Step 2: Address Details  
✔️ Step 3: Account Confirmation

##### **Generating the Model and Controller**
```sh  
rails generate model User name:string email:string address:string city:string  
rails generate controller Users  
```

##### **Defining the Steps in the Controller**
```rb  
class UsersController < ApplicationController  
before_action :set_user, only: [:show, :edit, :update]

STEPS = %w[personal_info address confirmation].freeze

def new  
session[:user_data] ||= {}  
@user = User.new  
@step = 0  
end

def create  
step = params[:step].to_i  
session[:user_data].merge!(user_params)

    if step < STEPS.length - 1  
      redirect_to users_new_path(step: step + 1)  
    else  
      @user = User.new(session[:user_data])  
      if @user.save  
        session[:user_data] = nil  
        redirect_to @user, notice: "User successfully created!"  
      else  
        render "new"  
      end  
    end  
end

private

def set_user  
@user = User.find(params[:id])  
end

def user_params  
params.require(:user).permit(:name, :email, :address, :city)  
end  
end  
```

---

#### **3. Creating the Multi-Step Form Views**
##### **Step 1: Personal Information (app/views/users/_personal_info.html.erb)**
```erb  
<%= form_with model: @user, url: users_create_path(step: 0), local: true do |f| %>  
<%= f.label :name %>  
<%= f.text_field :name %>

<%= f.label :email %>  
<%= f.email_field :email %>

<%= f.submit "Next" %>  
<% end %>  
```

##### **Step 2: Address Details (app/views/users/_address.html.erb)**
```erb  
<%= form_with model: @user, url: users_create_path(step: 1), local: true do |f| %>  
<%= f.label :address %>  
<%= f.text_field :address %>

<%= f.label :city %>  
<%= f.text_field :city %>

<%= f.submit "Next" %>  
<% end %>  
```

##### **Step 3: Confirmation (app/views/users/_confirmation.html.erb)**
```erb  
<h3>Review Your Information</h3>
<p>Name: <%= session[:user_data]["name"] %></p>  
<p>Email: <%= session[:user_data]["email"] %></p>  
<p>Address: <%= session[:user_data]["address"] %>, <%= session[:user_data]["city"] %></p>  

<%= form_with model: @user, url: users_create_path(step: 2), local: true do |f| %>  
<%= f.submit "Confirm and Submit" %>  
<% end %>  
```

---

#### **4. Enhancing the User Experience**
##### **Adding a Progress Indicator**
Include a simple progress tracker in `application.html.erb`:

```erb
<ul class="progress-bar">  
  <% UsersController::STEPS.each_with_index do |step_name, index| %>  
    <li class="<%= 'active' if index <= @step %>">  
      <%= step_name.humanize %>  
    </li>  
  <% end %>  
</ul>  
```  

📌 **Why?** Users see their progress and are less likely to abandon the form.

---

#### **5. Handling Validations and Errors in Multi-Step Forms**
We need to ensure **each step validates only its fields**.

##### **Updating the User Model**
```rb  
class User < ApplicationRecord  
validates :name, :email, presence: true, if: -> { step == "personal_info" }  
validates :address, :city, presence: true, if: -> { step == "address" }

attr_accessor :step  
end  
```

##### **Displaying Errors in Views**
```erb  
<% if @user.errors.any? %>
  <div class="error-messages">  
    <h4>Errors:</h4>  
    <ul>  
      <% @user.errors.full_messages.each do |msg| %>  
        <li><%= msg %></li>  
      <% end %>  
    </ul>  
  </div>  
<% end %>  
```  

🚀 **Why?** Shows validation errors without disrupting the user experience.

---

#### **6. Using Database Persistence Instead of Sessions**
For **long forms or user onboarding**, store progress in the database:

##### **Adding a `step` Column to Users Table**
```sh  
rails generate migration AddStepToUsers step:integer  
rails db:migrate  
```

##### **Modifying Controller to Persist Data**
```rb  
@user.update(step: step) if @user.save  
```

🚀 **Why?** Users can **resume their form later** without losing data.

---

#### **Conclusion**
Multi-step forms **simplify complex data collection** in Rails applications.

🚀 **Key Takeaways:**  
✔️ Use **sessions for simple multi-step forms**.  
✔️ Use **database persistence for long forms or onboarding flows**.  
✔️ Implement **progress indicators** to improve UX.  
✔️ Validate data **step-by-step** to prevent user frustration.

