# SOLID – Order Refactor

## 1. Goal of this exercise

In this small exercise I had to refactor the `Order` code to apply some SOLID principles.  
The main idea was to clean up the responsibilities inside the class and make the code easier to extend and maintain.

The task asked me to focus mainly on:

* **SRP (Single Responsibility Principle)**
* **OCP (Open/Closed Principle)**
* **DIP (Dependency Inversion Principle)**

---

## 2. Short recap of SOLID

* **S – Single Responsibility Principle (SRP)**  
  Each class or module should have one clear responsibility and one main reason to change.

* **O – Open/Closed Principle (OCP)**  
  The code should be open for extension but closed for modification. We should be able to add new behavior without editing existing classes.

* **L – Liskov Substitution Principle (LSP)**  
  Subclasses should be usable wherever the base class is expected, without breaking the program.

* **I – Interface Segregation Principle (ISP)**  
  It is better to have several small, specific interfaces than a single large one that forces classes to implement methods they do not need.

* **D – Dependency Inversion Principle (DIP)**  
  High-level modules should not depend directly on low-level details. Both should depend on abstractions (in Ruby this usually means depending on an object that responds to a certain method instead of a specific concrete class).

---

## 3. Problems in the original code

The original `Order` class looked like this (simplified here):

```ruby
class Order
  def initialize(items)
    @items = items
  end

  def calculate_total
    total = 0
    @items.each do |item|
      total += item.price
    end
    total
  end

  def send_confirmation_email
    # Lógica para enviar un correo electrónico de confirmación
    puts "Email enviado a customer@example.com"
  end

  def print_order
    @items.each do |item|
      puts "Item: #{item.name} - Price: #{item.price}"
    end
  end
end

class Item
  attr_accessor :name, :price

  def initialize(name, price)
    @name = name
    @price = price
  end
end
```

Main issues I identified:

* **SRP violation**:  
  The `Order` class was doing too many things at the same time:
  * Keeping the list of items.
  * Calculating the total.
  * Sending a confirmation email.
  * Printing the order.

* **OCP violation**:  
  If I wanted to add discounts, taxes or different pricing rules, I would have to change the `calculate_total` method inside `Order` directly.

* **DIP violation**:  
  The email logic was hard-coded inside `Order` using `puts` and a fixed email address.  
  The class depended on a very specific way of “sending” the email.

---

## 4. Refactor decisions

To solve these problems I made the following changes:

* I kept `Order` focused on representing an order and its items.
* I moved the printing logic to a separate class: `OrderPrinter`.
* I moved the email responsibility to a separate class: `EmailService`.
* I introduced different pricing strategies (`StandardPricing` and `DiscountPricing`) to calculate the total without modifying the `Order` class.

This way:

* The `Order` class has a more clear responsibility (SRP).
* New pricing rules can be added by creating new strategy classes (OCP).
* The `Order` class no longer knows how the email is sent; it just uses an external service for that (DIP, via dependency on another object instead of doing it itself).

---

## 5. Refactored code

Below is the refactored version of the code that applies these ideas:

```ruby
class Order
  attr_reader :items

  def initialize(items)
    @items = items
  end

  # The total is calculated by a pricing strategy
  def calculate_total(pricing_strategy)
    pricing_strategy.calculate_total(@items)
  end
end

class StandardPricing
  # Simple sum of item prices
  def calculate_total(items)
    items.sum(&:price)
  end
end

class DiscountPricing
  def initialize(discount)
    @discount = discount
  end

  # Applies a discount (for example 0.10 for 10%)
  def calculate_total(items)
    total = items.sum(&:price)
    total - (total * @discount)
  end
end

class EmailService
  # This class is responsible for sending confirmation emails
  def send_confirmation(email)
    # Here we would have the real email logic
    puts "Email sent to #{email}"
  end
end

class OrderPrinter
  # This class is responsible for printing the order
  def print(order)
    order.items.each do |item|
      puts "Item: #{item.name} - Price: #{item.price}"
    end
  end
end

class Item
  attr_accessor :name, :price

  def initialize(name, price)
    @name = name
    @price = price
  end
end

# Example of usage
items = [Item.new("Laptop", 1000), Item.new("Mouse", 50)]
order = Order.new(items)

# Using the standard pricing strategy
total = order.calculate_total(StandardPricing.new)
puts "Total: #{total}"

# Using a discount pricing strategy (10% discount)
discount_total = order.calculate_total(DiscountPricing.new(0.10))
puts "Discount total (10%): #{discount_total}"

# Sending confirmation email
email_service = EmailService.new
email_service.send_confirmation("customer@example.com")

# Printing the order
order_printer = OrderPrinter.new
order_printer.print(order)
```

---

## 6. How this applies SOLID

* **SRP**  
  * `Order` only manages the list of items and delegates work to other classes.  
  * `OrderPrinter` only prints the order.  
  * `EmailService` only deals with sending confirmation emails.

* **OCP**  
  * New pricing rules can be implemented by adding new classes that respond to `calculate_total(items)` (for example, a class with taxes or special promotions).  
  * The `Order` class does not need to be modified when a new pricing strategy is created.

* **DIP**  
  * The calculation of the total is delegated to an external strategy object passed into `calculate_total`.  
  * The email logic lives in `EmailService`, which can be replaced or extended without changing `Order`.

With this refactor the code is easier to read, test and extend, and it follows the main SOLID principles requested in the task.
