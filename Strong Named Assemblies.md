Starting in v3.2, most of the Insight.Database assemblies are now signed. That means you can reference them in your own strong named assemblies.

NOTE: since the project is open-source, I'll be keeping the key file with the code so other people can build it. Therefore, the strong name won't protect you against someone replacing the assembly with a forged version.

However, after I make the change, it would be easy to sign the assemblies yourself. Just fork the repository, replace Insight.Database.snk, and build the package yourself.