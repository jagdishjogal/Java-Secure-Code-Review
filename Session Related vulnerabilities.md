# Session Puzzling: Spring

### Abstract
Attackers may modify Spring session attributes which may lead to application logic abuse.
### Explanation
A class annotated with @SessionAttributes will mean Spring replicates changes to model attributes in the session object. If an attacker is able to store arbitrary values within a model attribute, these changes will be replicated in the session object where they may be trusted by the application. If the session attribute is initialized with trusted data which the user should not be able to modify, the attacker may be able to conduct a Session Puzzling attack and abuse the application logic.

Example 1: The following controller contains a method which loads the user data into the session upon a successful login.

```java
@Controller
@SessionAttributes("user")
public class HomeController {
    ...
    @RequestMapping(value= "/auth", method=RequestMethod.POST)
    public String authHandler(@RequestParam String username, @RequestParam String password, RedirectAttributes attributes, Model model) {
        User user = userService.findByNamePassword(username, password);
        if (user == null) {
            // Handle error
            ...
        } else {
            // Handle success
            attributes.addFlashAttribute("user", user);
            return "redirect:home";
        }
    }
    ...
}
```

A different controller handles the reset password feature. It tries to load the User instance from the session since the class is annotated with @SessionAttributes("user") and uses it to verify the reset password question.

```java
@Controller
@SessionAttributes("user")
public class ResetPasswordController {

    @RequestMapping(value = "/resetQuestion", method = RequestMethod.POST)
    public String resetQuestionHandler(@RequestParam String answerReset, SessionStatus status, User user, Model model) {

        if (!user.getAnswer().equals(answerReset)) {
            // Handle error
            ...
        } else {
            // Handle success
            ...
        }
    }
}
```

The developer's intention was to load the user instance from the session where it was stored during the login process. However Spring will check the request and will try to bind its data into the model user instance. If the received request contains data that can be bound to the User class, Spring will merge the received data into the user session attribute. This scenario can be abused by submitting both an arbitrary answer in the answerReset query parameter and the same value to override the value stored in the session. This way, the attacker may set an arbitrary new password for random users.
