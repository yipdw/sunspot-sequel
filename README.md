sunspot-sequel
==============

This gem is a Sunspot adapter for Sequel.   It provides the necessary glue code
to use Sequel models with Sunspot.

It also provides the following on Sequel models:

* instance-level `index` and `index!` methods
* class-level `searchable` and `search` methods

These methods behave similarly to the ActiveRecord extensions provided by
`sunspot_rails`.

Usage
=====

First, configure Sunspot in the usual way. (See
<https://github.com/outoftime/sunspot/wiki/Installation-and-configuration>.)
Then, in the models you want to use with Sunspot:

    class Widget < Sequel::Model
      plugin :sunspot

      searchable do
        text :name
        ...
      end
    end

To index an instance of a model:
 
    widgets.each { |w| w.index }
    Sunspot.commit

Or, if you want to index and commit in one line of code:

    widget.index!

To search for objects of a certain type:

    Widget.search do
      with :name, name
      ...
    end

Of course, all of Sunspot's usual indexing and searching methods also work.


Limitations
===========

This plugin has some limitations that you should know about before deciding if
it's a good fit for your application.


Thread safety (or lack thereof)
-------------------------------

Indexing and searching with this plugin isn't thread-safe.

To elaborate: This plugin delegates index and search calls to what Sunspot calls
its singleton session, which is marked as thread-unsafe.

Thread safety is desirable, but achieving that will require changes to Sunspot.
(It's probably possible to use Sunspot in a thread-safe manner via manual management of
sessions, but issues like this are something that are better handled outside of
an adapter.)


Models must be persisted before indexing
----------------------------------------

This adapter does not prevent you from invoking #index or #index! on
unpersisted models.  (I'm not yet quite sure if it should contain such a check:
Sunspot's official ActiveRecord adapter doesn't do such a thing.)

This may not actually be a limitation -- most people don't need to index
records that haven't yet been saved somewhere -- but it's worth mentioning anyway.



Models to be indexed cannot use composite primary keys
------------------------------------------------------

Composite primary keys are not yet compatible with Sunspot's ID generation scheme.


Search results from `.search` are not Sequel datasets
-----------------------------------------------------

The `.search` class method on models that is installed by this plugin does not
return Sequel datasets, so you can't use Sequel's dataset filtering methods on
those result sets.  This is a consequence of the way Sunspot currently reifies
Solr results.


Copyright
=========

Copyright (c) 2010 David Yip.  Licensed under the MIT license; see the LICENSE
file for more information.
