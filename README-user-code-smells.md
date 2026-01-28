# Code Smells – User Class

---

## 📌 0. What is this task about?

In this task I must:

1. Create a **new README** in the repository.
2. Copy the **original code** of the `User` class.
3. **Identify the existing code smells**.
4. **Explain why they are dangerous**.
5. **Refactor the code** to remove those code smells.
6. Make a **commit** and then open a **Pull Request**.

This file documents the complete analysis and refactor of the `User` class.

---

## 🧩 1. Original code

```java
public class User {
    private String name;
    private String address;
    private String phone;
    private String email;
    private int loyaltyPoints;
    private double accountBalance;
    private List<String> orders;
    private List<String> coupons;

    public void updateInfo(String name, String address, String phone, String email) {
        this.name = name;
        this.address = address;
        this.phone = phone;
        this.email = email;
    }

    public double calculateDiscount(int loyaltyPoints, double accountBalance) {
        double discount = 0.0;
        if (loyaltyPoints > 100) {
            discount = accountBalance * 0.1;
        } else if (loyaltyPoints > 200) {
            discount = accountBalance * 0.2;
        } else {
            discount = accountBalance * 0.05;
        }
        return discount;
    }

    public void printOrders() {
        for (String order : orders) {
            System.out.println("Order: " + order);
        }
    }

    public void applyCoupons(List<String> coupons) {
        for (String coupon : coupons) {
            System.out.println("Applying coupon: " + coupon);
        }
    }

    public void deprecatedMethod() {
        // This method is no longer used
    }
}
```

---

## 🚨 2. Identified Code Smells

### 2.1 Class with too many responsibilities

The `User` class is responsible for:
- Storing user data
- Calculating discounts
- Printing information to the console
- Applying coupons
- Containing deprecated code

**Why this is dangerous**
- It violates the **Single Responsibility Principle (SRP)**.
- The class becomes harder to understand and maintain.
- Any change in one responsibility increases the risk of bugs in others.

---

### 2.2 Incorrect conditional logic

```java
if (loyaltyPoints > 100) {
    discount = accountBalance * 0.1;
} else if (loyaltyPoints > 200) {
    discount = accountBalance * 0.2;
}
```

**Problem**
- The condition `loyaltyPoints > 200` is never reached because values greater than 200 also satisfy `> 100`.

**Why this is dangerous**
- Produces incorrect business results.
- Leads to misleading and error-prone code.

---

### 2.3 Unnecessary parameters

The `calculateDiscount` method receives `loyaltyPoints` and `accountBalance` as parameters even though these values already belong to the class.

**Why this is dangerous**
- Breaks encapsulation.
- Can cause inconsistencies between parameters and internal state.
- Makes the method harder to use correctly.

---

### 2.4 Tight coupling to the console

Methods like `printOrders` and `applyCoupons` directly use `System.out.println`.

**Why this is dangerous**
- Mixes business logic with presentation logic.
- Makes the class difficult to reuse in other contexts (APIs, UI, tests).
- Complicates unit testing.

---

### 2.5 Dead code

```java
public void deprecatedMethod() {
    // This method is no longer used
}
```

**Why this is dangerous**
- Adds unnecessary noise to the codebase.
- Confuses developers about what should or should not be used.
- Provides no real value.

---

### 2.6 Primitive obsession

```java
private List<String> orders;
private List<String> coupons;
```

Orders and coupons are represented only as `String` values.

**Why this is dangerous**
- The code is not expressive enough.
- It becomes difficult to extend when more data is needed.
- Indicates missing domain models.

---

## 🔧 3. Proposed Refactor

### 3.1 Goals

- Fix incorrect business logic.
- Improve encapsulation.
- Reduce responsibilities of the `User` class.
- Prepare the codebase for future scalability.

---

### 3.2 Refactored code

```java
public class User {
    private String name;
    private String address;
    private String phone;
    private String email;
    private int loyaltyPoints;
    private double accountBalance;

    public void updateInfo(String name, String address, String phone, String email) {
        this.name = name;
        this.address = address;
        this.phone = phone;
        this.email = email;
    }

    public void setLoyaltyPoints(int loyaltyPoints) {
        if (loyaltyPoints < 0) {
            throw new IllegalArgumentException("Loyalty points cannot be negative");
        }
        this.loyaltyPoints = loyaltyPoints;
    }

    public void setAccountBalance(double accountBalance) {
        if (accountBalance < 0) {
            throw new IllegalArgumentException("Account balance cannot be negative");
        }
        this.accountBalance = accountBalance;
    }

    /**
     * Calculates the discount based on the internal state of the user.
     *
     * > 200 points : 20%
     * > 100 points : 10%
     * otherwise    : 5%
     */
    public double calculateDiscount() {
        if (loyaltyPoints > 200) {
            return accountBalance * 0.20;
        }
        if (loyaltyPoints > 100) {
            return accountBalance * 0.10;
        }
        return accountBalance * 0.05;
    }
}
```

---

### 📌 Note about removed responsibilities

In this refactor, the `User` class no longer handles:
- Printing orders to the console
- Applying coupons
- Managing orders and coupons as `String` lists

These responsibilities should be moved to other classes or layers of the system
(for example, `OrderService`, `CouponService`, or the presentation layer).

This keeps the `User` class focused on representing the user and its core business logic,
fully respecting the **Single Responsibility Principle**.

---

## 🎯 4. Conclusion

In this exercise:

- Multiple **code smells** were identified in the original `User` class.
- Each smell was explained along with the risks it introduces.
- A refactor was proposed that:
  - Fixes incorrect logic
  - Improves encapsulation
  - Reduces unnecessary responsibilities
  - Makes the code easier to maintain and extend

As a result, the `User` class is cleaner, easier to understand, and better prepared
to evolve within a larger system.
