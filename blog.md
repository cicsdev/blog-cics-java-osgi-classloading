

OSGi demystified: dynamic class loading
=======================================


Author: Ivan Hargreaves, CICS Java and Liberty Architect. Designer and Senior Developer of JVM server technologies.

*'OSGi Demystified' is a series of articles addressing common OSGi issues in CICS. We offer insight into OSGi, discuss best practices, and provide setup and configuration advice. In this article we address a thorn in the side of OSGi -- dynamic class loading. It is one of the most common causes of a
`ClassNotFoundException` and often inflicted upon you by third-party applications. If you encounter this problem, chin up! All may not be lost.*

This article covers:
-   Dynamic Class Loading explained.
-   Why Dynamic Class Loading causes problems in OSGi.

## 1. What is Dynamic Class Loading?

*Dynamic Class Loading* is the act of programmatically loading classes.
An application that loads a class at runtime based on a *textual
string*, can avoid compiling in a dependency on that class. But why
would you do that?! Hiding a dependency from the compiler and running
the risk of it not being found at runtime -- that's the craft of mad men
right? Well yes, and no.

There are distinct advantages to *dynamic class loading*. Allowing an
application to choose which classes to load could help keep it
lightweight and relevant. Determining which classes to load can be based
on configuration, or perhaps user-input. So let's say your application
needs to connect to a database, and your aim is to support many of the
common databases. With JDBC you can abstract away the specific database,
but you'll still need an appropriate JDBC driver. Typically, the user
will configure your application for their specific (favourite) database,
and your application will need to load the relevant JDBC driver from
where the user installed it. By dynamically loading the configured
driver you can avoid the overhead of loading all possible drivers, as
well as being able to load a driver that you haven't supplied or
installed with your software.

Another example of *dynamic class loading* is evident in many Java
extension mechanisms. Allowing an extender to provide additional
function (a plug-in) to your application provides greater flexibility
and customisation. A plug-in is typically configured by a text/xml file
and then dynamically loaded. As the application developer you won't know
or care what the extenders classes will be called -- but using
`Class.forName()` -- the extenders implementation can be located,
loaded, and run.

### If you play with snakes, expect to be bitten!

As powerful as *dynamic class loading is*, alarm bells should have been
ringing when we said that dependencies are hidden from the compiler. In
a normal Java environment as long as the class is somewhere on the
*single large* application class path, you only run the risk of runtime
failures if you omit the class from the class path. However, with OSGi
and its *thou must be explicit* approach, your application is much more
sensitive to hidden dependencies. There are more boundaries across which
you must declare those dependencies and more scope for mistakes because
the tooling cannot detect these dependencies.

***Hidden dependencies in
code leads straight to missing dependencies in configuration, which
leads straight to ClassNotFoundExceptions!***

### Do you have a crystal ball?

To visualise the problems caused by *dynamic class loading* in OSGi,
let's walk through some details. Each OSGi bundle has its own class
loader. The bundle's class path is essentially everything inside the
bundle, plus anything on the Bundle-ClassPath, all complimented by any
package that is explicitly referenced by the `Import-Package` statement
in the MANIFEST.MF. Now, if the code in your bundle dynamically loads a
class from another package and you don't know in advance what that
package will be (because you can't predict what will go in the
configuration file you are reading) then how do you configure your
`Import-Package` statement to include it? Without that crystal ball, you
can't.

It is equally confounding if you try to second-guess and declare all the
possible packages in advance. By declaring those dependencies, the OSGi
framework will insist that they are found AND resolved before *your*
bundle can be started. If they are not present, and by very definition
they are dynamically configured and wouldn't typically be present, your
bundle doesn't get to join the party. The choice of demise is yours.

## 2.  Enterprise OSGi Applications with *dynamism* -- OSGi services and declarative services

This section covers:
-   The many benefits of using OSGi.
-   Use OSGi services for a better dynamic component model.

There are many benefits to OSGi, including:

-   efficient and quicker class loading
-   componentisation of the application
-   the ability to run multiple versions of a package/class within the same JVM
-   component update without JVM restart
-   missing dependencies are detected at install time thus avoiding
    runtime failures

*Dynamic class loading* however, is ***not*** on its list of strengths.

The crux of the problem for OSGi is that when dependencies
(classes/packages) are not known at application deployment time, and are
instead loaded dynamically -- the OSGi framework cannot know to wire
between the relevant class loaders, and so `ClassNotFoundExceptions`
occur.

The good news is that OSGi provides a far better alternative with its
***dynamic service model***. Backed by the OSGi service repository, OSGi
service implementations declared with this model can come and go --
giving you the same benefits of *dynamic class loading*, without the
pain, and without playing class loader roulette. Not only will you reap
the aforementioned benefits, but updates and improvements will be bound
to your application dynamically. Indeed, the use of OSGi services is
generally encouraged because it provides flexible and *genuinely
dynamic* application updates. It is a common misconception of OSGi that
bundle-wiring provides the dynamic update capability -- it does not. The
OSGi service model does.

If that is not enough to persuade you to use OSGi services, then let's
talk **declarative services**. Declarative Services is a way to define
your services and the components that use them; and to have those
services automatically injected/removed as they become available.
Declarative services removes the need to write service tracking code
yourself. This is a very powerful, convenient, way to architect your
OSGi applications for the provision of dynamic updates. Ultimately OSGi
services provide a far cleaner solution and offer a wealth of benefits
beyond any class loader based approach.

###### Further reading:

- [OSGi Services](http://moi.vonos.net/java/osgi-services/)
- [Declarative Services](https://eclipsesource.com/blogs/2009/05/12/osgi-declarative-services/)
- [Getting Started with Declarative Services](http://blog.vogella.com/2016/06/21/getting-started-with-osgi-declarative-services/)
- [OSGi Dependency Injection](https://dzone.com/articles/osgi-dependency-injection)


## 3. Workarounds for exploiting OSGi Services effectively

This section includes:
-   Real world reasons why you might not be able to exploit OSGi services.
- Potential workarounds for your coding restrictions.
    services.

### A spanner in the works -- exploiting OSGi Services

Despite all the benefits, there are real world reasons why you may not
be able to exploit OSGi services. If you have third party code outside
of your control, or if re-structuring an existing app is above your
pay-grade, then you won't be able to make the necessary code changes.
For those unfortunate enough to have their shoe-laces tied in these
matters, there are workarounds -- but expect to be compromised. The
following approaches are potential workarounds -- in the absence of
better application architecture.

#### i) Swap out third-party JARs for third-party bundles

On the face of it, this is a no-brainer. Many vendors and open-source
communities offer OSGi aware versions of their libraries. Sometimes it
can be as simple as tracking down an OSGi friendly bundled version of
the library and swapping the old JAR for the equivalent bundle.

#### ii) DynamicImport-Package

If you know the dynamically loaded classes will belong to a specific
package, you can use `DynamicImport-Package: com.mycompany.package` in
the MANIFEST.MF of the bundle doing the class loading. This allows OSGi
to look up the classes to load on-the-fly, but it also negates some of
the benefits of using OSGi. If the problem is confined to a very small
number of stable classes this may be a pragmatic solution to keep
disruption minimal, but use with care.

If you don't know the package from which classes will be dynamically
loaded, you could use the more generic `DynamicImport-Package: *"`.
Doing so effectively turns your OSGi framework into one big classpath,
negates all the benefits of OSGi, and causes potentially CPU intensive
operations every-time the bundle attempts to load classes. It is to be
**avoided** where possible!

Put simply, DynamicImport-Package and buddy class loading are not a
viable alternative -- if you browse the web you find lots of well
meaning people pointing to these "solutions". Unfortunately, these
"solutions" move you right back all the problems of the class path.

Further reading:
[Use of 'DynamicImport-Package: \*' in
OSGi](http://keheliya.blogspot.co.uk/2013/02/use-of-dynamicimport-package-in-osgi.html)

#### iii) Thread Context Class Loader (TCCL)

Another potential solution might be to set the Thread context class
loader to the bundle's class loader. Many legacy and/or third-party
libraries attempt to load application classes by name at runtime: for
example, [Hibernate](http://hibernate.org/) reads class names from
hbm.xml files and then creates instances of those classes for each
database record. When you move such a library to a modular environment,
it breaks -- the name of a class is not sufficient to uniquely identify
it. The identity of a class consists of its fully qualified name AND the
class loader that defined it, which in OSGi usually corresponds to the
bundle that contains it. So, in addition to the name, we need to know
the class loader from which to load the class. Due to the wide variety
of class loading environments created by various application servers,
many libraries attempt to solve this problem with a set of heuristics.
Consulting the TCCL is usually one of the heuristics, along with
checking the library's own class loader or the JRE extension class
loader, etc.

If you want to gamble and take the TCCL heuristic approach, you would do
something like this:
```
• save away the current TCCL
• set the current TCCL to be the class loader of the bundle
• call the third-party code which attempts to load the class
• hope you get lucky!
```
Don't forget to set the TCCL back to the original on the way out. Here's
the pseudo-code:
```
    ClassLoader tccl = Thread.currentThread().getContextClassLoader();Thread.currentThread().setContextClassLoader(this.getClass().getClassLoader());
    try
    {
        // execute third-party code
    }
    finally
    {
        Thread.currentThread().setContextClassLoader(tccl);
    }
```
 
Further reading:
[What You Should Know about Class
Loaders](https://blog.osgi.org/2011/05/what-you-should-know-about-class.html)

#### vi) Leverage the OSGi frameworks PackageAdmin utilities

Rather than using Class.forName() and hoping your current class loader
has the right visibility to the target package/class, you can invoke the
OSGi packageAdmin class. By supplying the package name from which your
class is exported, you let it do the donkey work. If there is a bundle
in the system that exports the class you require from the intended
package, this convenience call finds and loads the class on your behalf.
For example:

`Class clazz = packageAdmin.getExportedPackage(packageName).getExportingBundle().loadClass(className);`

#### vii) Centralize

If you really need class loaders, then centralize and isolate your code
in a different package. Make sure your class loading strategies are not
visible in your application code. Module systems will provide an API
that allows you to correctly load classes from the appropriate module.
So when you move to a module system you only have to port one piece of
code and not change your whole code base.

## Conclusion

This article has discussed dynamic class loading, the
problems it causes, and how to achieve the same flexibility with OSGi
services without the pain. We've also provided a set of real-world
workarounds for convenience, but these are no substitute for
well-designed applications.
