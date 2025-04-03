---
layout: post  
title: Implementing Domain-Driven Design in Ruby on Rails  
subtitle: Learn how to apply Domain-Driven Design (DDD) principles in Rails applications for better scalability and maintainability  
categories: RubyOnRails  
tags: ["Rails", "DDD", "Software Architecture", "Microservices", "Event-Driven Design"]  
excerpt: A deep dive into implementing Domain-Driven Design (DDD) in Ruby on Rails, covering bounded contexts, aggregates, repositories, and event-driven design for scalable applications.  
---


Modern Rails applications require **scalable architectures** to manage complexity, and **Domain-Driven Design (DDD)** offers a structured approach. DDD helps in **aligning business logic with the codebase**, improving **modularity, maintainability, and scalability**.

In this guide, we will explore:

✔️ **Key principles of DDD in Rails**  
✔️ **How to structure a Rails project using bounded contexts**  
✔️ **Implementing aggregates, repositories, and domain services**  
✔️ **Using event-driven design with Rails**

---

#### **1. Understanding Domain-Driven Design (DDD)**
DDD focuses on designing software **around the business domain**. Key principles include:

✔️ **Bounded Contexts** – Dividing the application into isolated business areas  
✔️ **Aggregates** – Grouping domain objects for transactional consistency  
✔️ **Repositories** – Managing persistence logic separately  
✔️ **Domain Services** – Handling business logic that doesn’t fit in a single entity  
✔️ **Event-Driven Design** – Using domain events to decouple components

---

#### **2. Structuring a Rails Application for DDD**
Traditional Rails projects follow **MVC**, but for DDD, we need a modular structure.

##### **Recommended Folder Structure**
```sh  
app/  
├── domains/  
│   ├── orders/  
│   │   ├── models/  
│   │   ├── repositories/  
│   │   ├── services/  
│   │   ├── events/  
│   ├── payments/  
│   │   ├── models/  
│   │   ├── repositories/  
│   │   ├── services/  
│   │   ├── events/  
├── infrastructure/  
├── interfaces/  
├── controllers/  
├── views/  
```

Each **domain** (e.g., `orders`, `payments`) has its own models, repositories, services, and events.

---

#### **3. Implementing Bounded Contexts in Rails**
A **bounded context** defines a boundary within the application where a **particular domain model is valid**.

##### **Example: Orders and Payments Contexts**
- The `Orders` context manages order processing and tracking.
- The `Payments` context handles transactions and invoices.
- These **should not directly share models** but communicate via **domain events**.

**Example Model (`app/domains/orders/models/order.rb`)**  
```rb  
module Orders  
class Order < ApplicationRecord  
has_many :items, class_name: "Orders::Item"  
end  
end  
```

**Example Model (`app/domains/payments/models/payment.rb`)**  
```rb  
module Payments  
class Payment < ApplicationRecord  
belongs_to :order, class_name: "Orders::Order"  
end  
end  
```

This keeps **Payments** and **Orders** separate while linking them via `order_id`.

---

#### **4. Implementing Aggregates and Entities**
An **aggregate** is a group of domain objects that should be treated as a single unit.

**Example: Order as an Aggregate Root**  
```rb  
module Orders  
class Order < ApplicationRecord  
has_many :items, class_name: "Orders::Item", dependent: :destroy  
end  
end  
```

The `Order` is the **aggregate root** and ensures that changes to `Items` are consistent.

---

#### **5. Using Repositories for Data Access**
Repositories **abstract database interactions**, preventing direct queries in controllers or models.

##### **Order Repository (`app/domains/orders/repositories/order_repository.rb`)**
```rb  
module Orders  
class OrderRepository  
def self.find_by_id(id)  
Order.includes(:items).find(id)  
end

    def self.save(order)  
      order.save!  
    end  
end  
end  
```

Instead of `Order.find(id)`, use:  
```rb  
Orders::OrderRepository.find_by_id(1)  
```

This **decouples persistence logic** from the business domain.

---

#### **6. Implementing Domain Services**
Some logic doesn’t fit in a model. **Domain services** handle such cases.

##### **Example: Payment Processing Service**
```rb  
module Payments  
class PaymentService  
def self.process(order, payment_method)  
payment = Payment.create!(order: order, method: payment_method, status: "pending")  
PaymentGateway.charge(payment)  
payment.update!(status: "completed")  
end  
end  
end  
```

Now, controllers only call:  
```rb  
Payments::PaymentService.process(order, "credit_card")  
```

---

#### **7. Using Domain Events for Decoupling**
Domain events allow different contexts to communicate asynchronously.

##### **Example: OrderPlaced Event**
```rb  
module Orders  
class OrderPlacedEvent  
attr_reader :order_id

    def initialize(order_id)  
      @order_id = order_id  
    end  
end  
end  
```

##### **Publishing an Event**
```rb  
order = Orders::Order.create!(...)  
event = Orders::OrderPlacedEvent.new(order.id)  
EventBus.publish(event)  
```

##### **Subscribing to the Event in Payments**
```rb  
module Payments  
class OrderPlacedListener  
def self.handle(event)  
order = Orders::Order.find(event.order_id)  
PaymentService.process(order, "credit_card")  
end  
end  
end  
```

This allows `Payments` to **react to `Orders` without direct coupling**.

---

#### **8. Implementing an Event Bus**
A simple event bus **handles event distribution**.

##### **Example Event Bus (`app/infrastructure/event_bus.rb`)**
```rb  
class EventBus  
@subscribers = {}

def self.subscribe(event_class, handler)  
@subscribers[event_class] ||= []  
@subscribers[event_class] << handler  
end

def self.publish(event)  
(@subscribers[event.class] || []).each do |handler|  
handler.handle(event)  
end  
end  
end  
```

This **decouples components** while maintaining **business logic flow**.

---

#### **Conclusion**
By **applying Domain-Driven Design (DDD) principles**, Rails applications become **scalable, maintainable, and domain-focused**.

🚀 **Key Takeaways:**  
✔️ **Use bounded contexts** to separate business domains  
✔️ **Implement aggregates** to maintain consistency  
✔️ **Use repositories** to manage persistence  
✔️ **Leverage domain events** for communication between services

🔗 **Next Steps:** Explore **event sourcing and CQRS** for further scalability!  
