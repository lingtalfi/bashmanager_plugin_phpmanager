Php manager plugin
=========================
2015-10-14




[Bash manager](https://github.com/lingtalfi/bashmanager)
provides 
[different mechanisms to foreign scripts](https://github.com/lingtalfi/bashmanager/blob/master/doc/foreign-script-guidelines.eng.md)
to communicate with the core.

However, it is a little cumbersome to deal with.

This phpManager plugin fills the gap by providing native php methods which not only emulate the bash manager core's native
functions, but allows you to use the full power of php into bash manager's scripts.



Features
-------------

- Ide autocompletion friendly
- really appropriated when you deal with a config task/action task model
- no setup: plug it in your software and enjoy







With and Without phpManager plugin comparison test
----------------------------------------

Let's compare a foreign script written WITHOUT the phpManager plugin with the same script written WITH the phpManager plugin.
  
  
### Script 'A' written WITHOUT the phpManager's plugin
  
  

```php
<?php



require_once __DIR__ . "/../phpManager.plugin/init.php";

echo 'startTask: patch2Remote' . PHP_EOL;


$req = [
    'BASH_MANAGER_CONFIG_LOCALDB_DB',
    'BASH_MANAGER_CONFIG_REMOTEDB_USER',
    'BASH_MANAGER_CONFIG_SQLPATCH_LOCATION',
    'BASH_MANAGER_CONFIG_SSH_STRING',
    'BASH_MANAGER_CONFIG_LOCALDB_PASS',
    'BASH_MANAGER_CONFIG_REMOTEDB_PASS',
    'BASH_MANAGER_CONFIG_REMOTEDB_DB',
    'BASH_MANAGER_CONFIG_LOCALDB_USER',
];


$m = [];
foreach ($req as $r) {
    if (!array_key_exists($r, $_SERVER)) {
        $m[] = $r;
    }
}


if ($m) {
    foreach ($m as $missing) {
        echo 'log:Missing configuration key: ' . $missing . PHP_EOL;
    }
}
else {
    $sshString = $_SERVER['BASH_MANAGER_CONFIG_SSH_STRING'];
    $remoteDbUser = $_SERVER['BASH_MANAGER_CONFIG_REMOTEDB_USER'];
    $sqlPatchLoc = $_SERVER['BASH_MANAGER_CONFIG_SQLPATCH_LOCATION'];
    $remoteDbPass = $_SERVER['BASH_MANAGER_CONFIG_REMOTEDB_PASS'];
    $remoteDb = $_SERVER['BASH_MANAGER_CONFIG_REMOTEDB_DB'];

    
    $sqlPatchLoc = str_replace('"', '\"', $sqlPatchLoc);
    
    $cmd = "ssh ${sshString} 'mysql -u${remoteDbUser} -p${remoteDbPass} ${remoteDb}' < \"${sqlPatchLoc}\"";
    $displayCmd = $cmd;

    if (
        array_key_exists('BASH_MANAGER_CONFIG_SECURE', $_SERVER) &&
        '1' === $_SERVER['BASH_MANAGER_CONFIG_SECURE']
    ) {
        $displayCmd = "ssh ${sshString} 'mysql -u${remoteDbUser} -pXXX ${remoteDb}' < \"${sqlPatchLoc}\"";
    }
    echo "log:executing command: $displayCmd" . PHP_EOL;
    passthru($cmd);

}


echo 'endTask: patch2Remote' . PHP_EOL;


```


### Script 'A' written WITH the phpManager plugin


```php

require_once __DIR__ . "/../phpManager.plugin/init.php";

PhpManager::create()
    ->execute('patch2Remote', function (PhpManager $o, Config $c) {


        $cmd = $o->replaceTags("ssh {sshString} 'mysql -u{remoteDbUser} -p{remoteDbPass} {remoteDb}' < \"{&sqlPatchLoc}\"");

        if ('1' === $c->secure) {
            $displayCmd = $o->replaceTags("ssh {sshString} 'mysql -u{remoteDbUser} -pXXX {remoteDb}' < \"{&sqlPatchLoc}\"");
        }
        else {
            $displayCmd = $cmd;
        }
        $o->log("executing command: $displayCmd");
        passthru($cmd);
    });


```


As we can see, the variation written with the phpManager plugin is more concise,
and provide methods that integrates better with the flow of writing bash manager tasks. 


How to install
--------------------

Download the phpManager plugin and paste the phpManager.plugin directory it into your bash manager's home, 
right next to the tasks.d and config.d directories.


Your structure should looks like this:


    - [home]
    ----- config.defaults
    ----- config.d
    ----- tasks.d
    ----- phpManager.plugin



  
How to use
-------------  

### Hello world tutorial

Create a php task (or read more about [bash manager](https://github.com/lingtalfi/bashmanager)
if you don't know how to do it).

Actually, I will tell you how (but you can still check the docs for more details).

Create the following structure:

    - [home]
    ----- phpManager.plugin
    ----- config.defaults
    ----- config.d
    --------- myconf.txt    # you have to create this one
    ----- tasks.d
    --------- hello.php    # you have to create this one


In your myconf.txt config file, put the following content:

```

# creating php task hello, 
# and assign project p1 to it with task value of haha

hello(php):
p1=haha

```


Now open your hello.php file and put the contents in it:
 
 

```php
<?php


require_once __DIR__ . "/../phpManager.plugin/init.php";



PhpManager::create()
    ->execute('hello', function (PhpManager $o, Config $c) {
        $o->log("Hello " . $o->getTaskValue());
    });

```


Ok. Now you just need to test.
Assuming that your bash manager command is accessible via bashman (see bash manager docs), you can type this:


```bash
bashman -h /path/to/home -t hello -p p1 -v
```




Show me the other phpManager methods
---------------------------------

```php
<?php


require_once __DIR__ . "/../phpManager.plugin/init.php";



PhpManager::create()
    ->execute('featuresDemoUselessTask', function (PhpManager $o, Config $c) {


        $o->error("Proxy to the bash manager core 'error' function");
        $o->log("Proxy to the bash manager core 'log' function");
        $o->warning("Proxy to the bash manager core 'warning' function");
        
        
        // access the current task value (CONFIG[BASH_MANAGER_CONFIG__VALUE])
        $currentTaskValue = $o->getTaskValue();


        /**
         * Prints the CONFIG array keys and values 
         */
        $o->printConfigEnv();


        /**
         * Will set
         *      CONFIG[BASH_MANAGER_CONFIG_MY_KEY] = 789
         */
        $o->setConfigValue('myKey', 789);
        $key = $c->myKey;

        /**
         * Will set
         *      CONFIG[BASH_MANAGER_CONFIG_MY_SECRET_PASS] = 780ZEGK4fl
         * and actually display:  
         * 
         * populating CONFIG[BASH_MANAGER_CONFIG_MY_SECRET_PASS] = (some pass)
         * 
         */
        $o->setConfigValue('mySecretPass', ["780ZEGK4fl", "(some pass)"]);


        /**
         * Will parse the given string, and replace the {tags} with their corresponding values in the CONFIG array.
         * The special tag {_value_} is replaced with the current task's value.
         * If the first symbol of the tag is an ampersand (&), like the {&_value_} tag in the example below,
         * then the value will be escaped in a manner that it fits inside double quotes inside a bash command.
         * 
         * Typically, if you have a bash command like cd "/some/path", you want to use this feature.
         * 
         * 
         */
        $cmd = $o->replaceTags("ssh {sshString} 'mysqldump -u{remoteDbUser} -p{remoteDbPass} {remoteDb}' > \"{&_value_}\"");
    });

```


What about the Config object?
--------------------------------

I almost forgot. The Config object represents the CONFIG array for the current php script being executed.
It only contains public properties and is automatically fed by the phpManager plugin, but you need to configure it first.
  
To configure the Config class, just open the file and set the names of your properties as public properties of the 
object.

For instance, if your php script uses a property named 'mySecretPass' and another called 'patchLocation',
your Config file should look like this:


```php
<?php


class Config
{
    public $mySecretPass;
    public $patchLocation;
    
}

```


And that's it.





History revisions
----------------------

- 1.02 -- 2016-02-27

    Add PhpManager.getHome sugar method
    
- 1.01 -- 2015-10-15

    Add Tool::replaceTimeStamps method
    
- 1.0 -- 2015-10-14

    Initial version





