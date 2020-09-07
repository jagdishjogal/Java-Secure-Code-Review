# J2EE Misconfiguration

## Missing Data Transport Constraint

### Abstract

A security constraint that does not specify a user data constraint cannot guarantee that restricted resources will be protected at the transport layer.

### Explanation

web.xml security constraints are typically used for role based access control, but the optional user-data-constraint element specifies a transport guarantee that prevents content from being transmitted insecurely.

Within the ```<user-data-constraint>``` tag, the ```<transport-guarantee>``` tag defines how communication should be handled. There are three levels of transport guarantee:

1) ```NONE``` means that the application does not require any transport guarantees.
2) ```INTEGRAL``` means that the application requires that data sent between the client and server be sent in such a way that it cannot be changed in transit.
3) ```CONFIDENTIAL``` means that the application requires that data be transmitted in a fashion that prevents other entities from observing the contents of the transmission.

In most circumstances, the use of INTEGRAL or CONFIDENTIAL means that SSL/TLS is required. If the <user-data-constraint> and <transport-guarantee> tags are omitted, the transport guarantee defaults to NONE.

Example 1: The following security constraint does not specify a transport guarantee.

```xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Storefront</web-resource-name>
        <description>Allow Customers and Employees access to online store front</description>
        <url-pattern>/store/shop/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <description>Anyone</description>
        <role-name>anyone</role-name>
    </auth-constraint>
</security-constraint>
```
