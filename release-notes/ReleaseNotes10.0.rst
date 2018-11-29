Release 0.10.0 (TBD, 2018)
==========================

Highlights
----------
 1. :ref:`Kylo's user interface <new_ui>` has been redone to make it easier to create and edit feeds.  Saving a feed is now separated from deploying to NiFi, allowing for users to save and come back to their feed prior to deploying.
 2. :ref:`New template repository <repository>` for exposing common templates and assisting in upgrading
 3. :ref:`Wrangler improvements <wrangler>`. Many new features have been added to the wrangler such as: quick clean feature, quick schema manipulation, column statistics view, and a number of bug fixes
 4. :ref:`New catalog support <catalog>`. Kylo allows you to create various catalog entries for browsing different datasources such as Amazon S3, Azure, JDBC, and others

Download Links
--------------
- Visit the :doc:`Downloads <../about/Downloads>` page for links.


Upgrade Instructions from v0.9.1
--------------------------------

1. Backup any Kylo plugins

  When Kylo is uninstalled it will backup configuration files, but not the `/plugin` jar files.
  If you have any custom plugins in either `kylo-services/plugin`  or `kylo-ui/plugin` then you will want to manually back them up to a different location.

2. Uninstall Kylo:

 .. code-block:: shell

   /opt/kylo/remove-kylo.sh

 ..

3. Install the new RPM:

 .. code-block:: shell

     rpm –ivh <RPM_FILE>

 ..

4. Restore previous application.properties files. If you have customized the the application.properties, copy the backup from the 0.9.1 install.


     4.1 Find the /bkup-config/TIMESTAMP/kylo-services/application.properties file

        - Kylo will backup the application.properties file to the following location, */opt/kylo/bkup-config/YYYY_MM_DD_HH_MM_millis/kylo-services/application.properties*, replacing the "YYYY_MM_DD_HH_MM_millis" with a valid time:

     4.2 Copy the backup file over to the /opt/kylo/kylo-services/conf folder

        .. code-block:: shell

          ### move the application.properties shipped with the .rpm to a backup file
          mv /opt/kylo/kylo-services/conf/application.properties /opt/kylo/kylo-services/conf/application.properties.0_10_0_template
          ### copy the backup properties  (Replace the YYYY_MM_DD_HH_MM_millis  with the valid timestamp)
          cp /opt/kylo/bkup-config/YYYY_MM_DD_HH_MM_millis/kylo-services/application.properties /opt/kylo/kylo-services/conf

        ..

     4.3 If you copied the backup version of application.properties in step 4.2 you will need to make a couple of other changes based on the 0.10.0 version of the properties file

        .. code-block:: shell

          vi /opt/kylo/kylo-services/conf/application.properties

        ..

     4.4 Repeat previous copy step for other relevant backup files to the /opt/kylo/kylo-services/conf folder. Some examples of files:

        - spark.properties
        - ambari.properties
        - elasticsearch-rest.properties
        - log4j.properties
        - sla.email.properties

        **NOTE:**  Be careful not to overwrite configuration files used exclusively by Kylo


     4.5 Copy the /bkup-config/TIMESTAMP/kylo-ui/application.properties file to `/opt/kylo/kylo-ui/conf`

     4.6 Ensure the property ``security.jwt.key`` in both kylo-services and kylo-ui application.properties file match.  They property below needs to match in both of these files:

        - */opt/kylo/kylo-ui/conf/application.properties*
        - */opt/kylo/kylo-services/conf/application.properties*

          .. code-block:: properties

            security.jwt.key=

          ..

    4.7 (If using Elasticsearch for search) Create/Update Kylo Indexes

        Execute a script to create/update kylo indexes. If these already exist, Elasticsearch will report an ``index_already_exists_exception``. It is safe to ignore this and continue.
        Change the host and port if necessary.

            .. code-block:: shell

                /opt/kylo/bin/create-kylo-indexes-es.sh localhost 9200 1 1

            ..


5. Update the NiFi nars.

   Stop NiFi

   .. code-block:: shell

      service nifi stop

   ..

   Run the following shell script to copy over the new NiFi nars/jars to get new changes to NiFi processors and services.

   .. code-block:: shell

      /opt/kylo/setup/nifi/update-nars-jars.sh <NIFI_HOME> <KYLO_SETUP_FOLDER> <NIFI_LINUX_USER> <NIFI_LINUX_GROUP>

      Example:  /opt/kylo/setup/nifi/update-nars-jars.sh /opt/nifi /opt/kylo/setup nifi users

   ..
   
   Setup the shared Kylo encryption key:
   
      1. Copy Kylo's encryption key file (ex: ``/opt/kylo/encrypt.key``) to the NiFi extention config directory ``/opt/nifi/ext-config``
      
      2. Change the ownership of that file to the "nifi" user and ensure only nifi can read it

   .. code-block:: shell

      chown nifi /opt/nifi/ext-config/encrypt.key
      chmod 400 /opt/nifi/ext-config/encrypt.key

   ..
   
      3. Edit the ``/opt/nifi/current/bin/nifi-env.sh`` file and add the ENCRYPT_KEY variable with the key value

   .. code-block:: shell

      export ENCRYPT_KEY="$(< /opt/nifi/ext-config/encrypt.key)"
      
   ..

   Start NiFi

   .. code-block:: shell

      service nifi start

   ..


6. :ref:`Install XML support <install-xml-support>` if not using Hortonworks.

7. Start Kylo to begin the upgrade

 .. code-block:: shell

   kylo-service start

 ..
 .. note:: NiFi must be started and available during the Kylo upgrade process.

8. The Hive data source is no longer accessible to all users by default. To grant permissions to Hive go to the Catalog page and click the pencil icon to the left of the Hive data source. This page will provide options for granting access to Hive or granting permissions to edit the data source details.

Mandatory Template Updates
--------------------------
The following templates need to get updated.

  - XML Ingest
  - Data Transformation

To update these templates use the new :doc:`Repository <../how-to-guides/KyloTemplatesDocs>` feature within Kylo to import the latest templates.

Highlight Details
-----------------

.. _new_ui:

   - New User Interface

       - Kylo now has a new user interface for creating and editing feeds.

           |new_ui_image01|

       - Editing feeds is separate from deploying to NiFi.  This allows you to edit and save your feed state and when ready deploy it.

       - Centralized feed information. The feed activity view of the running feed jobs is now integrated with the feed setup.

           |new_ui_image02|

.. _catalog:

    - Kylo allows you to create and browse various catalog sources. Kylo ships with the following datasource connectors:  Amazon S3, Azure, HDFS, Hive, JDBC, Local Files

         |catalog_image01|

    - During feed creation and data wrangling you can browse the catalog to preview and select specific sources to work with:

       |catalog_image02|

    - *Note:* Kylo Datasources  have been upgraded to a new Catalog feature.  All legacy JDBC and Hive datasources will be automatically converted to catalog data source entries.

.. _wrangler:

     The Wrangler has been upgrade with many new features

      |wrangler_image01|

     - New quick clean feature allows you to modify the entire dataset

      |wrangler_image02|

     - New schema view allows you to rename, delete, and move columns

       |wrangler_image03|

     -  New column profile view shows graphical stats about each column

        |wrangler_image04|


.. _repository:

   Kylo now has customizable repository locations to store feed and template exports.  The repository is an easy way to browse for new feeds/templates and import directly into Kylo.
   Kylo creates a default repository exposing the sample templates.

     |repository_image01|




.. |JIRA_Issues_Link| raw:: html

   <a href="https://kylo-io.atlassian.net/issues/?jql=project%20%3D%20KYLO%20AND%20status%20%3D%20Done%20AND%20fixVersion%20%3D%200.10.0%20ORDER%20BY%20summary%20ASC%2C%20lastViewed%20DESC" target="_blank">Jira Issues</a>


.. |new_ui_image01| image:: ../media/release-notes/release-0.10.0/new_ui_image01.png
   :width: 2632px
   :height: 1348px
   :scale: 15%
.. |new_ui_image02| image:: ../media/release-notes/release-0.10.0/new_ui_image02.png
   :width: 2612px
   :height: 652px
   :scale: 15%
.. |catalog_image01| image:: ../media/release-notes/release-0.10.0/catalog_image01.png
    :width: 1544px
    :height: 392px
    :scale: 15%
.. |catalog_image02| image:: ../media/release-notes/release-0.10.0/catalog_image02.png
   :width: 3312px
   :height: 444px
   :scale: 15%
.. |repository_image01| image:: ../media/release-notes/release-0.10.0/repository_image01.png
   :width: 2766px
   :height: 1500px
   :scale: 15%
.. |wrangler_image01| image:: ../media/release-notes/release-0.10.0/wrangler_image01.png
   :width: 942px
   :height: 280px
   :scale: 15%
.. |wrangler_image02| image:: ../media/release-notes/release-0.10.0/wrangler_image02.png
   :width: 2562px
   :height: 358px
   :scale: 15%
.. |wrangler_image03| image:: ../media/release-notes/release-0.10.0/wrangler_image03.png
   :width: 2562px
   :height: 402px
   :scale: 15%
.. |wrangler_image04| image:: ../media/release-notes/release-0.10.0/wrangler_image04.png
   :width: 2546px
   :height: 416px
   :scale: 15%