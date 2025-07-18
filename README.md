# Paymentus Server Java SDK

## What is the Paymentus Server SDK?
The Paymentus Server SDK is a comprehensive Java-based toolkit that simplifies integration with the Paymentus Server-side APIs. Designed for backend services, it enables secure, streamlined access to Paymentus' powerful XOTP platform—including payments, user management, autopay, account inquiries, and more. The SDK abstracts authentication flows, allowing developers to focus on building functionality without managing token lifecycles manually.
For full documentation and implementation guides, visit https://developer.paymentus.io

## Project Structure

This project uses a Gradle multi-project build structure with the following modules:

- **auth**: Authentication module
- **xotp**: XOTP module for transaction operations
- **core**: Core SDK that depends on auth and xotp modules

### Build System

The project follows standard Gradle multi-project conventions where:
- Dependencies between modules are handled directly via project references
- All modules share a common version defined at the root level
- Common build logic is defined in the root build.gradle
- Each module defines only its specific configurations

### Building the Project

To build all modules:
```bash
./gradlew build
```

To build a specific module:
```bash
./gradlew :auth:build
./gradlew :xotp:build
./gradlew :core:build
```

### Publishing

To publish all artifacts:
```bash
./gradlew publish
```

This repository contains the Java SDK for the Paymentus Server API

## Packages

The SDK is organized into the following packages:

- `auth`: Authentication and authorization functionality
- `xotp`: XOTP API client and related utilities
- `core`: Core SDK functionality integrating auth and XOTP services

## Getting Started

### Prerequisites

- Java 11 or higher


### Using the SDK

Import the required dependencies in your Maven project:

```xml
<!-- Install and use individual modules -->
<dependencies>
  <dependency>
    <groupId>com.paymentus.sdk</groupId>
    <artifactId>auth</artifactId>
    <version>1.0.0</version>
  </dependency>
  <dependency>
    <groupId>com.paymentus.sdk</groupId>
    <artifactId>xotp</artifactId>
    <version>1.0.0</version>
  </dependency>
</dependencies>

<dependencies>
<!--Or install the core module that includes all other modules  -->
<dependency>
  <groupId>com.paymentus.sdk</groupId>
  <artifactId>core</artifactId>
  <version>1.0.0</version>
</dependency>
</dependencies>
```

## Basic Usage

Here's a simple example of using the Auth SDK:

```java
package com.paymentus.sdk.example;

import java.util.Arrays;

import com.paymentus.sdk.auth.Auth;
import com.paymentus.sdk.auth.AuthSdk;
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;

public class AuthExample {
  public static void main(String[] args) {
    // Create AuthConfig
    AuthConfig config = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(Arrays.asList(Scope.XOTP))
            .build();

    // Initialize AuthClient
    Auth authClient = AuthSdk.createAuthClient(config);
    try {
      // Fetch token
      String token = authClient.fetchToken();
      System.out.println("Token: "+ token);
      boolean isTokenExpired = authClient.isTokenExpired();
      System.out.println("Token Expired: "+ isTokenExpired);
      String currentToken = authClient.getCurrentToken();
      System.out.println("Current Token: "+ currentToken);

    } catch (Exception e) {
      System.err.println("Error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```

## Authentication

The SDK handles authentication automatically when using the Core SDK.

```java
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;

import java.util.List;
/**
 * Auth Example using core SDK
 */
public class AuthUsingCoreSdkExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);
    // Fetch token
    try {
      String token = sdk.auth().fetchToken();
      System.out.println("Token: "+ token);
      boolean isTokenExpired = sdk.auth().isTokenExpired();
      System.out.println("Token Expired: "+ isTokenExpired);
      String currentToken = sdk.auth().getCurrentToken();
      System.out.println("Current Token: "+ currentToken);
    } catch (Exception e) {
      System.err.println("Error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```

Auth can be done by providing pixels in the config
```java
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.PixelType;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;

import java.util.List;

public class AuthUsingPixelsExample {
    public static void main(String[] args) {
        // Create AuthConfig first
        AuthConfig authConfig = AuthConfig.builder()
                .baseUrl("https://<environment>.paymentus.com")
                .preSharedKey("shared-256-bit-secret")
                .tla("ABC")
                .pixels(List.of(PixelType.TOKENIZATION_PIXEL)) // PIXEL used for config
                .build();

        CoreConfig config = CoreConfig.builder()
                .authConfig(authConfig)
                .build();

        SDK sdk = CoreSdk.createSdk(config);
        // Fetch token
        try {
            String token = sdk.auth().fetchToken();
            System.out.println("Token: "+ token);
            boolean isTokenExpired = sdk.auth().isTokenExpired();
            System.out.println("Token Expired: "+ isTokenExpired);
            String currentToken = sdk.auth().getCurrentToken();
            System.out.println("Current Token: "+ currentToken);
        } catch (Exception e) {
            System.err.println("Error occurred: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

## XOTP API

The XOTP API provides access to Paymentus services. It's recommended to use the Core SDK which automatically handles authentication.

### Payment Examples
```java
// Example 1 Make Payment
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.Address;
import com.paymentus.sdk.xotp.model.CreditCardExpiryDate;
import com.paymentus.sdk.xotp.model.Customer;
import com.paymentus.sdk.xotp.model.MakePayment;
import com.paymentus.sdk.xotp.model.MakePaymentItem;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.PaymentMethod;
import com.paymentus.sdk.xotp.model.PaymentMethodTypeEnum;
import com.paymentus.sdk.xotp.model.PaymentResponse;
import com.paymentus.sdk.xotp.types.LogLevel;
import com.paymentus.sdk.xotp.types.MaskingLevel;

import java.util.List;

public class MakePaymentExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PAYMENT))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .logLevel(LogLevel.VERBOSE)
            .maskingLevel(MaskingLevel.ALL_PII) // masking all sensitive data
            .build();

    // Initialize SDK
    SDK sdk = CoreSdk.createSdk(config); //or new SDK(config)

    // Create payment headers
    PaymentHeader header1 = new PaymentHeader()
            .accountNumber("6759375")
            .paymentAmount(11.22)
            .paymentTypeCode("UTILITY");

    PaymentHeader header2 = new PaymentHeader()
            .accountNumber("6759375")
            .paymentAmount(13.22)
            .paymentTypeCode("WATER");

    // Create payment method
    CreditCardExpiryDate expiryDate = new CreditCardExpiryDate()
            .month(12).year(2035);

    PaymentMethod paymentMethod = new PaymentMethod()
            .type(PaymentMethodTypeEnum.VISA)
            .accountNumber("4111111111111111")
            .cardHolderName("Jonny Doe")
            .creditCardExpiryDate(expiryDate);

    // Create address
    Address address = new Address().line1("10 Fifth Ave.").state("NY")
            .zipCode("12345").city("New York").country("US");

    // Create customer
    Customer customer = new Customer().firstName("John").lastName("Doe")
            .email("john@paymentus.com")
            .dayPhoneNr("9051112233").address(address);

    // Create payment item
    MakePaymentItem payment = new MakePaymentItem()
            .addHeaderItem(header1).addHeaderItem(header2)
            .paymentMethod(paymentMethod).customer(customer);

    // Create make payment request
    MakePayment paymentPayload = new MakePayment().payment(payment);

    try {
      // Call Xotp API
      PaymentResponse result = sdk.xotp().makePayment(paymentPayload);
      System.out.println(result.toString());

//		-----	Sample Output -----
// class PaymentResponse {
//	paymentResponse: class PaymentResponseList {
//         response: [
//	     	 class PaymentResponseItem {
//             accountNumber: 6759375
//             convenienceFee: 1.5
//             convenienceFeeCountry: US
//             convenienceFeeState: VA
//             convenienceFeeCategoryCode: null
//             convenienceFeePercent: null
//             errors: null
//             paymentAmount: 11.22
//             paymentComponent: null
//             paymentDate: 06302025053357
//             paymentStatus: ACCEPTED
//             paymentStatusDescription: Approved
//             paymentScheduleStatus: null
//             referenceNumber: 734871
//             status: null
//             stagedId: null
//             totalAmount: 12.72
//             additionalProperties: null
//         },
//          class PaymentResponseItem {
//             accountNumber: 6759375
//             convenienceFee: 1.5
//             convenienceFeeCountry: US
//             convenienceFeeState: VA
//             convenienceFeeCategoryCode: null
//             convenienceFeePercent: null
//             errors: null
//             paymentAmount: 13.22
//             paymentComponent: null
//             paymentDate: 06302025053400
//             paymentStatus: ACCEPTED
//             paymentStatusDescription: Approved
//             paymentScheduleStatus: null
//             referenceNumber: 734870
//             status: null
//             stagedId: null
//             totalAmount: 14.72
//             additionalProperties: null
//         }]
//         additionalProperties: null
//     }
//     additionalProperties: null
// }
    } catch (Exception e) {
      System.err.println("Error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```

```java
// Example 2 Refund Payment
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.Address;
import com.paymentus.sdk.xotp.model.CreditCardExpiryDate;
import com.paymentus.sdk.xotp.model.Customer;
import com.paymentus.sdk.xotp.model.Payment;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.PaymentMethod;
import com.paymentus.sdk.xotp.model.PaymentMethodTypeEnum;
import com.paymentus.sdk.xotp.model.PaymentRequest;
import com.paymentus.sdk.xotp.model.PaymentResponse;
import com.paymentus.sdk.xotp.types.LogLevel;
import com.paymentus.sdk.xotp.types.MaskingLevel;

import java.util.List;

public class RefundPaymentExample {
  public static void main(String[] args) {
    try {
      // Create AuthConfig first
      AuthConfig authConfig = AuthConfig.builder()
              .baseUrl("https://<environment>.paymentus.com")
              .preSharedKey("shared-256-bit-secret")
              .tla("ABC")
              .scope(List.of(Scope.XOTP))
              .build();

      // Create SDK configuration with authConfig
      CoreConfig config = CoreConfig.builder()
              .authConfig(authConfig)
              .logLevel(LogLevel.VERBOSE)
              .maskingLevel(MaskingLevel.ALL_PII) // masking all sensitive data
              .build();

      // Initialize SDK
      SDK sdk = CoreSdk.createSdk(config); //or new SDK(config)

      // Create payment headers
      PaymentHeader header = new PaymentHeader()
              .accountNumber("6759370")
              .referenceNumber("735114")
              .paymentAmount(11.22)
              .paymentTypeCode("UTILITY");

      // Create payment method
      CreditCardExpiryDate expiryDate = new CreditCardExpiryDate()
              .month(12).year(2035);

      PaymentMethod paymentMethod = new PaymentMethod()
              .type(PaymentMethodTypeEnum.VISA_IREFUND)
              .accountNumber("4111111111111111")
              .cardHolderName("John Doe")
              .creditCardExpiryDate(expiryDate);

      // Create address
      Address address = new Address().line1("10 Fifth Ave.").state("NY")
              .zipCode("12345").city("New York").country("US");

      // Create customer
      Customer customer = new Customer().firstName("John").lastName("Doe")
              .email("john@paymentus.com")
              .dayPhoneNr("9051112233").address(address);

      // Create payment item
      Payment payment = new Payment().header(header)
              .paymentMethod(paymentMethod).customer(customer);

      // Create make payment request
      PaymentRequest refundPayload = new PaymentRequest()
              .payment(payment);

      PaymentResponse result = sdk.xotp().refundPayment(refundPayload);
      System.out.println(result.toString());

    } catch (Exception e) {
      System.err.println("Error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```
```java
// Example 3 Payment History
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.PaymentHistoryResponse;
import com.paymentus.sdk.xotp.model.PaymentSearchRequest;

import java.time.LocalDate;
import java.util.List;

public class PaymentHistoryExample {

  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PAYMENT))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config); // or new SDK(config)
    PaymentSearchRequest searchPayload = new PaymentSearchRequest()
            .accountNumber("6759371") // required
            .dateFrom(LocalDate.of(2025, 06, 18)) // optional
            .dateTo(LocalDate.of(2025, 06, 19)); // optional

    try {
      PaymentHistoryResponse result = sdk.xotp().getPaymentHistory(searchPayload);
      System.out.println(result);
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```

```java
// Example 4 Calculate Convenience Fee
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.ConvenienceFeeCountryEnum;
import com.paymentus.sdk.xotp.model.Payment;
import com.paymentus.sdk.xotp.model.PaymentFailedResponse;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.PaymentMethod;
import com.paymentus.sdk.xotp.model.PaymentMethodCategoryEnum;
import com.paymentus.sdk.xotp.model.PaymentMethodTypeEnum;
import com.paymentus.sdk.xotp.model.PaymentRequest;
import com.paymentus.sdk.xotp.model.PaymentResponse;
import com.paymentus.sdk.xotp.types.LogLevel;

import java.util.List;

public class ConvFeeCalculationExample {

  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PAYMENT))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .logLevel(LogLevel.MINIMAL) // minimal logs
            .build();


    SDK sdk = CoreSdk.createSdk(config); // or new SDK(config)

    PaymentHeader header = new PaymentHeader()
            .paymentAmount(22.00)
            .convenienceFeeState("NY")
            .convenienceFeeCountry(ConvenienceFeeCountryEnum.US);

    PaymentMethod method = new PaymentMethod()
            .type(PaymentMethodTypeEnum.MC);

    Payment payment = new Payment()
            .header(header)
            .paymentMethod(method)
            .paymentMethodCategory(PaymentMethodCategoryEnum.CC); // comment this line to generate 400 error response

    PaymentRequest feePayload = new PaymentRequest()
            .payment(payment);

    try {
      PaymentResponse result = sdk.xotp().cnvCalculation(feePayload);
      System.out.println(result);
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400) {
        PaymentFailedResponse errorResponse = e.getErrorBodyAs(PaymentFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch(Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```
```java
// Example 5 Fetch Last Payment
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.PaymentSearchRequest;
import com.paymentus.sdk.xotp.model.PaymentSearchResponse;
import com.paymentus.sdk.xotp.types.LogLevel;
import com.paymentus.sdk.xotp.types.MaskingLevel;

public class FetchLastPaymentExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PAYMENT))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .logLevel(LogLevel.NORMAL)
            .maskingLevel(MaskingLevel.ALL_PII) // masking all sensitive data
            .build();


    // Initialize SDK
    SDK sdk = CoreSdk.createSdk(config); // or new SDK(config)

    PaymentSearchRequest payload = new PaymentSearchRequest()
            .accountNumber("6759371")
            .paymentTypeCode("UTILITY")
            .authToken1("12345");
    try {
      PaymentSearchResponse result = sdk.xotp().fetchLastPayment(payload);
      System.out.println(result.toString());
    } catch (Exception e) {
      // Catch any other unexpected exceptions (e.g., network issues)
      System.err.println("An unexpected error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```
```java
// Example 6 Stage Payment
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.Customer;
import com.paymentus.sdk.xotp.model.Payment;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.PaymentRequest;
import com.paymentus.sdk.xotp.model.StagePaymentResponse;

import java.util.List;

public class StagePaymentExample {

  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    PaymentHeader header = new PaymentHeader()
            .accountNumber("6759370")
            .paymentAmount(13.21)
            .paymentTypeCode("UTILITY")
            .authToken1("12345");

    Customer customer = new Customer()
            .firstName("John")
            .lastName("Doe")
            .email("john@paymentus.com")
            .dayPhoneNr("9051112233");

    Payment payment = new Payment()
            .header(header)
            .customer(customer);

    PaymentRequest stagePaymentPayload = new PaymentRequest()
            .payment(payment);

    try {
      StagePaymentResponse result = sdk.xotp().stagePayment(stagePaymentPayload);
      System.out.println(result.toString());
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```
### Autopay Examples
```java
// Example 1 Create Autopay
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.AutopayFailedResponse;
import com.paymentus.sdk.xotp.model.AutopayRequest;
import com.paymentus.sdk.xotp.model.AutopayResponse;
import com.paymentus.sdk.xotp.model.Customer;
import com.paymentus.sdk.xotp.model.Payment;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.PaymentMethod;
import com.paymentus.sdk.xotp.model.ScheduleTypeCodeEnum;

import java.util.List;

public class CreateAutopayExample {

  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_AUTOPAY))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    PaymentHeader header = new PaymentHeader()
            .accountNumber("6759370")
            .paymentTypeCode("UTILITY")
            .scheduleTypeCode(ScheduleTypeCodeEnum.MONTHLY)
            .scheduleDay(3)
            .paymentAmount(21.01);

    PaymentMethod paymentMethod = new PaymentMethod()
            .token("827BFC458708F0B442009C9C9836F7E4B65557FB");

    Customer customer = new Customer()
            .firstName("John")
            .lastName("Doe")
            .email("john@paymentus.com")
            .dayPhoneNr("9051112233");

    Payment paymentSchedule = new Payment()
            .header(header)
            .paymentMethod(paymentMethod)
            .customer(customer);

    AutopayRequest autopayPayload = new AutopayRequest()
            .paymentSchedule(paymentSchedule);

    try {
      AutopayResponse result = sdk.xotp().createAutopay(autopayPayload);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400) {
        AutopayFailedResponse errorResponse = e.getErrorBodyAs(AutopayFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch(Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}

```
```java
// Example 2 Update Autopay
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.AutopayFailedResponse;
import com.paymentus.sdk.xotp.model.AutopayRequest;
import com.paymentus.sdk.xotp.model.AutopayResponse;
import com.paymentus.sdk.xotp.model.Payment;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.ScheduleTypeCodeEnum;

import java.util.List;

public class UpdateAutopayExample {

  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_AUTOPAY))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    PaymentHeader header = new PaymentHeader()
            .scheduleTypeCode(ScheduleTypeCodeEnum.MONTHLY)
            .scheduleDay(3)
            .paymentAmount(11.09);

    Payment paymentSchedule = new Payment()
            .header(header);

    AutopayRequest autopayPayload = new AutopayRequest()
            .paymentSchedule(paymentSchedule);

    try {
      String referenceNumber = "3433";
      AutopayResponse result = sdk.xotp().updateAutopay(referenceNumber, autopayPayload);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400) {
        AutopayFailedResponse errorResponse = e.getErrorBodyAs(AutopayFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch(Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```
```java
// Example 3 Get Autopay By Reference Number
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.AutopayFailedResponse;
import com.paymentus.sdk.xotp.model.AutopayFindResponse;

import java.util.List;

public class GetAutopayByReferenceNumberExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_AUTOPAY))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();


    SDK sdk = CoreSdk.createSdk(config);

    String referenceNumber = "3413";
    try {
      AutopayFindResponse result = sdk.xotp().getAutopay(referenceNumber);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode == 404) {
        AutopayFailedResponse errorResponse = e.getErrorBodyAs(AutopayFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500 etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}


```
```java
// Example 4 List Autopay
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.AutopayListResponse;
import com.paymentus.sdk.xotp.model.AutopaySearchRequest;

import java.util.List;

public class ListAutopayExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_AUTOPAY))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    AutopaySearchRequest listPayload = new AutopaySearchRequest()
            .loginId("john@paymentus.com")
            .accountNumber("6759370");

    AutopayListResponse result = sdk.xotp().listAutoPay(listPayload);
    System.out.println(result.toString());
  }
}
```
```java
// Example 5 Delete Autopay
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.AutopayFailedResponse;
import com.paymentus.sdk.xotp.model.AutopayResponse;

import java.util.List;

public class DeleteAutopayExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);
    String referenceNumber = "3413";

    try {
      AutopayResponse result = sdk.xotp().deleteAutopay(referenceNumber);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        AutopayFailedResponse errorResponse = e.getErrorBodyAs(AutopayFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```
```java
// Example 6 Stage Autopay
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.Customer;
import com.paymentus.sdk.xotp.model.Payment;
import com.paymentus.sdk.xotp.model.PaymentHeader;
import com.paymentus.sdk.xotp.model.PaymentRequest;
import com.paymentus.sdk.xotp.model.ScheduleTypeCodeEnum;
import com.paymentus.sdk.xotp.model.StagePaymentResponse;

import java.time.LocalDate;
import java.util.List;

public class StageAutopayExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    PaymentHeader header = new PaymentHeader()
            .accountNumber("6759370")
            .paymentAmount(13.21)
            .paymentTypeCode("UTILITY")
            .scheduleTypeCode(ScheduleTypeCodeEnum.MONTHLY)
            .scheduleStartDate(LocalDate.of(2025, 8, 15))
            .scheduleDay(11)
            .authToken1("12345");

    Customer customer = new Customer()
            .firstName("John")
            .lastName("Doe")
            .email("john@paymentus.com")
            .dayPhoneNr("9051112233");

    Payment payment = new Payment()
            .header(header)
            .customer(customer);

    PaymentRequest stageAutopayPayload = new PaymentRequest()
            .payment(payment);

    try {
      StagePaymentResponse result = sdk.xotp().stageAutopay(stageAutopayPayload);
      System.out.println(result.toString());
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
      e.printStackTrace();
    }
  }
}

```



### Accounts Examples
```java
// Example 1 Get Account Info By AccountNumber
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.ListAccountInfoResponse;

import java.util.List;

public class AccountInfoByAccountNumberExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    String accountNumber = "6759370";
    try {
      SDK sdk = CoreSdk.createSdk(config);
      ListAccountInfoResponse result = sdk.xotp().getAccountInfoByAccountNumber(accountNumber);
      System.out.println(result.toString());
    } catch (Exception e) {
      System.err.println("An unexpected error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```
```java
// Example 2 Get Account Info By Email
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.ListAccountInfoResponse;

import java.util.List;

public class AccountInfoByEmailExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();
    String email = "john@paymentus.com";
    try {
      SDK sdk = CoreSdk.createSdk(config);
      ListAccountInfoResponse result = sdk.xotp().getAccountInfoByEmail(email);
      System.out.println(result.toString());
    } catch (Exception e) {
      System.err.println("An unexpected error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```
```java
// Example 3 Account Inquiry
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.AccountInquiryRequest;
import com.paymentus.sdk.xotp.model.AccountInquiryResponse;

import java.util.List;

public class AccountInquiryExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_LIST_ACCOUNTS))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    AccountInquiryRequest payload = new AccountInquiryRequest()
            .accountNumber("6759370")
            .paymentTypeCode("UTILITY")
            .authToken1("12345")
            .includeSchedules(true)
            .includeLastUsedPm(true)
            .detailedInfo(true);

    try {
      SDK sdk = CoreSdk.createSdk(config);
      AccountInquiryResponse result = sdk.xotp().accountInquiry(payload);
      System.out.println(result.toString());
    } catch (Exception e) {
      System.err.println("An unexpected error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```

### User Examples

```java
// Example 1 Create User
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.ClientAccountItem;
import com.paymentus.sdk.xotp.model.UserFailedResponse;
import com.paymentus.sdk.xotp.model.UserInfo;
import com.paymentus.sdk.xotp.model.UserProfileInfo;
import com.paymentus.sdk.xotp.model.UserRequest;
import com.paymentus.sdk.xotp.model.UserRequestItem;
import com.paymentus.sdk.xotp.model.UserResponse;

import java.util.Collections;
import java.util.List;

public class CreateUserExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();


    SDK sdk = CoreSdk.createSdk(config);

    UserProfileInfo profile = new UserProfileInfo()
            .firstName("Jacky")
            .lastName("Doe")
            .email("jacky@paymentus.com");

    UserInfo userInfo = new UserInfo()
            .loginId("jacky@paymentus.com")
            .password("SecretPassword123")
            .forcePasswordChange(true)
            .profile(profile);

    ClientAccountItem clientAccount = new ClientAccountItem()
            .accountNumber("6759374")
            .paymentTypeCode("UTILITY")
            .authToken1("12345");

    UserRequestItem userProfile = new UserRequestItem()
            .userInfo(userInfo)
            .clientAccount(Collections.singletonList(clientAccount));

    UserRequest createUser = new UserRequest().userProfile(userProfile);

    try {
      UserResponse result = sdk.xotp().createUser(createUser);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400) {
        UserFailedResponse errorResponse = e.getErrorBodyAs(UserFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Error: " + e.getMessage());
    }
  }
}

```
```java
// Example 2 Update User
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.UserFailedResponse;
import com.paymentus.sdk.xotp.model.UserInfo;
import com.paymentus.sdk.xotp.model.UserLoginId;
import com.paymentus.sdk.xotp.model.UserProfileInfo;
import com.paymentus.sdk.xotp.model.UserResponse;
import com.paymentus.sdk.xotp.model.UserUpdateRequest;
import com.paymentus.sdk.xotp.model.UserUpdateRequestItem;

import java.util.List;

public class UpdateUserExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    UserProfileInfo profile = new UserProfileInfo()
            .dayPhoneNr("5559098888")
            .zipCode("28202");

    UserInfo userInfo = new UserInfo()
            .loginId("jacky@paymentus.com")
            .profile(profile);

    UserLoginId userWithPermission = new UserLoginId()
            .loginId("admin@paymentus.com");

    UserUpdateRequestItem userProfile = new UserUpdateRequestItem()
            .userInfo(userInfo)
            .user(userWithPermission);

    UserUpdateRequest updateUser = new UserUpdateRequest()
            .userProfile(userProfile);


    try {
      UserResponse result = sdk.xotp().updateUser(updateUser);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        UserFailedResponse errorResponse = e.getErrorBodyAs(UserFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Error: " + e.getMessage());
    }
  }
}
```
```java
// Example 3 Get User
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.UserFailedResponse;
import com.paymentus.sdk.xotp.model.UserFindResponse;

public class GetUserExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);
    try {
      boolean includeContactInfo = true;
      UserFindResponse result = sdk.xotp().getUser("jacky@paymentus.com", includeContactInfo);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        UserFailedResponse errorResponse = e.getErrorBodyAs(UserFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Error: " + e.getMessage());
    }
  }
}
```
```java
// Example 4 Delete User
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.UserDeleteRequest;
import com.paymentus.sdk.xotp.model.UserDeleteRequestItem;
import com.paymentus.sdk.xotp.model.UserFailedResponse;
import com.paymentus.sdk.xotp.model.UserLoginId;
import com.paymentus.sdk.xotp.model.UserResponse;
import com.paymentus.sdk.xotp.types.LogLevel;

import java.util.List;

public class DeleteUserExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .logLevel(LogLevel.VERBOSE)
            .build();
    SDK sdk = CoreSdk.createSdk(config);

    UserLoginId userToDelete = new UserLoginId()
            .loginId("jacky@paymentus.com");

    UserLoginId userWithPermission = new UserLoginId()
            .loginId("admin@paymentus.com");

    UserDeleteRequestItem user = new UserDeleteRequestItem()
            .userInfo(userToDelete)
            .user(userWithPermission);

    UserDeleteRequest deleteUser = new UserDeleteRequest()
            .userProfile(user);

    try {
      UserResponse result = sdk.xotp().deleteUser(deleteUser);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        UserFailedResponse errorResponse = e.getErrorBodyAs(UserFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Error: " + e.getMessage());
    }
  }
}
```

### Profile Examples
```java
// Example 1 Create Profile
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.CreditCardExpiryDate;
import com.paymentus.sdk.xotp.model.PaymentMethod;
import com.paymentus.sdk.xotp.model.PaymentMethodTypeEnum;
import com.paymentus.sdk.xotp.model.ProfileCustomer;
import com.paymentus.sdk.xotp.model.ProfileFailedResponse;
import com.paymentus.sdk.xotp.model.ProfileRequest;
import com.paymentus.sdk.xotp.model.ProfileRequestItem;
import com.paymentus.sdk.xotp.model.ProfileResponse;
import com.paymentus.sdk.xotp.model.ProfileUserInfo;

import java.util.List;

public class CreateProfileExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PROFILE))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    PaymentMethod paymentMethod = new PaymentMethod()
            .type(PaymentMethodTypeEnum.VISA)
            .accountNumber("4444444444444448")
            .creditCardExpiryDate(new CreditCardExpiryDate()
                    .month(12)
                    .year(2035))
            .cardHolderName("Jacky Doe");

    ProfileUserInfo userInfo = new ProfileUserInfo()
            .loginId("jacky@paymentus.com");

    ProfileCustomer customer = new ProfileCustomer()
            .firstName("Jacky")
            .lastName("Doe");

    ProfileRequestItem profileItem = new ProfileRequestItem()
            .paymentMethod(paymentMethod)
            .customer(customer)
            .userInfo(userInfo);

    ProfileRequest payload = new ProfileRequest().profile(profileItem);
    try {
      ProfileResponse result = sdk.xotp().createProfile(payload);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400) {
        ProfileFailedResponse errorResponse = e.getErrorBodyAs(ProfileFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```
```java
// Example 2 Update Profile
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.ProfileFailedResponse;
import com.paymentus.sdk.xotp.model.ProfileResponse;
import com.paymentus.sdk.xotp.model.ProfileUpdateItem;
import com.paymentus.sdk.xotp.model.ProfileUpdateRequest;

import java.util.List;

public class UpdateProfileExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PROFILE))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);
    ProfileUpdateItem profileItem = new ProfileUpdateItem()
            .profileDescription("Jacky Main Payment Profile")
            .defaultFlag(true);

    ProfileUpdateRequest payload = new ProfileUpdateRequest().profile(profileItem);

    String profileToken = "some-token";

    try {
      ProfileResponse result = sdk.xotp().updateProfile(profileToken, payload);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400) {
        ProfileFailedResponse errorResponse = e.getErrorBodyAs(ProfileFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```
```java
// Example 3 Delete Profile
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.ProfileFailedResponse;
import com.paymentus.sdk.xotp.model.ProfileResponse;

import java.util.Arrays;

public class DeleteProfileExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(Arrays.asList(Scope.XOTP_PROFILE,
                    Scope.XOTP_PROFILE_DELETE))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    String profileToken = "some-token";
    try {
      ProfileResponse result = sdk.xotp().deleteProfile(profileToken);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        ProfileFailedResponse errorResponse = e.getErrorBodyAs(ProfileFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```
```java
// Example 4 Get Profile
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.ProfileFailedResponse;
import com.paymentus.sdk.xotp.model.ProfileResponse;

import java.util.List;

public class GetProfileExample {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP_PROFILE))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    String profileToken = "some-token";
    try {
      ProfileResponse result = sdk.xotp().getProfile(profileToken);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        ProfileFailedResponse errorResponse = e.getErrorBodyAs(ProfileFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}

```
```java
// Example 5 List Profile
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.ListProfilesResponse;
import com.paymentus.sdk.xotp.model.ProfileFailedResponse;

import java.util.Arrays;

public class ListProfilesExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(Arrays.asList(
                    Scope.XOTP_PROFILE,
                    Scope.XOTP_LIST_PROFILES))
            .build();

    // Create SDK configuration with authConfig
    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();

    SDK sdk = CoreSdk.createSdk(config);

    String loginId = "jacky@paymentus.com";
    try {
      ListProfilesResponse result = sdk.xotp().getProfiles(loginId);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 400 bad request
      if (e.getCode() == 400 || e.getCode() == 404) {
        ProfileFailedResponse errorResponse = e.getErrorBodyAs(ProfileFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 404, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch (Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }
  }
}
```



### Other Examples
```java
// Example 1 Get Bank Info
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.exceptions.XotpApiException;
import com.paymentus.sdk.xotp.model.BankInfoFailedResponse;
import com.paymentus.sdk.xotp.model.BankInfoResponse;

import java.util.List;

public class BankInfoExample {
  public static void main(String[] args) {
    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();


    SDK sdk = CoreSdk.createSdk(config);

    String routingNumber = "021000128";
    try {
      BankInfoResponse result = sdk.xotp().getBankInfo(routingNumber);
      System.out.println(result.toString());
    } catch (XotpApiException e) {
      // 404 invalid routing number
      if (e.getCode() == 404) {
        BankInfoFailedResponse errorResponse = e.getErrorBodyAs(BankInfoFailedResponse.class);
        System.err.println(errorResponse.toString());
      } else {
        // Handle other API exceptions (e.g., 500, 400, etc.)
        System.err.println("API Exception: " + e.getCode() + " - " + e.getMessage());
        System.err.println("Response Body: " + e.getResponseBody());
      }
    } catch(Exception e) {
      System.err.println("Exception: " + e.getMessage());
    }

  }
}

```
```java
// Example 2 Resend Email Confirmation
import com.paymentus.sdk.auth.models.AuthConfig;
import com.paymentus.sdk.auth.types.Scope;
import com.paymentus.sdk.core.CoreSdk;
import com.paymentus.sdk.core.SDK;
import com.paymentus.sdk.core.models.CoreConfig;
import com.paymentus.sdk.xotp.model.ResendEmailRequest;
import com.paymentus.sdk.xotp.model.ResendEmailRequestHeader;
import com.paymentus.sdk.xotp.model.ResendEmailRequestItem;
import com.paymentus.sdk.xotp.model.ResendEmailResponse;

import java.util.List;

public class ResendEmailConfirmationExample  {
  public static void main(String[] args) {

    // Create AuthConfig first
    AuthConfig authConfig = AuthConfig.builder()
            .baseUrl("https://<environment>.paymentus.com")
            .preSharedKey("shared-256-bit-secret")
            .tla("ABC")
            .scope(List.of(Scope.XOTP))
            .build();

    CoreConfig config = CoreConfig.builder()
            .authConfig(authConfig)
            .build();


    SDK sdk = CoreSdk.createSdk(config);
    ResendEmailRequestHeader header = new ResendEmailRequestHeader()
            .accountNumber("6759370")
            .referenceNumber("731358");

    ResendEmailRequestItem payment = new ResendEmailRequestItem().header(header);
    ResendEmailRequest resendRequest = new ResendEmailRequest().payment(payment);

    try {
      ResendEmailResponse result = sdk.xotp().resendEmailConfirmation(resendRequest);
      System.out.println(result.toJson());
    } catch (Exception e) {
      System.err.println("An unexpected error occurred: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```
## Custom Logging Middleware

You can implement custom logging middleware by creating a class that implements the Logger interface. Here's an example:
```java
import com.paymentus.sdk.xotp.util.Logger;

public class MyCustomLogger implements Logger {

  @Override
  public void info(String message, Object... args) {
    String formattedMessage = String.format(message, args);
    System.out.println("[[INFO]] -- " +formattedMessage);
  }

  @Override
  public void error(String message, Object... args) {
    String formattedMessage = String.format(message, args);
    System.out.println("[[ERROR]] -- " +formattedMessage);
  }

  @Override
  public void warn(String message, Object... args) {
    String formattedMessage = String.format(message, args);
    System.out.println("[[WARN]] -- " +formattedMessage);
  }

  @Override
  public void debug(String message, Object... args) {
    String formattedMessage = String.format(message, args);
    System.out.println("[[DEBUG]] -- " +formattedMessage);
  }
}

// Then provide custom logger while initializing sdk
AuthConfig authConfig = AuthConfig.builder()
        .baseUrl("https://<environment>.paymentus.com")
        .preSharedKey("shared-256-bit-secret")
        .tla("ABC")
        .scope(List.of(Scope.XOTP))
        .build();
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .logger(new MyCustomLogger()) // Provide Custom Logger
        .build();
```
### Available Log Levels

The SDK supports the following log levels to control the verbosity of logging output:

#### SILENT
Suppresses all logging output. Use this in production environments where you want to minimize overhead and don't need operational visibility.
```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .logLevel(LogLevel.SILENT) // For No logging
        .build();
```

#### MINIMAL
Logs only essential information such as request initiation and completion status with response codes. Use this for basic operational monitoring with minimal verbosity.

```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .logLevel(LogLevel.MINIMAL) // For Minimal logging
        .build();
```

#### NORMAL
The default log level. Logs request/response information including basic request bodies and error details. Suitable for most development and staging environments.

```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .logLevel(LogLevel.NORMAL) // Default logging mode
        .build();
```

#### VERBOSE
Maximum logging level that includes all request/response details, headers, and timing information. Ideal for debugging integration issues in development environments.

```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .logLevel(LogLevel.NORMAL) // For Maximum logging
        .build();
```

### Available Masking Levels

The SDK also supports data masking to protect sensitive information in logs:

- **NONE**: No data masking is applied. Only use in secure environments where logs are properly protected.
```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .maskingLevel(MaskingLevel.NONE) // No masking in logs
        .build();
```

- **PCI_ONLY**: Masks payment card information (account numbers, CVV) while preserving other data for debugging.
```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .maskingLevel(MaskingLevel.PCI_ONLY) // Default Mode
        .build();
```

- **ALL_PII**: Maximum protection that masks all personally identifiable information including names, addresses, and payment data.
```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .maskingLevel(MaskingLevel.ALL_PII) // Maximum masking while logging
        .build();
```

### Combining Log Levels with Custom Loggers

You can combine log levels with a custom logger implementation for advanced logging scenarios:
```java
CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .logger(new MyCustomLogger()) // Custom Logger
        .maskingLevel(MaskingLevel.ALL_PII) // Maximum masking
        .logLevel(LogLevel.VERBOSE) // Maximum logging
        .build();
```


## Available Scopes

The SDK supports the following scopes:

- `xotp` - Basic XOTP functionality
- `xotp:profile` - Profile management
- `xotp:profile:read` - Read profile data
- `xotp:profile:create` - Create profiles
- `xotp:profile:update` - Update profiles
- `xotp:profile:delete` - Delete profiles
- `xotp:listProfiles` - List profiles
- `xotp:payment` - Payment processing
- `xotp:autopay` - Autopay functionality
- `xotp:autopay:delete` - Delete autopay settings
- `xotp:accounts` - Account management
- `xotp:accounts:listAccounts` - List accounts

### Pixel Integration

The SDK provides automatic scope mapping for Paymentus pixels. When you specify pixels in the configuration, the SDK automatically adds the required scopes and validates required claims:

```java
AuthConfig authConfig = AuthConfig.builder()
        .baseUrl("https://<environment>.paymentus.com")
        .preSharedKey("shared-256-bit-secret")
        .tla("ABC")
        .pixels(Arrays.asList(PixelType.LIST_WALLETS_PIXEL))
        .userLogin("john@paymentus.com")
        .paymentsData(Arrays.asList(new PaymentData("6759370")))
        .build();

CoreConfig config = CoreConfig.builder()
        .authConfig(authConfig)
        .build()
```

Available pixels and their requirements:

- `tokenization-pixel`:
  - Scopes: `['xotp:profile']`
  - Claims: None required

- `list-wallets-pixel`:
  - Scopes: `['xotp:profile', 'xotp:listProfiles']`
  - Claims: `userLogin` required

- `user-checkout-pixel`:
  - Scopes: `['xotp:profile', 'xotp:payment', 'xotp:listProfiles', 'xotp:accounts:listAccounts']`
  - Claims: `userLogin` and `paymentsData` required, `pmTokens` optional

- `guest-checkout-pixel`:
  - Scopes: `['xotp:payment', 'xotp:accounts:listAccounts']`
  - Claims: `paymentsData` required

- `user-autopay-pixel`:
  - Scopes: `['xotp:profile', 'xotp:autopay', 'xotp:listProfiles', 'xotp:accounts:listAccounts']`
  - Claims: `userLogin` and `paymentsData` required, `pmTokens` optional


## Error Handling

The SDK provides specific error types for different scenarios:

- `AuthException`: Authentication Failed
- `ConfigurationException`: Invalid configuration (e.g., empty scope array)
- `NetworkException`: API request failures
- `TokenException`: Token validation or authentication failures

```
try {
    sdk.auth().fetchToken();
} catch (AuthException e) {
    System.err.println("Authentication Failed: " + e.getMessage());
} catch (ConfigurationException e) {
    System.err.println("Invalid Configuration: " + e.getMessage());
} catch (NetworkException e) {
    System.err.println("Network Error: " + e.getMessage());
} catch (TokenException e) {
    System.err.println("Invalid Token: " + e.getMessage());
}
```

## Disclaimer

These SDKs are intended for use with the URLs and keys that are provided to you for your company by Paymentus. If you do not have this information, please reach out to your implementation or account manager. If you are interested in learning more about the solutions that Paymentus provides, you can visit our website at paymentus.com. You can request access to our complete documentation at developer.paymentus.io. If you are currently not a customer or partner and would like to learn more about the solution and how you can get started with Paymentus, please contact us at https://www.paymentus.com/lets-talk/.

## Contact us

If you have any questions or need assistance, please contact us at sdksupport@paymentus.com.

## License
This project is proprietary and confidential. 