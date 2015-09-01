The first NuGet package for ELMAH (based on version 1.1) was prepared by the NuGet team and is in use in numerous projects. The package not only came with the ELMAH assembly but also some configuration that got people up and running quickly.

NuGet's configuration transformation support is great for first-time installs but [somewhat lacking](http://nuget.codeplex.com/workitem/232) for migration and upgrade scenarios. As a result, if you have the ELMAH 1.1 package installed, your existing configuration for ELMAH may get lost or corrupted (especially duplicated entires) if you upgrade to the newer 1.2 packages.

It is **strongly advised that you backup your configuration** before upgrading (or switching) packages and then use a diff/merge tool to recover or resolve conficts in your configuration after making the upgrade/switch.

The new ELMAH 1.2 packages come in two flavors: _Base Packages_ and _Quick Start Packages_.

The base packages only adds the assemblies without any configuration. You are required to setup the configuration yourself, leaving you in full control. These packages are therefore designed to avoid loss or corruption of configuration.

The quick start packages build on top of the base packages by adding a basic amount of configuration to get you up and started quickly (much like the initial ELMAH 1.1 package released by the NuGet team). They are, in principal, designed for demo and prototype applications.