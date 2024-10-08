# Time-Based Policy

You can use this type of policy to define time conditions for your permissions.

To create a new time-based policy, select **Time** in the dropdown list in the upper right corner of the policy listing.

Add Time Policy

![Add Time Policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/policy/create-time.png)

**Configuration**

*   **Name**

    A human-readable and unique string describing the policy. A best practice is to use names that are closely related to your business and security requirements, so you can identify them more easily.
*   **Description**

    A string containing details about this policy.
*   **Not Before**

    Defines the time before which access must **not** be granted. Permission is granted only if the current date/time is later than or equal to this value.
*   **Not On or After**

    Defines the time after which access must **not** be granted. Permission is granted only if the current date/time is earlier than or equal to this value.
*   **Day of Month**

    Defines the day of month that access must be granted. You can also specify a range of dates. In this case, permission is granted only if the current day of the month is between or equal to the two values specified.
*   **Month**

    Defines the month that access must be granted. You can also specify a range of months. In this case, permission is granted only if the current month is between or equal to the two values specified.
*   **Year**

    Defines the year that access must be granted. You can also specify a range of years. In this case, permission is granted only if the current year is between or equal to the two values specified.
*   **Hour**

    Defines the hour that access must be granted. You can also specify a range of hours. In this case, permission is granted only if current hour is between or equal to the two values specified.
*   **Minute**

    Defines the minute that access must be granted. You can also specify a range of minutes. In this case, permission is granted only if the current minute is between or equal to the two values specified.
*   **Logic**

    The [Logic](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/logic.html#\_policy\_logic) of this policy to apply after the other conditions have been evaluated.

Access is only granted if all conditions are satisfied. Keycloak will perform an _AND_ based on the outcome of each condition.

\
