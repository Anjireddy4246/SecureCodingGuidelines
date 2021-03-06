# ACCESS-3: Safely invoke java.security.AccessController.doPrivileged
![Author](https://img.shields.io/badge/Author-Oracle-blue.svg)


AccessController.doPrivileged enables code to exercise its own permissions when performing SecurityManager-checked operations. For the purposes of security checks, the call stack is effectively truncated below the caller of doPrivileged. The immediate caller is included in security checks.

        +--------------------------------+
        | action                         |
        |   .run                         |
        +--------------------------------+
        | java.security.AccessController |
        |   .doPrivileged                |
        +--------------------------------+
        | SomeClass                      |
        |   .someMethod                  |
        +--------------------------------+
        | OtherClass                     |
        |  .otherMethod                  |

        +--------------------------------+
        |                                |
In the above example, the privileges of the OtherClass frame are ignored for security checks.

To avoid inadvertently performing such operations on behalf of unauthorized callers, be very careful when invoking doPrivileged using caller-provided inputs (tainted inputs):

        package xx.lib;

        import java.security.*;

        public class LibClass {
            // System property used by library, 
            //  does not contain sensitive information
            private static final String OPTIONS = "xx.lib.options";

            public static String getOptions() {
                return AccessController.doPrivileged(
                    new PrivilegedAction<String>() {
                        public String run() {
                            // this is checked by SecurityManager
                            return System.getProperty(OPTIONS);
                        }
                    }
                );
            }
        }

The implementation of getOptions properly retrieves the system property using a hardcoded value. More specifically, it does not allow the caller to influence the name of the property by passing a caller-provided (tainted) input to doPrivileged.

It is also important to ensure that privileged operations do not leak sensitive information. Whenever the return value of doPrivileged is made accessible to untrusted code, verify that the returned object does not expose sensitive information. In the above example, getOptions returns the value of a system property, but the property does not contain any sensitive data.

Caller inputs that have been validated can sometimes be safely used with doPrivileged. Typically the inputs must be restricted to a limited set of acceptable (usually hardcoded) values.

Privileged code sections should be made as small as practical in order to make comprehension of the security implications tractable.

By convention, instances of PrivilegedAction and PrivilegedExceptionAction may be made available to untrusted code, but doPrivileged must not be invoked with caller-provided actions.

The two-argument overloads of doPrivileged allow changing of privileges to that of a previous acquired context. A null context is interpreted as adding no further restrictions. Therefore, before using stored contexts, make sure that they are not null (AccessController.getContext never returns null).


        if (acc == null) {
            throw new SecurityException("Missing AccessControlContext");
        }
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    ...
                }
            }, acc);
