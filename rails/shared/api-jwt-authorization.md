# Setting Up API Authentication in a Rails 8 API-Only Application

Rails 8 introduces a built-in authentication system that can be adapted for API-only applications. This guide walks you through setting up token-based authentication for a Rails 8 API application.

## 1. Create a New Rails API Application

Start by generating a new Rails application in API mode:

```bash
rails new my_api_app --api
cd my_api_app
```

## 2. Run the Authentication Generator

Rails 8 provides an authentication generator to set up the basics:

```bash
bin/rails generate authentication
```

This creates necessary files, including models (`User`, `Session`), controllers (`SessionsController`, `PasswordsController`), and concerns (`Authentication`). Since the default setup is designed for HTML applications, we need to adapt it for API use.

## 3. Modify Controllers for API Responses

Update the generated `SessionsController` to handle JSON responses instead of HTML:

```ruby
class SessionsController < ApplicationController
  include Authentication

  def create
    if user = User.authenticate(session_params[:email_address], session_params[:password])
      session = user.sessions.create!
      render json: { token: session.token }, status: :created
    else
      render json: { error: "Invalid email or password" }, status: :unauthorized
    end
  end

  def destroy
    current_session.destroy
    head :no_content
  end

  private

  def session_params
    params.require(:session).permit(:email_address, :password)
  end
end
```

Upon successful authentication, a session token is generated and returned in the JSON response. Clients should include this token in the `Authorization` header for subsequent requests.

## 4. Implement Token-Based Authentication

Update the `Authentication` concern to use token-based authentication:

```ruby
module Authentication
  extend ActiveSupport::Concern

  included do
    before_action :require_authentication
  end

  private

  def require_authentication
    authenticate_with_http_token do |token, _options|
      Current.session = Session.find_by(token: token)
    end
    render json: { error: "Unauthorized" }, status: :unauthorized unless Current.session
  end
end
```

This extracts the token from the `Authorization` header and verifies its validity.

## 5. Update Routes

Define routes for session management:

```ruby
Rails.application.routes.draw do
  resources :sessions, only: [:create, :destroy]
end
```

## 6. Secure API Endpoints

Include the `Authentication` concern in controllers that require authentication:

```ruby
class ProtectedResourcesController < ApplicationController
  include Authentication

  def index
    # Your code here
  end
end
```

This ensures that all actions within this controller require authentication.

## 7. Test the Authentication Flow

Use `curl` or Postman to test your API:

### Sign In

```bash
curl -X POST http://localhost:3000/sessions \
-H "Content-Type: application/json" \
-d '{"session": {"email_address": "user@example.com", "password": "password"}}'
```

On success, you will receive a JSON response containing the session token.

### Access a Protected Resource

```bash
curl http://localhost:3000/protected_resources \
-H "Authorization: Bearer your_session_token"
```

Replace `your_session_token` with the token obtained from the sign-in response.

## Conclusion

By following these steps, you can implement a token-based authentication system in a Rails 8 API application. This setup ensures secure user authentication while keeping your API stateless.

---

_Reference: [Original article by Alejandro Chacón](https://a-chacon.com/en/on%20rails/2024/10/16/poc-using-rails-8-auth-system-in-api-only.html)._
