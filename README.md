# Test Task

## File and Folder Structure

```text
web-shop/
│
├── client/ (Frontend)
│   ├── public/
│   ├── src/
│   │   ├── components/ (Reusable UI components)
│   │   ├── pages/ (Page components, e.g., Home, ProductList, ProductDetail)
│   │   ├── services/ (API calls)
│   │   ├── utils/ (Utility functions)
│   │   └── app.js (Main application component)
│   └── package.json
│
├── server/ (Backend)
│   ├── src/
│   │   ├── controllers/ (Request handling logic)
│   │   ├── models/ (Database models)
│   │   ├── routes/ (API routes)
│   │   ├── services/ (Business logic, e.g., ProductService, OrderService)
│   │   └── app.js (Server setup and middleware)
│   ├── package.json
│   └── .env (Environment variables)
│
└── README.md (Project documentation)
```

## Separation of Business Logic
* **Frontend (client):** Focuses on presenting data to the user and capturing user input. It communicates with the backend through API calls, handled in the services/ directory.
* **Backend (server):** Handles business logic, database operations, and serves API requests. The services/ directory contains the core business logic, while controllers/ manage the request-response cycle.


## Explanation of Architecture/Design Choices

* **Scalability:** This structure allows for easy scaling of the application. New features like additional payment processors or shipping providers can be integrated into the services/ directory without affecting other parts of the application.
* **Maintainability:** Separating concerns makes the codebase easier to understand and maintain. For example, changing the UI does not require changes to the business logic.
* **Flexibility:** The architecture is designed to be framework-agnostic on the backend and can be adapted to different frontend frameworks with minimal changes.


## Payment Processing Providers Draft Implementation

For handling multiple payment processing services within an app, the Strategy Pattern is an excellent choice. This pattern enables an object to change its behavior when its internal state changes, appearing as if it changed its class. In the context of payment processing, it allows us to define a family of algorithms (payment methods), encapsulate each one, and make them interchangeable. The strategy pattern lets the payment method vary independently from clients that use it, facilitating the addition or removal of payment options without altering the main order processing logic.

The Strategy Pattern is chosen for its flexibility and adherence to the open/closed principle, one of the SOLID principles of object-oriented design. It allows the payment processing logic to be extended (by adding new payment methods) without modifying the existing codebase, thus avoiding potential bugs or the need for extensive testing after each modification. Each payment provider has its own class that implements the PaymentStrategy interface, ensuring that all payment strategies have a common method signature. The PaymentContext class acts as a facade to these strategies, allowing the client code to switch payment methods dynamically at runtime. This design makes it easy to add or remove payment options with minimal impact on the overall system.

### PaymentStrategy Interface

First, define a common interface for all payment strategies.
```javascript
class PaymentStrategy{
  pay(amount){
    throw new Error("Method 'pay()' must be implemented.");
  }
}
```

### Specific Payment Strategies

Here I implement concrete strategies for each payment provider.

```javascript
class StripePaymentStrategy extends PaymentStrategy{
  pay(amount){
    console.log(`Paying $${amount} using Stripe`);
    // Logic to integrate with Stripe's API
  }
}

class BraintreePaymentStrategy extends PaymentStrategy{
  pay(amount){
    console.log(`Paying $${amount} using Braintree`);
    // Logic to integrate with Braintree's API
  }
}

class PayPalPaymentStrategy extends PaymentStrategy{
  pay(amount){
    console.log(`Paying $${amount} using PayPal`);
    // Logic to integrate with PayPal's API
  }
}
```

### Context Class

Creating a context class to use the payment strategies.

```javascript
class PaymentContext{
  constructor(strategy){
    this.strategy = strategy;
  }

  setStrategy(strategy){
    this.strategy = strategy;
  }

  executePayment(amount){
    this.strategy.pay(amount);
  }
}
```

### Usage

```javascript
const paymentAmount = 100;

const stripePayment = new StripePaymentStrategy();
const braintreePayment = new BraintreePaymentStrategy();
const paypalPayment = new PayPalPaymentStrategy();

const paymentContext = new PaymentContext(stripePayment);
paymentContext.executePayment(paymentAmount);

// Switching to another payment provider
paymentContext.setStrategy(braintreePayment);
paymentContext.executePayment(paymentAmount);

// Adding a new payment option in the future would simply involve creating a new strategy class.
```
