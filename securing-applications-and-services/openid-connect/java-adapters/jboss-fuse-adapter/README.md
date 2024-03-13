# JBoss Fuse Adapter

Currently Keycloak supports securing your web applications running inside [JBoss Fuse](http://developers.redhat.com/products/fuse/overview/).

It leverages [Jetty 9 adapter](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/jetty9-adapter.html#\_jetty9\_adapter) as JBoss Fuse 6.3.0 Rollup 1 is bundled with [Jetty 9.2 server](http://eclipse.org/jetty/) under the covers and Jetty is used for running various kinds of web applications.

| Warning | The only supported version of Fuse is JBoss Fuse 6.3.0 Rollup 1. If you use earlier versions of Fuse, it is possible that some functions will not work correctly. In particular, the [Hawtio](http://hawt.io/) integration will not work with earlier versions of Fuse. |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Security for the following items is supported for Fuse:

* Classic WAR applications deployed on Fuse with [Pax Web War Extender](https://ops4j1.jira.com/wiki/display/ops4j/Pax+Web+Extender+-+War)
* Servlets deployed on Fuse as OSGI services with [Pax Web Whiteboard Extender](https://ops4j1.jira.com/wiki/display/ops4j/Pax+Web+Extender+-+Whiteboard)
* [Apache Camel](http://camel.apache.org/) Jetty endpoints running with the [Camel Jetty](http://camel.apache.org/jetty.html) component
* [Apache CXF](http://cxf.apache.org/) endpoints running on their own separate [Jetty engine](http://cxf.apache.org/docs/jetty-configuration.html)
* [Apache CXF](http://cxf.apache.org/) endpoints running on the default engine provided by the CXF servlet
* SSH and JMX admin access
* [Hawtio administration console](http://hawt.io/)

**Securing Your Web Applications Inside Fuse**

You must first install the Keycloak Karaf feature. Next you will need to perform the steps according to the type of application you want to secure. All referenced web applications require injecting the Keycloak Jetty authenticator into the underlying Jetty server. The steps to achieve this depend on the application type. The details are described below.

The best place to start is look at Fuse demo bundled as part of Keycloak examples in directory `fuse` . Most of the steps should be understandable from testing and understanding the demo.
