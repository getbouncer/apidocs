# iOS integration guide

## Installing the CardVerify module

Please follow the [general instructions](../bouncer-scan/verifying-high-risk-cards/ios-integration-guide/) for installing, setting up permissions, and configuring CardVerify.

## Using Bouncer Scan to verify cards when users add them

Our `VerifyCardAddViewController`  proactively verifies any cards that users add to your platform.  To support this flow, you should use the `VerifyCardAddViewController` but you don't need to implement the optional `fraudModelResultsVerifyCardAdd` protocol method in the `VerifyCardAddResult` protocol:

```swift
import CardVerify
import UIKit

class VerifyAddFlowViewController: UIViewController, VerifyCardExplanationResult, VerifyCardAddResult {
    @IBAction func buttonPressed() {
        let vc = VerifyCardAddViewController(userId: self.currentUser.userId)
        vc.cardAddDelegate = self
        
        // only use this if you're using a "default to scan" flow
        vc.enableManualEntry = true
        
        viewController.present(vc, animated: true, completion: nil)
    }
    
    // MARK: -VerifyCardAddResult protocol implementation
    func userDidCancelCardAdd(_ viewController: UIViewController) {
        dismiss(animated: true)
    }
    
    func userDidScanCardAdd(_ viewController: UIViewController, creditCard: CreditCard) {
        // Add the card details in `creditCard` to your card entry form
    }
    
    func userDidPressManualCardAdd(_ viewController: UIViewController) {
        // The user pressed the "enter details manually" button
        // navigate to your card entry form without filling in details
    }
}
```

## Instrumenting credit card forms

To instrument credit card forms, add the following code to your card entry ViewController:

{% tabs %}
{% tab title="Stripe Form" %}
```swift
import CardVerify
import Stripe
import UIKit

class PaymentFormViewController: UIViewController {

    @IBOutlet weak var paymentCardTextField: STPPaymentCardTextField!
    @IBOutlet weak var addCardButton: UIButton!
    
    var creditCardFormListener: BouncerFormInstrumentation?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        guard let viewWithForm = paymentCardTextField.subviews.first else { return }
        creditCardFormListener = BouncerFormInstrumentation(userId: self.currentUser.userId, viewWithForm: viewWithForm, fieldOrder: [.cardNumber, .expiry, .cvv, .postalCode], advanceButton: addCardButton)
    }
}
```
{% endtab %}

{% tab title="Custom Form" %}
```swift
import CardVerify
import UIKit

class PaymentFormViewController: UIViewController {

    @IBOutlet weak var cardNumberTextField: UITextField!
    @IBOutlet weak var expiryTextField: UITextField!
    @IBOutlet weak var cvvTextField: UITextField!
    @IBOutlet weak var postalCodeTextField: UITextField!
    @IBOutlet weak var addCardButton: UIButton!
    
    var creditCardFormListener: BouncerFormInstrumentation?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        guard let viewWithForm = paymentCardTextField.subviews.first else { return }
        creditCardFormListener = BouncerFormInstrumentation(userId: self.currentUser.userId, cardNumberTextField: cardNumberTextField, expiryTextField: expiryTextField, advanceButton: addCardButton, cvvTextField: cvvTextField, postalCodeTextField: postalCodeTextField)
    }
}
```
{% endtab %}
{% endtabs %}

## Logging signup and login events

To provide signals for our risk engine, we also log events around signup and login actions.

### Signup events

In your user signup flow, create an `UserCreateEvent` and invoke `recordSuccess` after creating a user successfully:

```swift
import CardVerify

class SignupViewController: UIViewController {

    private let userCreateEvent = UserCreateEvent()
    
    private func createUser(userName: String, password: String) -> Bool {
        // create a user
        val user = MyApi.createUser(userName, password)
        if (user != null) {
            userCreateEvent.recordSuccess(userId: user.id, loginIdentifier: userName)
        }

        return user != null
    }
}
```

### Login events

In your login flow, create an `UserLoginEvent` object and invoke the `recordSuccess` method after a successful login attempt, and invoke the `recordFailure` method after an unsuccessful login attempt.

```swift
class UserLoginViewController: UIViewController {

    private let userLoginEvent = UserLoginEvent()

    private func userLogIn(userName: String, password: String) {
        // log the user in
        guard let user = MyApi.getUserFromUserName(userName) else {
            showLoginFailed()
            return
        }
        user.logIn(password) { authToken in
            if (authToken != null) {
                userLoginEvent.recordSuccess(userId: user.id, loginIdentifier: userName)
            } else {
                userLoginEvent.recordFailure(loginIdentifier: userName)
                showLoginFailed()
            }
        }
    }
}
```

