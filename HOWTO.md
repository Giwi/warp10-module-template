# Warp 10™ Modules

Warp 10™ is very modular, you can augment or modify the features of a Warp 10™ deployment through the use of several mechanisms commonly called **modules**.

The four types of modules are:

* WarpScript™ Extensions (`extensions`)
* Warp 10™ Plugins (`plugins`)
* Warp 10™ Authentication Plugins (`authplugins`)
* Macro Packages (`packages`)

All those module types can be created using this template.

Modules can be private, *i.e.* their use is limited to your organization, or public. For public modules, [SenX](https://senx.io/) manages a directory called WarpFleet which describes modules which were made public by their authors.

# Extension

WarpScript™ [extensions](https://warp10.io/content/03_Documentation/07_Extending_Warp_10/03_Extensions) add, remove or modify functions.

## Authoring an extension

An extension is a Java class which declares WarpScript™ functions, each function being itself a Java class.

The skeleton of the extension Java class is

```
import io.warp10.warp.sdk.WarpScriptExtension;

import java.util.HashMap;
import java.util.Map;

/**
 * Functions declared by this extension
 * must be present in the 'functions' field.
 */
public class MyExtension extends WarpScriptExtension {

  private static final Map<String, Object> functions;

  static {
    functions = new HashMap<String, Object>();

    //
    // Declare the functions in the 'functions' map
    //
    // The key is name of the WarpScript™ function
    // The value is an instance of the associated Java class
    //

    functions.put("MYFUNC", new MYFUNC("MYFUNC"));
  }

  public Map<String, Object> getFunctions() {
    return functions;
  }
}
```

Each function is a Java class with the following skeleton:

```
package ext;

import io.warp10.script.NamedWarpScriptFunction;
import io.warp10.script.WarpScriptException;
import io.warp10.script.WarpScriptStack;
import io.warp10.script.WarpScriptStackFunction;

public class MYFUNC extends NamedWarpScriptFunction implements WarpScriptStackFunction {

  public MYFUNC(String name) {
    super(name);
  }

  /**
   * The 'apply' method is where your function logic is implemented
   *
   * @param stack The stack from which your function was called
   * @return stack The original stack
   * @throws WarpScriptException If errors are encountered
   */
  public Object apply(WarpScriptStack stack) throws WarpScriptException {
    
    //
    // You function logic goes here
    //

    return stack;
  }
}
```

## Building the extension

The `jar` and `shadowJar` gradle tasks will build the extension and package it in a simple or über (with all dependencies) jar.

## Using an extension

Extensions must be deployed in the classpath of the Warp 10™ instance, typically by placing their `.jar` file in the `lib` directory of the Warp 10™ installation. We recommend you use the über jar as you do not have to worry about copying the dependencies jar files as they are already included.

The extension must then be declared in the configuration file of the Warp 10™ instance using the following syntax:

```
warpscript.extension.NAME = your.extension.package.ANDCLASS
```

When the Warp 10™ instance is restarted, the functions declared by the extension will be available in WarpScript™.

# Plugin

A [plugin]() is a Java class which can add features to a Warp 10™ instance. The plugin mechanism is very flexible, so the features that can be added are really anything a Java class can do, from launching background threads to listening on a port using a particular protocol, your imagination is the only limit to what plugins can do.

## Authoring a plugin

A plugin a a simple Java class with the following skeleton:

```
import java.util.Properties;

import io.warp10.warp.sdk.AbstractWarp10Plugin;

public class TemplateWarp10Plugin extends AbstractWarp10Plugin {
  @Override
  public void init(Properties properties) {
    //
    // Your plugin initialization logic goes here
    //
  }
}
```

The `init` method is called when the plugin is registered by the Warp 10™ instance.

## Building the plugin

The `jar` and `shadowJar` gradle tasks will build the extension and package it in a simple or über (with all dependencies) jar.

## Deploying the plugin

The jar file of the plugin must be avalable in the classpath of the Warp 10™ instance, typically by placing it in the `lib` directory of the Warp 10™ installation. We recommend you use the über jar as you do not have to worry about copying the dependencies jar files as they are already included.

The plugin must then be declared in the configuration file of the Warp 10™ instance using the following syntax:

```
warpscript.plugin.NAME = your.plugin.package.ANDCLASS
```

When the Warp 10™ instance is restarted, the plugin `init` function will be called.

# Authentication Plugin

[Authentication plugins](https://www.warp10.io/content/03_Documentation/05_Security/06_Auth_plugins#registering-an-authentication-plugin) are Warp 10™ plugin Java classes which also implement the `io.warp10.continuum.AuthenticationPlugin` interface.

As part of their `init` method, those plugins must register themselves using 

```
Tokens.register(this)
```

# Macro Package


# WarpFleet integration

If you intend to reference your extension in WarpFleet, you need to publish it on a Maven repository.

The `build.gradle` included in this template contains the definition of two Maven publications, `stdjar` and `uberjar` which will publish your extension either as a simple `.jar` with dependencies listed in the accompanying `.pom` file, or as an ûber jar with no external dependencies.

## Publishing to a maven repository

The `publish` task will publish the `stdjar` or `uberjar` (if `-Duberjar` is specified) publications to the declared maven repository (defaults to the local one).

To use a specific repo, add it to the `publishing` section in the `build.gradle` file. The syntax is describe [here](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:repositories).

## Publishing on bintray

The `build.gradle` contains a task to publish the jars on [bintray](), simply execute

```
./gradlew bintrayUpload
```

for uploading the simple jar, or

```
./gradlew -Duberjar bintrayUpload
```

to upload the über jar.

Bintray user, API key and optional organization must be specified in a `gradle.properties`. Please refer to the beginning of the `build.gradle` file to learn what properties should be defined.
