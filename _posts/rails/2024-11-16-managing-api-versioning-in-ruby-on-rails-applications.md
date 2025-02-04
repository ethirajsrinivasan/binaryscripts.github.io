
Inside `app/controllers/api/v1/users_controller.rb`:  
```ruby
module Api
module V1
class UsersController < ApplicationController
def index
users = User.all
render json: users, status: :ok
end
end
end
end
```

Inside `app/controllers/api/v2/users_controller.rb`, we introduce an **extra field in the response**:  
```ruby
module Api
module V2
class UsersController < ApplicationController
def index
users = User.all
render json: users.map { |user| user.as_json.merge({ version: "v2" }) }, status: :ok
end
end
end
end
```

Now, `v2` includes **extra metadata**, while `v1` remains unchanged. 🚀

---

#### **4. Implementing Header-Based Versioning**
Another approach is **header-based versioning** using `Accept` headers.

##### **Step 1: Modify Routes**
Instead of separate namespaces, use a **single controller** and detect versions dynamically.

Modify `routes.rb`:  
```ruby
Rails.application.routes.draw do
scope :api do
resources :users, only: [:index, :show]
end
end
```

##### **Step 2: Implement Version Detection in ApplicationController**
Modify `application_controller.rb`:  
```ruby
class ApplicationController < ActionController::API
before_action :set_api_version

def set_api_version
version = request.headers["Accept"]&.match(/vnd.myapp.v(\d+)/)
@api_version = version ? version[1].to_i : 1
end
end
```

##### **Step 3: Modify Controller to Handle Versions**
Inside `users_controller.rb`:  
```ruby
class UsersController < ApplicationController
def index
users = User.all
response_data = if @api_version == 2
users.map { |user| user.as_json.merge({ version: "v2" }) }
else
users
end

    render json: response_data, status: :ok
end
end
```

Now, clients can request versions via headers:  
✔️ `Accept: application/vnd.myapp.v1+json` → Returns **v1**  
✔️ `Accept: application/vnd.myapp.v2+json` → Returns **v2**

---

#### **5. Deprecating Older API Versions**
Over time, older API versions should be **phased out**. Strategies include:  
✔️ **Setting an expiration date** and notifying clients via response headers.  
✔️ **Logging usage metrics** to track active versions.  
✔️ **Providing migration guides** for clients to upgrade smoothly.

Example: Notify clients of deprecation:  
```ruby
if @api_version == 1
response.set_header("Warning", "API v1 will be deprecated on 2025-06-30")
end
```

This ensures developers **migrate before breaking changes** occur.

---

#### **6. Testing and Maintaining Versioned APIs**
Use **RSpec** or **Minitest** to test multiple API versions:  
```ruby
describe "GET /api/v1/users" do
it "returns version 1 response" do
get "/api/v1/users"
expect(response.body).to include("users")
end
end
```

For header-based versioning:  
```ruby
describe "GET /users with API version header" do
it "returns v2 response when requested" do
get "/users", headers: { "Accept" => "application/vnd.myapp.v2+json" }
expect(response.body).to include("version")
end
end
```

---

#### **Conclusion**
API versioning is crucial for **maintaining stable, scalable Rails applications**. We explored:  
✔️ **Path-based and header-based versioning strategies**  
✔️ **Implementation of versioned routes, controllers, and responses**  
✔️ **Deprecating old versions while ensuring smooth client migration**  
✔️ **Testing versioned APIs for stability**

💡 **Next Steps:** Try integrating versioning into **GraphQL APIs** or use **API Gateway solutions** like **AWS API Gateway** for centralized version control. 🚀  
