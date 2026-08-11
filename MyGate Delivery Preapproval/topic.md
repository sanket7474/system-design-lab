# Automatic Pre-Approval Notification for App-Based Deliveries - Requirements

## 1. Background

Modern gated-community management platforms often provide visitor and delivery management features for residents. In some communities, when a resident orders food or goods from selected third-party apps such as Zomato, Swiggy, Blinkit, Zepto, Amazon, Flipkart, or similar platforms, the gated-community app may show a **pre-approval notification** for the expected delivery.

The interesting part of this problem is that the resident may not have explicitly linked the third-party app account with the gated-community app. The resident may also not have provided their gated-community app account ID, flat ID, or resident profile details inside the e-commerce or food-delivery application.

Despite this, the gated-community app is somehow able to infer that a delivery is expected for a particular resident or address and sends a notification asking the resident to pre-approve the delivery person before they arrive at the gate.

---

## 2. Problem Statement

Design a system that enables a gated-community management platform to send a **pre-approval notification** to a resident when a delivery order is placed from selected third-party apps, even when the resident has not explicitly linked their third-party app account with the gated-community app.

The system should be able to detect or infer that a delivery is expected at a particular apartment, villa, or household inside a gated community and notify the relevant resident in advance.

The notification should allow the resident to pre-approve the delivery so that the security gate can process the delivery person faster when they arrive.

---

## 3. Example Scenario

A resident lives in a gated apartment community and uses a gated-community app for visitor approvals.

The resident orders food from a supported delivery platform such as Zomato or Swiggy.

The resident does not explicitly link their Zomato or Swiggy account with the gated-community app.

The resident also does not enter any gated-community app account identifier while placing the order.

Shortly after placing the order, the gated-community app sends a notification such as:

> "You have an upcoming food delivery. Would you like to pre-approve the delivery partner?"

The resident taps **Approve**.

Later, when the delivery partner reaches the security gate, the guard already sees that the delivery is expected and pre-approved.

---

## 4. Scope of the Problem

The system should focus on the following capabilities:

- Detect that a delivery order has been placed for an address inside a managed gated community.
- Identify the likely household, flat, or resident associated with the delivery.
- Notify the resident before the delivery partner reaches the gate.
- Allow the resident to pre-approve or reject the upcoming delivery.
- Help the gate security staff validate the delivery when the delivery person arrives.
- Handle cases where the mapping between delivery address and resident account is uncertain.
- Avoid exposing unnecessary personal information between third-party apps, residents, guards, and the gated-community platform.


## 6. Key Functional Requirements

### 6.1 Delivery Signal Detection

The system should receive or detect a signal that an order has been placed from a supported third-party app and is expected to be delivered to an address inside a gated community.

The signal may contain partial information such as:

- Delivery address
- Community or society name
- Tower or block name
- Flat or house number
- Delivery platform name
- Estimated delivery time
- Delivery partner details, if available
- Order category, such as food, groceries, package, or courier


### 6.2 Resident or Household Identification

The system should attempt to map the delivery signal to a resident, family, household, flat, or unit inside the gated community.

The mapping may be exact or uncertain depending on the quality of the incoming information.

The system should consider cases where:

- The address is written in different formats.
- The flat number is missing or ambiguous.
- Multiple residents live in the same flat.
- The order was placed by a guest, family member, tenant, or domestic helper.
- The third-party app has old or partially incorrect address data.

### 6.3 Pre-Approval Notification

Once a likely resident or household is identified, the system should send a notification asking whether the delivery should be pre-approved.

The notification should ideally include enough context for the resident to make a decision, without exposing sensitive information unnecessarily.

Example notification content:

- Delivery platform name
- Approximate expected arrival time
- Type of delivery
- Delivery person's name or masked phone number, if available
- Action buttons such as Approve, Reject, or Not Mine

### 6.4 Gate-Side Validation

When the delivery partner reaches the community gate, the guard-facing system should be able to verify whether the delivery was pre-approved by the resident.

The guard should be able to distinguish between:

- Pre-approved delivery
- Delivery pending resident approval
- Delivery rejected by resident
- Delivery not recognized by the system
- Delivery mapped to the wrong unit

### 6.5 Resident Feedback

The resident should be able to indicate when the notification is incorrect.

Examples:

- "This is not my order"
- "Wrong flat"
- "I did not order anything"
- "Do not auto-suggest from this platform"

This feedback may be used to improve future matching and reduce false notifications.

---

## 7. Non-Functional Requirements

### 7.1 Privacy

The system should minimize the amount of personal information exchanged or displayed.

It should avoid exposing sensitive order details to the gated-community platform, security guards, or unrelated residents unless necessary.

### 7.2 Security

The system should prevent unauthorized users from approving deliveries for another resident or unit.

It should also ensure that delivery-related signals cannot be spoofed to gain entry into the community.

### 7.3 Accuracy

The system should aim to reduce false positives and false negatives.

A false positive occurs when a resident receives a pre-approval notification for an order that is not theirs.

A false negative occurs when a genuine delivery order does not generate a pre-approval notification.

### 7.4 Latency

The notification should be sent quickly enough for the resident to act before the delivery partner reaches the gate.

### 7.5 Availability

The system should remain available during high traffic periods such as lunch time, dinner time, weekends, festivals, and sale events.

### 7.6 Scalability

The system should support many communities, residents, delivery platforms, and concurrent delivery events.

### 7.7 Auditability

The system should maintain an audit trail of key events, such as notification sent, approval received, gate entry created, rejection received, and manual override by guard.

---