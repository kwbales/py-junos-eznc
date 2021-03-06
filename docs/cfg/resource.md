## Resource Item

Each resource item has a unique _name_ and a collection of _properties_.  

You can access the following item variables:

  * name - the name of the resource
  * properties - a list of properties you can read/write
  * manages - a list of other resources managed by this resource 
  * exists - True/False if resource exists in Junos config
  * active - True/False if resource is active in Junos config
  * xml - The actual Junos XML associated with this resource (typically for debug)

````python
>>> from jnpr.junos.cfg.user import User
>>> me = User(dev,'jeremy')

# What is this resource name?
>>> me.name
'jeremy'

# what properties can I read/write?
>>> me.properties
['_exists', '_active', 'uid', 'fullname', 'userclass', 'password', '$password', '$sshkeys']

# is this resource active configuration?
>>> me.active
True
````

### Reading and Writing Resource Properites

Each resource item has separate read and write caches (Python dictionaries).  When the resource is read from the Junos device the read-cache is loaded.  When you modify the resource properites you are storing values into the write-cache, but not directly to the device.  

For example, in the debug output of the _User_ resource, the "HAS" dictionary is the read-cache, and teh "SHOULD" dictionary is the write-cache.  The following shows that I haven't cahnged the write-cache.
````python
>>> me
NAME: User: jeremy
HAS: {'$password': '$1$n/RPB3fZ$RGPy8hymoTa8G5oGiJMdr.',
 '$sshkeys': [('ssh-rsa',
               'ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQE<snip>')],
 '_active': True,
 '_exists': True,
 'fullname': 'Jeremy L. Schulman',
 'uid': 2001,
 'userclass': 'super-user'}
SHOULD:{}
````

Any property that begins with a dollar-sign($) is read-only.

Reading can be done either as an attribute or as a dictionary lookup.  If the property is read-only, you must use the dictionary method.
````python
>>> me.fullname
'Jeremy L. Schulman'

>>> me['fullname']
'Jeremy L. Schulman'
````

Writing can be done in one of three ways: (1) as attribute, (2) as dictionary or (3) as call paramter.  The first two methods only update the write-cache.  To write the changes back to the device you must explicitly call _Resource.write()_, as shown:
````python
>>> me.fullname = "Jeremy Schulman"
>>> me.write()
True
>>> me['fullname'] = "Jeremy 'the dude' Schulman"
>>> me.write()
True
````

When you use the call method, it will make the chacen to the device directly:
````python
>>> me(fullname="Jeremy Schulman")
True
````
Another benefit of the call method is you can make multiple changes at once. 
````python
>>> me(fullname="Jeremy 'the dude' Schulman", uid=3000)
True
>>> me
NAME: User: jeremy
HAS: {'$password': '$1$n/RPB3fZ$RGPy8hymoTa8G5oGiJMdr.',
 '$sshkeys': [('ssh-rsa',
               'ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAm2<snip>')],
 '_active': True,
 '_exists': True,
 'fullname': "Jeremy 'the dude' Schulman",
 'uid': 3000,
 'userclass': 'super-user'}
SHOULD:{}
````

The changes are written into the **Junos candidate configuration**, so you still need to issue a "commit" before these changes become active.  The recommended approach for this is to use the _Config_ utils.  For more on Utilities, refer to the [documentation](../utils/README.md).

### Resource Item Routines

Additionally each item provides a number of routines for configuration management:

  * read() - reads the resource config from the Junos device
  * write() - writes the modified resource properties to the Junos device
  * delete() - removes the resource from the Junos config
  * rename() - renames the resource in the Junos config
  * activate() - activates the resource in the Junos config
  * deactivate() - deactivates the resource in the Junos config
  * reorder() - changes the ordering of the resource in the Junos config

For example, to deactivate a user account:
````python
>>> me.deactivate()
True
>>> me.active
False
````
Note that the act of deactivating a resource performs the change to the Junos device; i.e. you do **not** need to issue a separate _write()_.  This is true for all of these routines.

