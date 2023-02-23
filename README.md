# BiometricDemo

```swift
import UIKit
import LocalAuthentication

enum LocalAuthenticationError: Error {
    case biometricNotAvailable
    case forwarded(Error)
    case unknown
}

class ViewController: UIViewController {
    
    private func authenticateWithBiomtric(completion: @escaping (Error?) -> Void) {
        
        let authenticationContext = LAContext()
        
        var localAuthenticationError: NSError?
        
        if authenticationContext.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &localAuthenticationError),
           localAuthenticationError == nil {
            
            let reason: String
            
            switch authenticationContext.biometryType {
            case .touchID:
                reason = "Log in with your Touch ID"
            case .faceID:
                reason = "Log in wiht your Face ID"
            case .none:
                completion(LocalAuthenticationError.biometricNotAvailable)
                return
            }
            
            authenticationContext.evaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, localizedReason: reason) { success, error in
                if success {
                    completion(nil)
                } else {
                    if let localError = error {
                        completion(LocalAuthenticationError.forwarded(localError))
                    } else {
                        completion(LocalAuthenticationError.unknown)
                    }
                }
                
            }
            
        } else {
            if let error = localAuthenticationError {
                completion(LocalAuthenticationError.forwarded(error))
            } else {
                completion(LocalAuthenticationError.unknown)
            }
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        authenticateWithBiomtric { authenticationError in
            
            var title = "Success"
            var message = "You logged in successfully using biometrics"
            
            if let error = authenticationError {
                title = "Failure"
                message = "Could not authenticate using biometrics. Reason: \(error.localizedDescription  ?? "unknown")"
            }
            
            DispatchQueue.main.async {
                let alertController = UIAlertController.init(title: title, message: message, preferredStyle: .alert)
                
                let dismissAction = UIAlertAction(title: "OK", style: .cancel)
                alertController.addAction(dismissAction)
                
                self.present(alertController, animated: true)
            }
        }
    }
}
```
