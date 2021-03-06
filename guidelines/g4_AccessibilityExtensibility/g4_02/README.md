# EXTEND-2: Limit the accessibility of packages
![Author](https://img.shields.io/badge/Author-Oracle-blue.svg)


Containers may hide implementation code by adding to the package.access security property. This property prevents untrusted classes from other class loaders linking and using reflection on the specified package hierarchy. Care must be taken to ensure that packages cannot be accessed by untrusted contexts before this property has been set.

This example code demonstrates how to append to the package.access security property. Note that it is not thread-safe. This code should generally only appear once in a system.

        private static final String PACKAGE_ACCESS_KEY = "package.access";
        static {
            String packageAccess = java.security.Security.getProperty(
                PACKAGE_ACCESS_KEY
            );
            java.security.Security.setProperty(
                PACKAGE_ACCESS_KEY,
                (
                    (packageAccess == null ||
                     packageAccess.trim().isEmpty()) ?
                    "" :
                    (packageAccess + ",")
                ) +
                "xx.example.product.implementation."
            );
        }
