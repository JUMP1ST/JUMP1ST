# Guide Overview

The purpose of this guide is to walk through the steps that need to be completed prior to booting up the Keycloak server for the first time. If you just want to test drive Keycloak, it pretty much runs out of the box with its own embedded and local-only database. For actual deployments that are going to be run in production you’ll need to decide how you want to manage server configuration at runtime (standalone or domain mode), configure a shared database for Keycloak storage, set up encryption and HTTPS, and finally set up Keycloak to run in a cluster. This guide walks through each and every aspect of any pre-boot decisions and setup you must do prior to deploying the server.

One thing to particularly note is that Keycloak is derived from the Wildfly Application Server. Many aspects of configuring Keycloak revolve around Wildfly configuration elements. Often this guide will direct you to documentation outside of the manual if you want to dive into more detail.
