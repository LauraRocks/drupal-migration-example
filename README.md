# Drupal 8 Migration Example

Usually when a huge site makes the (wise) decision to migrate to Drupal, one of the biggest concerns of the site owners is _How to migrate the old site's data into the new Drupal site_. The existing old site might or might not be a Drupal site, but given that the new site is on Drupal, we can make use of the cool _migrate_ module to import data from a variety of data sources including but not limited to XML, JSON, CSV and SQL databases.

This project is an example module showing how to go about importing basic data from a CSV data source though things would work pretty similarly for other types of data sources. Apart from a basic data import, I have also included certain other important things which a migration might involve, for example, import of 2 different types of entities and the relation between them.

Though being written to serve as a simple example for demonstrating the basics of Drupal 8 migrations, in this project we cover:

* Import of basic content as Drupal _node_ entities or _taxonomy term_ entities.
* Certain ways of basic data manipulation during data the import process.
* Import of basic relationships between two entities, eg, articles and tags.
* Import of images / files as Drupal _file_ entities and relating them to the relevant content.

# The problem

As per project requirements, we wish to import certain data for an educational and cultural insitution.

* **Academic programs:** We have a CSV file containing details related to academic programs. We are required to create _nodes_ of type _program_ with the data.
* **Tags:** We have a CSV file containing details related to tags for these academic programs. We are required to import these as _terms_ of the _vocabulary_ named _tags_.
* **Images:** We have images for each academic program. The base name of the images are mentioned in the CSV file for academic programs. To make things easy, we have only one image per program.

# [c11n_migrate.info.yml](c11n_migrate.info.yml)

I usually prefer to name project-specific custom modules with a prefix of _c11n_ (being the numeronym for customization). This way, have a naming convention for custom modules and I can copy any custom module to another site without worrying about having to change prefixes. The fact to be noted is that we have a _.info.yml_ file instead of the _.info_ file we were used to in Drupal 7.

Nothing fancy about the module file as such. It includes a basic project definition with certain dependencies on other modules. Though the _migrate_ module is in Drupal 8 core, we need most of these dependencies to enable / enhance migrations on the site:
* **migrate**: Without the migrate module, we cannot migrate!
* **migrate_plus**: Improves the core _migrate_ module by adding certain functionality like migration groups. Apart from that this module includes an example module which I referred to on various occations while writing this module.
* **migrate_tools**: General-purpose drush commands and basic UI for managing migrations.
* **migrate_source_csv**: The core _migrate_ module provides a basic framework for migrations, which does not include support for specific data sources. This module makes the _migrate_ module work with CSV data sources. There are other modules which provide support for other data sources like JSON, XML, etc.
* **node**: We will be importing _academic programs_ as nodes. Thus, we need the _node_ module.
* **taxonomy**: We will be importing _tags_ as taxonomy terms. Thus, we need the _taxonomy_ module.

# [c11n_migrate.module](c11n_migrate.module)

In Drupal 8, unlike Drupal 7, a module only provides a _.module_ file if required. However, in our case, we use a custom callback for one of the migrations, so I had to create the _.module_ file and write the custom callback there.

# Data source

Ref: [import/README.md](import/README.md)

To import the data, we would obviously require a data source. For the sake of this example, I have provided the source data in a _import_ directory inside the module. In short, you have to copy that directory to the _files_ directory of your site. Kindly read the README.md file inside that directory for further instructions.

# Migration group

Ref: [migrate_plus.migration_group.c11n.yml](config/install/migrate_plus.migration_group.c11n.yml)

Like we used to implement _hook_migrate_api()_ in Drupal 7 to declare the API version, migration groups, individual migrations and more, in Drupal 8, we do something similar. Instead of implementing a hook, we create a migration group declaration inside the _config/install_ directory of our module. The file must be named something like _migrate_plus.migration_group.NAME.yml_ where _NAME_ is the machine name for the migration group.

In this example, we define a migration group _c11n_ to provide general information like:

* **id:** A unique ID for the migration. This is usually the _NAME_ part of the migration group declaration file name as discussed above.
* **label:** A human-friendly name of the migration group as it would in the UI.
* **description:** A brief description about the migration group.
* **source_type:** This would appear in the UI to provide a general hint as to where the data for this migration comes from.
* **dependencies:** Though this might sound a bit strange for Drupal 7 users (like me), this segment is used to define modules on which the migration depends. When one of these required modules are missing / removed, the migration group is automatically removed.

We can execute all migrations in a given group with the command `drush migrate-import --group=GROUP`.

# Migration definition: Meta-data

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

Now that we have a module to put our migration scripts in and a migration group for grouping them together, it's time we write a basic migration! To get started, we import basic data about academic programs, ignoring complex stuff such as tags, files, etc. In Drupal 7 we used to do this in a file containing a PHP class which used to extend the _Migration_ class provided by the _migrate_ module. In Drupal 8, like many other things, we do this in a YML file.

In migration declaration file, we declare some meta-data about the migration:

* **id:** A unique identifier for the migration. In this example, I allocated the ID _program_data_, hence, the migration declaration file has been named migrate_plus.migration._program_data_.yml. We can execute specific migrations with the command `drush migrate-import --idlist=ID1,ID2,ID3`.
* **label:** A human-friendly name of the migration as it would in the UI.
* **migration_group:** This puts the migration into the migration group _c11n_ we created above. We can execute all migrations in a given group with the command `drush migrate-import --group=GROUP`.
* **migration_tags:** Here we provide multiple tags for the migration and just like groups, we can execute all migrations with the same tag using the command `drush migrate-import --tag=TAG`
* **dependencies:** Just like in case of migration groups, this segment is used to define modules on which the migration depends. When one of these required modules are missing / removed, the migration is automatically removed.
* **migration_dependencies:** This element is used to mention IDs of other migrations which must be run before this migration. For example, if we are importing articles and their authors, we need to import author data first so that we can refer to the author's ID while importing the articles. Note that we can leave this undefined for now as we do not have any other migrations defined. I defined this section only after I finished writing the migrations for tags, files, etc.

# Migration definition: Source

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

Once done with the meta-data, we define the source of the migration data with the _source_ element in the YAML.

* **plugin:** The plugin responsible for reading the source data. In our case we use the _migrate_source_csv_ module which provides the source plugin _csv_.
* **path:** Path to the data source file - in this case, the [program.data.csv](import/program/program.data.csv) file.
* **header_row_count:** This is a plugin-specific parameter which allows us to skip a number of rows from the top of the CSV. I found this parameter in the plugin class file, but I'm sure it must also be mentioned in the documentation for the _migrate_source_csv_ module.
* **keys:** This parameter defines a number of columns in the source data which form a unique key in the source data. Luckily in our case, the program.data.csv provides a unique ID column so things get easy for us in this migration. This unique key will be used by the migrate module to relate records from the source with the records created in our Drupal site. With this relation, the migrate module can interpret changes in the source data and update the relevant data on the site. To execute an update, we use the parameter `--update` while our `drush migrate-import` command.
* **fields:** This parameter defines provides a description for the various columns available in the CSV data source. These descriptions just appear in the UI and explain purpose behind each column of the CSV.

# Migration definition: Destination

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

Similarly, we need to tell the migrate module how we want it to use the source data. We do this with the _destination_ element in the YAML.

* **plugin:** Just like source data is handled by separate plugins, we have _destination_ plugins to handle the output of the migrations. In this case, we want Drupal to create _node_ entities with the academic program data, so we use the _node_ plugin.
* **default_bundle:** Here, we define the type of nodes we wish to obtain using the migration. Though we can override the bundle for individual imports, this parameter provides a default bundle for entities created by this migration. We will be creating only _program_ nodes, so we mention that here.

# Migration definition: Mapping and processing

Ref: [migrate_plus.migration.program_data.yml](config/install/migrate_plus.migration.program_data.yml)

If you ever wrote a migration in an earlier version of Drupal, you might already know that migration are usually not as simple as copying data from one column of a CSV file to a given property of the relevant entity. We need to process certain columns and eliminate certain columns and much more. In Drupal 8, we define these processes using a _process_ element in the migration declaration. This is where we put our YAML skills to real use.

* **title:** An easy property to start with, we just assign the _Title_ column of the CSV as the _title_ property of the node.
* **sticky:** Though Drupal can apply the default value for this property if we skip it, I wanted to demonstrate how to specify a default value for a property. We use the _default_value_ plugin with the _default_value_ parameter to make the imports non-sticky with _sticky = 0_.
* **uid:** Similarly we specify default owner for the article as _root_ with _uid = 1_.
* **body:** The _body_ is a filtered long text field and has various sub-properties we can set. So, we copy the _Body_ column from the CSV file to the _body/value_ property (instead of assigning it to just _body_). In the next line, we specify the _body/format_ property as _restricted_html_. Similary, one can also add a custom summary for the nodes using the _body/summary_ property. However, we should keep in mind that while defining these sub-properties, we need to wrap the property name in quotes because we have a `/` in the property name.
* **field_program_level:** With this property we get to try out the useful _static_map_ plugin. Here, the source data uses the values _graduate/undergraduate_ whereas the destination field only accepts _gr/ug_. In Drupal 7, we would have written a few lines of code in a _ProgramDataMigration::prepareRow()_ method, but in Drupal 8, we just write some more YAML. Here, we have the plugin specifications as usual, but we have small dashes with which we are actually defining an array of plugins or a _plugin pipeline_. With the first plugin, we call the function _strtolower_ (with `callback: strtolower`) on the _Level_ property (with `source: Level`). Once the old value is in lower case, we pass it through the _static_map_ (with `plugin: static_map`) and define a map of new values which should be used instead of old values (with the `map` element). Done!

With the parameters above, we can write basic migrations with basic data-manipulation. If you wish to see another basic migration, you can take a look at [migrate_plus.migration.program_tags.yml](config/install/migrate_plus.migration.program_tags.yml). In the sections below, I would explain how to do some complex tasks like importing taxonomy terms and their relations with nodes and uploading files / images and associating them with their relevant nodes.
