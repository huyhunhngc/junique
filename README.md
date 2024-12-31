# JUnique Library Overview

The JUnique library can perform cross-JVM lock operations. It is mainly intended to prevent a user from running multiple instances of the same Java application simultaneously. It also offers a communication layer enabling message exchange between different JVMs.

JUnique is based on a *user-related cross-JVM exclusive lock* concept. Suppose you want to prevent your Java desktop application from being started more than once by the same user. When the `main()` method of your application is called, you can require JUnique to lock a certain ID, e.g., *myapplicationid*. On the first application launch, that ID will be available, and the lock will be taken. The obtained lock can be explicitly released with a `JUnique.releaseLock()` call; otherwise, it will be automatically released when the JVM halts. If another JVM instance (or even the same one) tries to acquire a lock on the same ID, JUnique will throw an `AlreadyLockedException`, notifying the instance that another one is currently running. The second instance can then choose to abort its start sequence and optionally send messages to the active one before exiting.

Please note that a JUnique lock is user-related: two different users accessing the system at the same time can still run two separate instances of the same application since JUnique IDs reside in a user scope.

---

## Quickstart

### Preventing Multiple Running Instances
A sample `main()` method preventing multiple running instances of the same application for the same user:

```java
public static void main(String[] args) {
    String appId = "myapplicationid";
    boolean alreadyRunning;
    try {
        JUnique.acquireLock(appId);
        alreadyRunning = false;
    } catch (AlreadyLockedException e) {
        alreadyRunning = true;
    }
    if (!alreadyRunning) {
        // Start sequence here
    }
}
```

### Message Handling Example

A message handling example, transferring arguments from the aborting instance to the active one:

```java
public static void main(String[] args) {
    String appId = "myapplicationid";
    boolean alreadyRunning;
    try {
        JUnique.acquireLock(appId, new MessageHandler() {
            public String handle(String message) {
                // A brand new argument received! Handle it!
                return null;
            }
        });
        alreadyRunning = false;
    } catch (AlreadyLockedException e) {
        alreadyRunning = true;
    }
    if (!alreadyRunning) {
        // Start sequence here
    } else {
        for (int i = 0; i < args.length; i++) {
            JUnique.sendMessage(appId, args[i]);
        }
    }
}

```
### Best Practices for Lock ID

1. **Use Unique IDs**: Always use a unique and descriptive ID for your application to avoid conflicts. Generic IDs are prone to collisions, especially in environments with multiple applications using JUnique.
   
2. **Namespace Your ID**: Prefix your lock ID with a namespace related to your application or organization. For example, if your application is `MyApp` and your organization is `MyCompany`, an ideal lock ID could be `com.mycompany.myapp`.

3. **Match ID to Main Class**: A straightforward strategy is to use your application's main class name as the lock ID. This ensures the ID is both unique and easily identifiable.

---

## Additional Notes

- JUnique locks are **user-specific**. If two users are logged into the same machine, they can run separate instances of the same application without interference.
- Proper exception handling is crucial to gracefully manage situations where a lock is already taken. Plan your application flow to handle these scenarios effectively.
- For advanced use cases, JUnique's message-passing capabilities can facilitate seamless communication between multiple JVM instances.

---

For further details, refer to the JUnique documentation and explore the examples provided in the distribution archive.
