# Session Related vulnerabilities
## Session Puzzling: Spring
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

## Session Fixation
### Abstract
Authenticating a user without invalidating any existing session identifier gives an attacker the opportunity to steal authenticated sessions
### Explanation
Session fixation vulnerabilities occur when:

1. A web application authenticates a user without first invalidating the existing session, thereby continuing to use the session already associated with the user.
2. An attacker can force a known session identifier on a user so that, after the user authenticates, the attacker has access to the authenticated session.

In the generic exploit of session fixation vulnerabilities, an attacker creates a new session on a web application and records the associated session identifier. The attacker then causes the victim to authenticate against the server using that session identifier, giving the attacker access to the user's account through the active session.

Some frameworks such as Spring Security automatically invalidates existing sessions when creating a new one. This behaviour can be disabled leaving the application vulnerable to this attack.

Example 1: The following example shows a snippet of a Spring Security protected application where session fixation protection has been disabled.

```xml
	<http auto-config="true">
        ...
        <session-management session-fixation-protection="none"/>
	</http>
```

Even given a vulnerable application, the success of the specific attack described here depends on several factors working in the attacker's favor: access to an unmonitored public terminal, the ability to keep the compromised session active, and a victim interested in logging into the vulnerable application on the public terminal. In most circumstances, the first two challenges are surmountable given a sufficient investment of time. Finding a victim who is both using a public terminal and interested in logging into the vulnerable application is possible as well, as long as the site is reasonably popular. The less popular the site, the lower the odds of an interested victim using the public terminal and the less chance of success for the attack vector previously described.

The biggest challenge an attacker faces in exploiting session fixation vulnerabilities is inducing victims to authenticate against the vulnerable application using a session identifier known to the attacker. In Example 1, the attacker does this through an obvious direct method that does not suitably scale for attacks involving less well-known web sites. However, do not be lulled into complacency; attackers have many tools in their belts that help bypass the limitations of this attack vector. The most common technique attackers use involves taking advantage of cross-site scripting or HTTP response splitting vulnerabilities in the target site [1]. By tricking the victim into submitting a malicious request to a vulnerable application that reflects JavaScript or other code back to the victim's browser, an attacker can create a cookie that causes the victim to reuse a session identifier controlled by the attacker.

It is worth noting that cookies are often tied to the top level domain associated with a given URL. If multiple applications reside on the same top level domain, such as bank.example.com and recipes.example.com, a vulnerability in one application can enable an attacker to set a cookie with a fixed session identifier that is used in all interactions with any application on the domain example.com [2].

Other attack vectors include DNS poisoning and related network-based attacks where an attacker causes the user to visit a malicious site by redirecting a request for a valid site. Network-based attacks typically involve a physical presence on the victim's network or control of a compromised machine on the network, which makes them harder to exploit remotely, but their significance should not be overlooked. Less secure session management mechanisms, such as the default implementation in Apache Tomcat, allow session identifiers normally expected in a cookie to be specified on the URL as well. This enables an attacker to cause a victim to use a fixed session identifier simply by emailing a malicious URL.
