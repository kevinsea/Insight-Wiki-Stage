## New Features ##

* Insight can now bind parameters (in/out) and TVP columns to members of subobjects. See [Customizing Object Mapping].
* New configuration system for controlling mapping and serialization. See [Customizing Object Mapping] and [Object Serialization].
* Support for Composite keys in parent/child relationships. See [Specifying Result Structures].


## Upgrading from v4.0 to v5.0 ##

Most of your code should just work.

Known changes:

* If you were using custom column/parameter mapping, you will have to switch to the new configuration system. See [Customizing Object Mapping].
* If you customized the serialization rules, you will have to switch to the new configuration system. See [Object Serialization].