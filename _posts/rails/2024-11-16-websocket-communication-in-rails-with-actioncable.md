---
layout: "post"
title: "WebSocket Communication in Rails with ActionCable"
subtitle: "Learn how to implement real-time WebSocket communication in Ruby on Rails using ActionCable."
categories: Rails
tags: ["Rails", "WebSockets", "ActionCable", "Real-time", "Broadcasting", "Turbo Streams"]
excerpt: "A deep dive into WebSocket communication in Rails using ActionCable, covering setup, channels, broadcasting, and real-world applications."
---



Real-time communication is crucial for modern web applications, whether for **chat apps, live notifications, collaborative editing, or stock market updates**. In Rails, **ActionCable** provides a built-in solution to integrate **WebSockets** seamlessly.

In this guide, we’ll explore:

- **What is ActionCable?**
- **Setting up WebSockets in Rails**
- **Creating and using channels**
- **Broadcasting messages to clients**
- **Enhancing real-time updates with Turbo Streams**

---

## **1. What is ActionCable?**

ActionCable is **Rails' built-in WebSocket framework** that allows **real-time, bidirectional communication** between the server and clients.

### **Key Features:**
- **Integrates WebSockets with Rails** seamlessly.
- **Works with ActiveRecord** for real-time data updates.
- **Supports authentication and authorization** for secure communication.

---

## **2. Setting Up ActionCable in Rails**

To use ActionCable, ensure you have WebSockets enabled in your Rails application.

### **Step 1: Enable ActionCable in `config/cable.yml`**
```yml
development:
adapter: redis
url: redis://localhost:6379/1

production:
adapter: redis
url: redis://localhost:6379/1
channel_prefix: myapp_production
```

**Note:** ActionCable **requires Redis** in production.

### **Step 2: Mount ActionCable in `routes.rb`**
```ruby
Rails.application.routes.draw do
mount ActionCable.server => "/cable"
end
```

---

## **3. Creating an ActionCable Channel**

ActionCable **channels** handle real-time communication between the client and server.

### **Generate a Channel**
Run:
```sh
rails generate channel Chat
```

This creates:

- `app/channels/chat_channel.rb`
- `app/javascript/channels/chat_channel.js`

### **Define Subscription Logic in `chat_channel.rb`**
```ruby
class ChatChannel < ApplicationCable::Channel
def subscribed
stream_from "chat_channel"
end

def unsubscribed
stop_all_streams
end

def speak(data)
Message.create!(content: data["message"], user_id: current_user.id)
ActionCable.server.broadcast("chat_channel", { message: data["message"] })
end
end
```

Here, we:
- **Subscribe clients** to `chat_channel`.
- **Broadcast messages** when a user sends a message.

### **Connect to the Channel on the Client Side**
Edit `app/javascript/channels/chat_channel.js`:
```javascript
import consumer from "./consumer";

consumer.subscriptions.create("ChatChannel", {
connected() {
console.log("Connected to ChatChannel");
},

received(data) {
document.querySelector("#messages").insertAdjacentHTML("beforeend", `<p>${data.message}</p>`);
},

sendMessage(message) {
this.perform("speak", { message });
}
});
```

**How It Works:**
- When a user **sends a message**, it **triggers** the `speak` method in `ChatChannel`.
- The server **broadcasts** the message to all subscribers.
- The client **updates the UI in real-time**.

---

## **4. Broadcasting Messages with ActionCable**

Instead of manually broadcasting in `speak`, use **ActiveRecord callbacks**.

### **Update `message.rb` for Automatic Broadcasting**
```ruby
class Message < ApplicationRecord
belongs_to :user
after_create_commit { broadcast_message }

private

def broadcast_message
ActionCable.server.broadcast("chat_channel", { message: self.content })
end
end
```

Now, **every new message automatically broadcasts** to the chat channel.

---

## **5. Enhancing Real-Time Updates with Turbo Streams**

If using **Hotwire/Turbo Streams**, broadcasting becomes even easier.

### **Update `message.rb` to Use Turbo Streams**
```ruby
class Message < ApplicationRecord
belongs_to :user
broadcasts_to :chat_room
end
```

### **Update `messages/index.html.erb`**
```erb
<%= turbo_stream_from "chat_room" %>

<div id="messages">
  <%= render @messages %>
</div>
```

With this setup:
- **New messages auto-update the UI without JavaScript.**
- **Leverages Rails’ Turbo Streams for WebSocket updates.**

---

## **6. Securing ActionCable Connections**

Since **WebSockets remain open**, securing connections is essential.

### **Authenticate Users in `application_cable/connection.rb`**
```ruby
module ApplicationCable
class Connection < ActionCable::Connection::Base
identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private

    def find_verified_user
      if (user = User.find_by(id: cookies.signed[:user_id]))
        user
      else
        reject_unauthorized_connection
      end
    end
end
end
```

Now, **only authenticated users can connect**.

---

## **7. Deploying ActionCable in Production**

In production, ActionCable **must use Redis**. Update:

### **`config/environments/production.rb`**
```ruby
config.action_cable.url = "wss://yourdomain.com/cable"
config.action_cable.allowed_request_origins = ["https://yourdomain.com"]
```

### **Use a Dedicated Redis Instance**
If using **Heroku**, add Redis:
```sh
heroku addons:create heroku-redis:hobby-dev
```

---

## **Conclusion**

WebSocket communication **enhances real-time features** in Rails applications. Using **ActionCable**, we can:
- **Establish WebSocket connections**
- **Handle broadcasting efficiently**
- **Enhance updates with Turbo Streams**
- **Secure and deploy WebSockets in production**

By following these steps, you can **build scalable, real-time Rails applications** with WebSockets.

*Have you used ActionCable in a project? Share your experience in the comments!*
