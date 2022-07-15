---
title: "Deploying a Vault to WildFly"
visibility: public
author: matthew.casperson@octopus.com
description: "Learn how to deploy a vault from Octopus to WildFly to secure passwords in configuration files"
published: 2017-06-26
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - Product
---

When dealing with sensitive information like passwords, [Octopus Deploy provides you with the ability to encrypt and save these values](https://octopus.com/docs/projects/variables/sensitive-variables) to ensure they remain secure.

If these passwords are for external systems like database servers, it is not uncommon to have to decrypt the passwords and store them in plain text in a configuration file somewhere to allow the password to actually be used.  But saving sensitive information in plain text is not ideal, because it makes it vulnerable to a host of unsophisticated attacks like looking over someone's shoulder as they edit the configuration file, or catching the contents of the contents file as it is shared in a chat system, email or posted to a help forum, or checked into the history of a version control system.

WildFly mitigates a number of these vulnerabilities by placing sensitive information into a [vault](https://developer.jboss.org/wiki/JBossAS7SecuringPasswords).

:::hint
While a vault is considered security by obscurity, it is still worthwhile placing sensitive information into a vault as it means a casual observer of the configuration files won't be able to extract a password.
:::

As part of the move to provide first class support for Java, we are planning to provide the ability to export Octopus variables into a WildFly vault. In this blog post I'll run you through how this can be achieved today.

## Exporting Variables

If you have ever used the `vault` script that comes with WildFly, you will know that each value to be stored in the vault has to be added one at a time. This is tedious and can be hard to maintain. To provide a faster way to get secure values into a vault, we have written a [Groovy script](https://github.com/OctopusDeploy/JBossDeployment/blob/master/create-vault.groovy) that takes the values from a CSV file.

But first we need to get the secure values out of Octopus and into a CSV file. Fortunately Powershell comes with the `Export-Csv` command which makes this trivial. The code below can be defined in a script step in Octopus to export a bunch of key/value pairs to a CSV file. It then calls the `create-vault.groovy` script to turn the CSV file into a WildFly vault. The vault password is saved into an output variable called `VaultPassword`, and the CSV file is deleted.

```powershell
try {
    # Dump all available variables
    # $OctopusParameters.GetEnumerator() | sort-object Name  | Export-Csv -Path "C:\variables.csv"

    # Dump only a few variables
    $variables = @{"wildfly_slave_password" = $wildfly_slave_password}
    $variables.GetEnumerator() | Select Key,Value  | Export-Csv -Path "C:\variables.csv"

    cd C:\Apps\JBossDeployment
    $vaultPassword = &groovy create-vault.groovy `
      --keystore-file C:\wildfly_dc\wildfly-11.0.0.Alpha1\domain\configuration\keystore.vault `
      --keystore-password Password01 `
      --enc-dir C:\wildfly_dc\wildfly-11.0.0.Alpha1\domain\configuration\vault `
      --csv C:\variables.csv

    Set-OctopusVariable -name "VaultPassword" -value $vaultPassword
}
finally {
    rm "C:\variables.csv"
}
```

Once this script is run we'll have a file called `C:\wildfly_dc\wildfly-11.0.0.Alpha1\domain\configuration\keystore.vault` a directory called `C:\wildfly_dc\wildfly-11.0.0.Alpha1\domain\configuration\vault`, and a password that can be used to access the vault.

You will want to run this script on all domain nodes or standalone nodes first so they all have a local copy of the vault ready to be configured in the next step.

## Configuring the Vault

With the vault created, we now need to configure WildFly to use it. This is done by adding the following XML to the WildFly host configuration file.

```xml
<vault>
    <vault-option name="KEYSTORE_URL" value="C:\wildfly_standalone\wildfly-11.0.0.Alpha1\standalone\configuration\keystore.vault"/>
    <vault-option name="KEYSTORE_PASSWORD" value="MASK-223/wEo1GLELe8EuQa5u20"/>
    <vault-option name="KEYSTORE_ALIAS" value="vault"/>
    <vault-option name="SALT" value="12345678"/>
    <vault-option name="ITERATION_COUNT" value="50"/>
    <vault-option name="ENC_FILE_DIR" value="C:\wildfly_standalone\wildfly-11.0.0.Alpha1\standalone\configuration\vault\/"/>
</vault>
```

To make this process easy, there is another [Groovy script](https://github.com/OctopusDeploy/JBossDeployment/blob/master/deploy-vault.groovy) that will add this configuration for you.

The Powershell below is used to run the Groovy script and add the vault configuration to the hosts.

You'll only need to run this script on the domain controller, and not the domain slaves, because the domain controller will push out the vault configuration to the slaves. However, the domain controller does not push out the actual vault file, which is why we created the vault file on all hosts before adding this configuration.

If you have standalone instances, then this script will be run on all of them.

```powershell
cd C:\Apps\JBossDeployment
&groovy deploy-vault.groovy `
  --controller localhost `
  --port 9990 `
  --user admin `
  --password password `
  --keystore-file C:\wildfly_dc\wildfly-11.0.0.Alpha1\domain\configuration\keystore.vault `
  --keystore-password $OctopusParameters["Octopus.Action[Create Vault].Output.VaultPassword"] `
  --enc-dir C:\wildfly_dc\wildfly-11.0.0.Alpha1\domain\configuration\vault
```

## Accessing the Vault Passwords

Passwords contained in the vault are accessed with through a special variable in the WildFly configuration files. Here we have referenced the `wildfly_slave_password` variable from the `vault` alias in the our vault.

```xml
<server-identities>
  <secret value="${VAULT::vault::wildfly_slave_password::1}"/>
</server-identities>
```

:::hint
As you can see, the actual password is no longer saved in plain text, and even if this configuration file were leaked, it would not reveal a compromising password.
:::

You can also access these passwords from code. See [Utilising masked passwords via the vault](https://developer.jboss.org/wiki/AS7UtilisingMaskedPasswordsViaTheVault) for an example.

## Next Steps
These Groovy scripts are being developed as a proof of concept for what will eventually be migrated into steps provided directly in Octopus Deploy.

If you have any questions about the script, please leave a comment. And if there are some Java features that you would like to see Octopus Deploy  support in future, join the discussion on the [Java RFC post](https://octopus.com/blog/java-rfc).
