# Open Closed Principle

It states that any class should be open for extension but closed for modification.

## Example (Not follwing OCP)

```java
       public class VoilateOCP {

        public void acceptPayment(String payment,double amount){
        if (payment.equals("Paypal")){
            System.out.println("Payment :"+amount +" done through "+payment);
        } else if (payment.equals("Credit Card")) {
            System.out.println("Payment: "+amount+" done through "+payment);
        }
        else {
            throw new RuntimeException("Cannot be interpreted");
        }
    }

    public static void main(String[] args) {
        VoilateOCP voilateOCP =new VoilateOCP();
        voilateOCP.acceptPayment("Paypal",2000);
        voilateOCP.acceptPayment("Credit Card",1000);
    }

} 

```
Now suppose if we want to add new payment method Crypto we need to modify the existing code

```java
        if (payment.equals("Paypal")){
            System.out.println("Payment :"+amount +" done through "+payment);
        } else if (payment.equals("Credit Card")) {
            System.out.println("Payment: "+amount+" done through "+payment);
        }
        else if (payment.equals("Crypto")) {
            System.out.println("Payment: "+amount+" done through "+payment);
        }
        else {
            throw new RuntimeException("Cannot be interpreted");
        }

```
**Problem**: When adding new payment method we need to add it to existing code which might break our existing code.

> Note: Here in each payment we are just printing but in real case there can be huge logic.

- `Open for extension` means we can add new payment methods by creating new classes that implement AcceptPayment..
- `Closed for modification` means we don’t need to change the ProcessPayment class when adding new payment methods."

## Example using OCP

**1**. Use Interface
 ```java
public interface AcceptPayment {
    boolean supports(String paymentType);
    void processPayment(String paymentType,double amount);
}

 ```
 **2** Add implementation class which implements `AcceptPayment`

 ```java
public class CreditCardPayment implements AcceptPayment{
    @Override
    public boolean supports(String paymentType) {
        return "Credit Card".equals(paymentType);
    }

    @Override
    public void processPayment(String paymentType,double amount) {
        System.out.println("Payment: "+amount+" done by "+paymentType);
    }
}

 ```

 ```java
public class PaypalPayment implements AcceptPayment{
    @Override
    public boolean supports(String paymentType) {
        return "Paypal".equals(paymentType);
    }

    @Override
    public void processPayment(String paymentType, double amount) {
        System.out.println("Payment: "+amount+" done by "+paymentType);
    }
}


 ```
**3**
```java
public class ProcessPayment {

    private final List<AcceptPayment> paymentProcessors;

    public ProcessPayment(List<AcceptPayment> paymentProcessors) {
        this.paymentProcessors = paymentProcessors;
    }

    public void processPayment(String paymentType, double amount) {
        AcceptPayment processor = paymentProcessors.stream()
                .filter(p -> p.supports(paymentType))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("Unsupported payment type: " + paymentType));

        processor.processPayment(paymentType,amount);
    }

    public static void main(String[] args) {
        ProcessPayment p =new ProcessPayment(List.of(new PaypalPayment(),new CreditCardPayment()));
        p.processPayment("Paypal",2000);
        p.processPayment("Credit Card",6000);
    }
}
```

**4** Now if we want new payment method we need just an implementation class.

```java
    public class CryptoPayment implements AcceptPayment{
    @Override
    public boolean supports(String paymentType) {
        return "Crypto".equals(paymentType);
    }

    @Override
    public void processPayment(String paymentType, double amount) {
        System.out.println("Payment: "+amount+" done by "+paymentType);
    }
}
```

and adding it to our logic
```java
  ProcessPayment p =new ProcessPayment(List.of(new PaypalPayment(),new CreditCardPayment(),new CryptoPayment()));
        p.processPayment("Paypal",2000);
        p.processPayment("Credit Card",6000);
        p.processPayment("Crypto",100);
```


