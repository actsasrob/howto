# Puppet 4

## Common Directories

Puppet is now installed using a new All-in-One (AIO) puppet-agent package
This AIO package includes private versions of Facter, Hiera, MCollective, and
Ruby.

Executables are in /opt/puppetlabs/bin/
All executables have been moved to /opt/puppetlabs/puppet/bin. A subset of exe‐
cutables has symlinks in /opt/puppetlabs/bin, which has been added to your path.
(You may need to restart your shell.)

A private copy of Ruby is installed in /opt/puppetlabs/puppet
Ruby and supporting commands such as gem are installed in /opt/puppetlabs/
puppet, to avoid them being accidentally called by users.

The configuration directory stored in $confdir is now /etc/puppetlabs/puppet
Puppet Open Source now uses the same configuration directory as Puppet Enter‐
prise. Files in /etc/puppet will be ignored.

The $ssldir directory is inside $confdir
On many platforms, Puppet previously put TLS keys and certificates in /var/lib/
puppet/ssl. With Puppet 4, TLS files will be installed inside the confdir/ssl/ direc‐ tory on all platforms.

The MCollective configuration directory is now /etc/puppetlabs/mcollective
Files in /etc/mcollective will be checked if the former directory doesn’t exist.

The $vardir for Puppet is now /opt/puppetlabs/puppet/cache/
This new directory is used only by puppet agent and puppet apply . You can
change this by setting $vardir in the Puppet config file.

The $rundir for Puppet agent is now /var/run/puppetlabs
This directory stores PID files only. Changing this directory will cause difficulty
with the installed systemd and init service scripts.

Modules, manifests, and the Hiera config file have a new directory: /etc/puppetlabs/code
Puppet code and data has moved from $confdir to a new directory $codedir .
This directory contains:
* The environments directory for $environmentpath
* The modules directory for $basemodulepath
* The hiera.yaml config file for $hiera_config

Useful command to display directory locations:
sudo /opt/puppetlabs/bin/puppet config print |grep dir

## Writing Manifests

A manifest is a file containing Puppet configuration language that describes how resources should be configured. The manifest is the closest thing to what one might consider a Puppet program. It uses resources to define a policy to be enforced on a node. It is therefore the base component for Puppet configuration policy, and a building block for complex Puppet modules.

## Writing

Resources are the smallest building block of the Puppet configuration language. They
represent a singular element that you wish to evaluate, create, or remove. Puppet
comes with many built-in resource types.

Example notify resouce:
```
notify { 'greeting':
   message => 'Hello, world!'
}
```

This code declares a notify resource with a title of greeting . It has a single attribute, message , which is assigned the value we’d expect in our first program. Attributes are separated from their values by a hash rocket (also called a fat comma), which is a very common way to identify key/value pairs in Perl, Ruby, and PHP scripting languages.

## Applying a Manifest

Example applying a local manifest file using 'puppet apply' command. The example assumes the above content is stored in a file name helloworld.pp:

    puppet apply helloworld.pp

    Notice: Compiled catalog for puppetagent1 in environment production in 0.07 seconds
    Notice: Hello, world!
    Notice: /Stage[main]/Main/Notify[greeting]/message: defined 'message' as 'Hello, world!'
    Notice: Applied catalog in 0.02 seconds

There are only a few rules to remember when declaring resources. The format is
always the same:
```
resource_type { 'resource_title':
ensure => present, 	# usually 'present' or 'absent'
attribute1 => 1234,     # number
attribute2 => 'astring' 
attribute3 => [ 'arrayentry1', 'arrayentry2'],
noop => false, 		# boolean
}
````

The most important rule for resources is: there can be only one. Within a manifest or set of manifests being applied together (the catalog for a node), a resource of a given type can only be declared once with a given title. Every resource of that type must have a unique title.

## Viewing Resources

Puppet can show you an existing resource written out in the Puppet configuration
language. This makes it easy to generate code based on existing configurations.

    puppet resource mailalias postmaster

The puppet resource command queries the node using the exact same code used by
Puppet to compare and alter the system state. The output gives you the exact struc‐
ture, syntax, and attributes to declare this alias resource in a Puppet manifest.

Need to use 'sudo' if accessing sensitive data like password hashes in /etc/passwd:
    sudo puppet resource user sshd

## Executing Programs

You can use the 'exec' resource to execute programs. But it is considered bad form and should only be used when other more appropriate resources will not work.


## Managing Files

```
file { '/tmp/testfile.txt':
  ensure => present,
  mode => '0644',
  replace => true,
  content => 'holy cow!\n',
}
```

See this link for all file resource attributes: https://docs.puppet.com/puppet/latest/type.html#file


Finding File Backups
Every file changed by a Puppet file resource is backed up on the node in a directory specified by the $clientbucketdir configuration setting. Unfortunately, the file backups are stored in directories indexed by a hash of the file contents, which makes them quite tricky to find.  

### Backup a file

    sudo puppet filebucket --local backup /tmp/testfile.txt

Use this command to get a list of every file backed up.
    sudo puppet filebucket --local list

### Restoring Files

Use the hash associated with a specific file version to view the contents of the file at that point in time:
    sudo puppet filebucket --local get 0eb429526e5e170cd9ed4f84c24e442b


Youu can compare an installed file to a backup version, or compare two backup ver‐ sions, using the filebucket diff command:
    sudo puppet filebucket --local diff  0eb429526e5e170cd9ed4f84c24e442b /tmp/testfile.txt

You can restore a backup file to the original location or another path:
    sudo puppet filebucket --local restore /tmp/testfile.txt 0eb429526e5e170cd9ed4f84c24e442b

# Using the Puppet Configuration Language

The section covers the data types, operators, conditionals, and iterations that can be used to build manifests for Puppet 4.

## Defining Variables

Puppet makes use of variables (named storage of data) much like any other language you’ve learned to write code in.  Like many scripting languages, variables are prefaced with a $ in Puppet. The variable name must start with a lowercase letter or underscore, and may contain lowercase letters, numbers, and underscores.

e.g.
```
$my_name	= 'Rob'	# string
$my_age		= 29	# numeric
$a_boolean	= false	# boolean
```

*NOTE: Previous versions of Puppet allowed uppercase letters, periods, and dashes with inconsistent results. Puppet 4 has improved reliability by enforcing these standards.*
*NOTE: Variables starting with underscores should only be used within the local scope*

A variable that has not been initialized will evaluate as undefined, or undef . You can also explicitly assign undef to a variable:
    $notdefined = undef

The puppetlabs/stdlib module provides a function you can use to see a variable’s type:
```
include stdlib
$nametype = type_of( $my_name )
$numtype = type_of( $my_age )
```

## Defining Numbers
In Puppet 4, unquoted numerals are evaluated as a Numeric data type. Numbers are assigned specific numeric types based on the characters at the start of or within the number:
* Decimal numbers start with 1 through 9.
* Floating-point numbers contain a single period.
* Octal numbers (most commonly used for file modes) start with a 0 .
* Hexadecimal numbers (used for memory locations or colors) start with 0x .

*NOTE: In previous versions of Puppet, bare numbers were evaluated as strings. Best practice as of Puppet 3 was to quote all numbers to ensure they were evaluated as a String . This was preparation for Puppet 4, where unquoted numbers are the Numeric data type, and validation is performed on the value.*

Any time an unquoted word starts with numbers, it will be validated as a Numeric type.

Always quote numbers that need to be represented intact, such as decimals with leading zeros.
Creating Arrays and Hashes
It is possible to declare an Array (ordered list) that contains many values. As I’m sure
you’ve used arrays in other languages, we’ll jump straight to some examples:
$my_list	= [1,3,5,7,11]
$my_names	= ['Amy','Sam','Jen']
$mixed_up	= ['Alice',3,true]
$trailing	= [4,5,6,]
$embedded	= [4,5,['a','b']]

Some functions require a list of input values, instead of an array. In a function call, the splat operator ( * ) operator will convert an Array into a comma-separated list of values:
    myfunction( *$Array_of_arguments ) { ... }

## Mapping Hash Keys and Values
You can also create an unordered, random-access hash where member values are associated with a key value. At assignment time, the key and value should be separated by a hash rocket ( => ), as shown here:
```
# small hash of user home directories
$homes = { 'Jack' => '/home/jack', 'Jill' => '/home/jill', }
```

## Using Variables in Strings
Strings with pure data should be surrounded by single quotes:
```
$my_name = 'Dr. Evil'
$how_much = '100 million'
```

Use double quotes when interpolating variables into strings, as shown here:
    notice( "Hello ${username}, glad to see you today!" )

## Using Braces to Limit Problems
As with most scripting languages, curly braces should be used to delineate variable boundaries:
```
$the_greeting = "Hello ${myname}, you have received ${num_tokens} tokens!"
notice( "The second value in the list is ${my_list[1]}" )
```

For example the following array index will not work:
    notice( "The second value in the list is $my_list[1]" )

## Preventing Interpolation

In most cases, you can simply preface the character with the escape character (a back slash or \ ) to avoid interpolation. Interpolation happens only once, even if a string is used within another string,

Using single quotes avoids the need for backslashes:
```
$num_tokens = '$100 million'
$cifs_share = '\\server\drive'
```

## Avoiding Redefinition
Variables may not be redefined in Puppet within a given namespace or scope. We’ll cover the intricacies of scope in Part II, “Creating Puppet Modules”, but understand that a manifest has a single namespace, and a variable cannot receive a new value within that namespace.


Puppet language reference for variables: https://docs.puppet.com/puppet/latest/lang_variables.html

Puppet language reference for Values and Data Types: https://docs.puppet.com/puppet/latest/lang_data.html

## Finding Facts

Facter provides many variables for you containing node- specific information. These are always available for use in your manifests.

```
facter
architecture => x86_64
augeasversion => 1.0.0
blockdevice_sda_model => VBOX HARDDISK
blockdevice_sda_size => 10632560640
blockdevice_sda_vendor => ATA
blockdevices => sda
domain => example.com
<snip>
```

Puppet adds several facts for use within Puppet modules. You can run the following command to see all facts used by Puppet, and installed on the node from Puppet modules:
    facter --puppet

Puppet always adds the following facts above and beyond system facts provided by Facter:
    $facts['clientcert']

The client-reported value of the node’s certname configuration value.
    $facts['clientversion']

The client-reported version of Puppet running on the node.
    $facts['clientnoop']
The client-reported value of whether noop was enabled in the configuration or on the command line to perform the comparison without actually making changes.
    $facts['agent_specified_environment']
The environment requested by the client. When using a Puppet server, the server could override the environment selection. This value contains the requested value. If this value is blank, the production environment is used by default.  All of these facts can be found in the $facts hash.

e.g.
```
puppet apply -e 'notice("blah ${facts['clientcert']}")'
Notice: Scope(Class[main]): blah puppetagent1
Notice: Compiled catalog for puppetagent1 in environment production in 0.06 seconds
Notice: Applied catalog in 0.02 seconds
```

You can also use Puppet to list out Puppet facts in JSON format:
    puppet facts find

Facter can provide the data in different formats, useful for passing to other programs.  The following options output Facter data in the common YAML and JSON formats:
```
facter --yaml
puppet facts --render-as yaml
facter --json
puppet facts --render-as json
```

## Calling Functions in Manifests
A function is executable code that may accept input parameters, and may output a return value. A function that returns a value can be used to provide a value to a variable:

    $zero_or_one = bool2num( $facts['is_virtual'] );
 
The function can also be used in place of a value, or interpolated into a string:
```
md5() function provides the value for the message attribute
notify { 'md5_hash':
   message => md5( $facts['fqdn'] )
}
Include the MD5 hash in the result string
$result = "The MD5 hash for the node name is ${md5( $facts['fqdn'] )}"
```
Functions can also take action without returning a value. Previous examples used the notice() function, which sends a message at notice log level but does not return a
value. In fact, there is a function for logging a message at each level, including:
* debug( message )
* info( message )
* notice( message )
* warning( message )
* err( message )

Puppet executes functions when building the catalog; thus, functions can change the catalog. Some of the more common uses for this level of power are:
* Look up data from external sources
* Add, modify, or remove items from the catalog
* Dynamically generate and execute code segments

Functions can be written in the common prefix format or in the Ruby-style chained format. The following two calls will return the same result:
```
Common prefix format
notice( 'this' )

Ruby-style chained format
'this'.notice()
```

## Calling Functions in Manifests
A function is executable code that may accept input parameters, and may output a return value. A function that returns a value can be used to provide a value to a variable:

    $zero_or_one = bool2num( $facts['is_virtual'] );

The function can also be used in place of a value, or interpolated into a string:
```
md5() function provides the value for the message attribute
notify { 'md5_hash':
  message => md5( $facts['fqdn'] )
}
Include the MD5 hash in the result string
$result = "The MD5 hash for the node name is ${md5( $facts['fqdn'] )}"
```


Puppet executes functions when building the catalog; thus, functions can change the catalog. Some of the more common uses for this level of power are:
* Look up data from external sources
* Add, modify, or remove items from the catalog
* Dynamically generate and execute code segments

## Using Variables in Resources
Now let’s cover how to use variables in resources. Each data type has different methods, and sometimes different rules, about how to access its values.

Constant strings without variables in them should be surrounded by single quotes.  Strings containing variables to be interpolated should be surrounded by double quotes. No other type should be quoted. Here’s an example:

You can access specific items within an Array by using a 0-based array index within
square brackets. Two indices can be specified to retrieve a range of items:
```
$first_item = $my_list[1]
$four_items = $my_list[3,6]
```

You can access specific items within a Hash by using the hash key within square brackets as follows:
    $username = $my_hash['username']


Use curly braces when interpolating variables into a double-quoted string. The curly braces must surround both the variable name and the index or key (within square brackets) when accessing hash keys or array indexes:
```
notice( "The user's name is ${username}" )
notice( "The second value in my list is ${my_list[1]}" )
notice( "The login username is ${my_hash['username']}" )
```
Retrieve specific values from a Hash by assigning to an Array of variables named for the keys you’d like to retrieve. Read that sentence carefully—the name of the variable in the array identifies the hash key to get the value from:
    [$Jack] = $homes	 # identical to $Jack = $homes['Jack']
    [$username,$uid] = $user # gets the values assigned to keys "username" and "uid"

The facts provided by Facter can be referenced like any other variable. The facts are available in a $facts hash. For example, to customize the message shown on login to each node, use a file resource like this:
```
file { '/etc/motd':
  ensure => present,
  mode => '0644',
  replace => true,
  content => "${facts['hostname']} runs ${facts['os']['release']['full']}",
}
```

You can receive Evaluation Error exceptions when Puppet tries to use a variable that has never been defined by enabling the strict_variables configuration setting in /etc/puppetlabs/puppet/puppet.conf:
```
[main]
strict_variables = true
```

This will not cause an error when a variable has been explicitly set to undef .

## Defining Attributes with a Hash
It is also possible to pass a hash of attribute names and values in to a resource definition. You do this with an attribute name of * (called a splat) with a value of the hash.
Here’s an example:
```
$resource_attributes = {
  ensure => present,
  owner => 'root',
  group => 'root',
  'mode' => '0644',
  'replace' => true,
}
file { '/etc/config/first.cfg':
  source => 'first.cfg',
  * => $resource_attributes,
}

```


The splat operator allows you to utilize a common set of values across many resources without repeating the same declaration. This is an essential strategy for don’t repeat yourself (DRY) development.

## Using Comparison Operators

Number comparisons operate much as you might expect.

String operators are a bit inconsistent. String equality comparisons are case insensitive, while substring matches are case sensitive:

```
coffee == 'coffee'	# bare word string is equivalent to quoted single word
'Coffee' == 'coffee' 	# string comparisons are case insensitive
'tea' !in 'coffee'	# you can't find tea in coffee
'Fee' !in 'coffee'	# substring matches are case sensitive
'fee' in 'coffee' 	# you can pay your daily barista fee

```

Array and hash comparisons match only with complete equality of both length and value. The in comparison looks for value matches in arrays, and key matches in hashes:
```
[1,2,5] != [1,2]	# array matching tests for identical arrays
5 in [1,2,5]		# # value found in array
 
You can also compare values to data types, like so:
$not_true =~ Boolean	 # true if true or false
$num_tokens =~ Integer   # true if an integer
$my_name !~ String       # true if no a string
```

## Evaluating Conditional Expressions

Four different ways to utilize the boolean results of a comparison:
* if / elsif / else statements
* unless / else statements
* case statements
* Selectors

As you’d expect, there’s always the basic conditional form I’m sure you know and love: 
```
if ($coffee != 'drunk') {
  notify { 'best-to-avoid': }
}
elsif ('scotch' == 'drunk') {
  notify { 'party-time': }
}
else {
  notify { 'party-time': }
}
```

There’s also unless to reverse comparisons for readability purposes:
```
unless $facts['kernel'] == Linux {
  notify { 'You are on an older machine.': }
}
else {
  notify { 'We got you covered.': }
}
```

The case operator can be used to do numerous evaluations, avoiding a long string of multiple elsif (s). You can test explicit values, match against another variable, use regular expressions, or evaluate the results of a function. The first successful match will execute the code within the block following a colon:
```
case $what_she_drank {
'wine': 		{ include state::california }
$stumptown: 		{ include state::portland }
/(scotch|whisky)/: 	{ include state::scotland }
is_tea( $drink ): 	{ include state::england }
default: 		{}
}
```

Always include a default: option when using case statements, even if the default does nothing, as shown in the preceding example.

Statements with selectors are similar to case statements, except they return a value instead of executing a block of code. This can be useful when you are defining variables. A selector looks like a normal assignment, but the value to be compared is followed by a question mark and a block of comparisons with fat commas identifying the matching values:
```
$native_of = $what_he_drinks ? {
  'wine' 		=> 'california',
  $stumptown 		=> 'portland',
  /(scotch|whisky)/ 	=> 'scotland',
  is_tea( $drink ) 	=> 'england',
  default 		=> 'unknown',
}
```

As a value must be returned in an assignment operation, a match is required. Always include a bare word default option with a value.

```
case $facts['osfamily'] {
  'redhat': { include yum }
  'debian', 'ubuntu': { include apt }
  'freebsd' and ($facts['os']['release']['major'] >= 9) { include pkgng }
  default: {}
}
```

Selectors are also useful for handling heterogenous environments:
```
$libdir = $facts['osfamily'] ? {
  /(?i-mx:centos|fedora|redhat)/ 	=> '/usr/libexec/mcollective',
  /(?i-mx:ubuntu|debian)/ 		=> '/usr/share/mcollective/plugins',
  /(?i-mx:freebsd)/ 			=> '/usr/local/share',
}
```

## Matching Regular Expressions
Puppet supports standard Ruby regular expressions, as defined in the Ruby Regexp docs. The match operator ( =~ ) requires a String value on the left, and a Regexp expression on the right:
```
$what_did_you_drink =~ /tea/
$what_did_you_drink !~ /coffee/
$what_did_you_drink !~ "^coffee$"
```
The value on the left must be a string. The value on the right can be a /Regexp/ definition within slashes, or a string value within double quotes. The use of a string allows variable interpolation to be performed prior to conversion into a Regexp .

You can use regular expressions in four places:
* Conditional statements: if and unless
* case statements
* Selectors
* Node definitions (deprecated)

Examples:
```
case $facts['hostname'] {
  /^web\d/: { include role::webserver }
  /^mail/ : { include role::mailserver }
  default : { include role::base }
}

$package_name = $facts['operatingsystem'] ? {
  /(?i-mx:centos|fedora|redhat)/ => 'mcollective',
  /(?i-mx:ubuntu|debian)/ => 'mcollective',
  /(?i-mx:freebsd)/ => 'sysutils/mcollective',
}
```

## Lambda Blocks

A lambda begins with one or more variable names between pipe operators | | . These name the variables that will contain the values passed into the block of code: 
```
| $firstvalue, $secondvalue | { 
   block of code that operates on these values.  
}
```

The lambda has its own variable scope. This means that the variables named between the pipes exist only within the block of code. You can name these variables any name
you want, as they will be filled by the values passed by the function into the lambda on each iteration. Other variables within the context of the lambda are also available, such as local variables or node facts.

The following example will output a list of disk partitions from the hash provided by
Facter. Within the loop, we refer to the hostname fact on each iteration. The device
name and a hash of values about each device are stored in the $name and $device
variables during each loop:
```
$ cat /vagrant/manifests/mountpoints.pp
each( $facts['partitions'] ) |$name, $device| {
  notice( "${facts['hostname']} has device ${name} with size ${device['size']}" )
}
$ puppet apply /vagrant/manifests/mountpoints.pp
Notice: Scope(Class[main]): puppetagent1 has device /dev/mapper/cl_puppet-root with size 8.50 GiB
Notice: Scope(Class[main]): puppetagent1 has device /dev/mapper/cl_puppet-swap with size 956.00 MiB
Notice: Scope(Class[main]): puppetagent1 has device /dev/vda1 with size 286.00 MiB
Notice: Scope(Class[main]): puppetagent1 has device /dev/vda2 with size 9.44 GiB
Notice: Compiled catalog for puppetagent1 in environment production in 0.07 seconds
Notice: Applied catalog in 0.02 seconds
```

## Looping Through Iterations
Puppet 4 contains new functions for iterating over sets of data. You can use iteration to evaluate many items within an array or hash of data using a single block of code.

There are five functions that iterate over a set of values and pass each one to a lambda for processing. The lambda will process each input and return a single response containing the processed values. Here are the five functions, what they do to provide input to the lambda, and what they expect the lambda to return as a response:
* each() acts on each entry in an array, or each key/value pair in a hash.
* filter() returns a subset of the array or hash that were matched by the lambda.
* map() returns a new array or hash from the results of the lambda.
* reduce() combines array or hash values using code in the lambda.
* slice() creates small chunks of an array or hash and passes it to the lambda.

The following examples show how these functions can be invoked. They can be invoked in traditional prefix style:
```
each( $facts['partitions'] ) |$name, $device| {
  notice( "${facts['hostname']} has device $name with size ${device['size']}" )
}
```
Or you can chain function calls to the values they operate on, which is a common usage within Ruby:
```
$facts['partitions'].each() |$name, $device| {
  notice( "${facts['hostname']} has device $name with size ${device['size']}" )
}
```

cat /vagrant/manifests/interfaces.pp
```
split( $facts['interfaces'], ',' ).each |$index, $interface| {
  notice( "Interface #${index} is ${interface}" )
tice: Scope(Class[main]): Interface #0 is eth0
Notice: Scope(Class[main]): Interface #1 is lo
Notice: Compiled catalog for puppetagent1 in environment production in 0.07 seconds
Notice: Applied catalog in 0.02 seconds


}

$ puppet apply /vagrant/manifests/interfaces.pp







# Best practices

* Use a comma after last attribute in a declaration.
** file { '/tmp/testfile.txt': ensure => present, mode => '0644', replace => true, content => 'holy cow!\n', } 
* Use single quotes for any value that does not contain a variable.  This protects against accidental interpolation of a variable. Use double quotes with strings containing variables.
* Always quote numbers that need to be represented intact, such as decimals with leading zeros.
* Use curly braces to delineate the beginning and end of a variable name within a string.
* Previously working unquoted strings can yield surprising results when functions or resource types with the same name are added to the catalog. Make code future-proof by quoting string values every time:
* Don’t surround standalone variables with curly braces or quotes.
* Refer explicitly to a fact using the $facts[] hash. This ensures access to unaltered values supplied by Facter, and tells the reader where the value came from.


