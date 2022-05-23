gvm-cli
=======

Create a user for ``gvm-cli``
-----------------------------

* Login Name: gvmcli
* Comment: Used by gvm-cli
* Authentication: abcde12345
* Roles: User
* Groups:
* Host Access:
    * Allow all and deny:
* Interface Access:
    * Allow all and deny:

Create config file
------------------

Create ``gvm-tools.conf`` file:

.. code-block:: text

    [main]
    timeout = 60

    [gmp]
    username=gvmcli
    password=abcde12345

    [tls]
    port=8000

Allow the ``gvm-tools.conf`` file to be mounted by container:

.. code-block:: bash

    chcon -v -t container_file_t ./gvm-tools.conf

Example how to scan
-------------------

Use the following command template:

.. code-block:: bash

    podman run -it --rm -v ./gvm-tools.conf:/home/gvmuser/.config/gvm-tools.conf:ro extra2000/greenbone/gvm-tools gvm-cli tls --host [GVMD_SERVER] --xml "[XML]" | xmllint --format -

where the ``[XML]`` value should be replaced according the following Subsections.

Get ``port_list`` UUID
~~~~~~~~~~~~~~~~~~~~~~

Find out UUID port name for ``All TCP and Nmap top 100 UDP``:

.. code-block:: xml

    <get_port_lists/>

Assuming the UUID for the port is ``730ef368-57e2-11e1-a90f-406186ea4fc5``.

Get ``OpenVAS Default`` scanner UUID
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: xml

    <get_scanners/>

Assuming the UUID for the scanner is ``08b69003-5fc2-4037-a479-93b440211c73``.

Get ``Full and fast`` scan config UUID
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: xml

    <get_configs/>

Assuming the UUID for the scan config is ``daba56c8-73ec-11df-a475-002264764cea``.

Create target
~~~~~~~~~~~~~

Create target for IP ``192.168.1.100``:

.. code-block:: xml

    <create_target>
        <name>Suspect Host</name>
        <comment>Example target host</comment>
        <hosts>192.168.1.100</hosts>
        <port_list id=\"730ef368-57e2-11e1-a90f-406186ea4fc5\"></port_list>
        <alive_tests>ICMP, TCP-ACK Service &#38; ARP Ping</alive_tests>
    </create_target>

Assuming the UUID for the target is ``afb8452b-4832-4a23-9d28-7b87b82677b5``.

Create task
~~~~~~~~~~~

Create task:

.. code-block:: xml

    <create_task>
        <name>My Server Scan</name>
        <comment>Example server scan</comment>
        <target id=\"afb8452b-4832-4a23-9d28-7b87b82677b5\"></target>
        <scanner id=\"08b69003-5fc2-4037-a479-93b440211c73\"></scanner>
        <config id=\"daba56c8-73ec-11df-a475-002264764cea\"></config>
    </create_task>

Assuming the UUID for this task is ``34527bc5-a964-41a2-89e1-7bd3afd88b39``. Use ``<get_tasks/>`` to find out this task's UUID.

Start task
~~~~~~~~~~

Start task:

.. code-block:: xml

    <start_task task_id=\"34527bc5-a964-41a2-89e1-7bd3afd88b39\"></start_task>

Get task status
~~~~~~~~~~~~~~~

Get task status:

.. code-block:: xml

    <get_tasks task_id=\"34527bc5-a964-41a2-89e1-7bd3afd88b39\"></get_tasks>

Example response when the current task is ``running`` with progress at 95% completed.

.. code-block:: xml

    <?xml version="1.0"?>
    <get_tasks_response status="200" status_text="OK">
      ...
      <task id="34527bc5-a964-41a2-89e1-7bd3afd88b39">
        ...
        <status>Running</status>
        <progress>95</progress>
        ...
      </task>
      ...
    </get_tasks_response>

Get report
~~~~~~~~~~

Get the last report UUID from the task:

.. code-block:: xml

  <get_tasks task_id=\"34527bc5-a964-41a2-89e1-7bd3afd88b39\"></get_tasks>


Example response:

.. code-block:: xml

    <?xml version="1.0"?>
    <get_tasks_response status="200" status_text="OK">
      ...
      <task id="34527bc5-a964-41a2-89e1-7bd3afd88b39">
        ...
        <last_report>
          <report id="342a7c1d-fda9-433f-86a3-1be1db034f72">
            <timestamp>2022-05-20T12:19:12Z</timestamp>
            <scan_start>2022-05-20T12:23:37Z</scan_start>
            <scan_end>2022-05-20T12:30:11Z</scan_end>
            <result_count>
              <hole>0</hole>
              <info>0</info>
              <log>3</log>
              <warning>0</warning>
              <false_positive>0</false_positive>
            </result_count>
            <severity>0.0</severity>
          </report>
        </last_report>
        ...
      </task>
      ...
    </get_tasks_response>

Get the detailed report for the last report UUID:

.. code-block:: xml

    <get_reports report_id=\"342a7c1d-fda9-433f-86a3-1be1db034f72\" details=\"1\"></get_reports>

Example response:

.. code-block:: xml

    <?xml version="1.0"?>
    <get_reports_response status="200" status_text="OK">
      <report id="342a7c1d-fda9-433f-86a3-1be1db034f72" format_id="" extension="" content_type="application/xml">
        ...
        <name>2022-05-20T12:23:37Z</name>
        ...
        <task id="34527bc5-a964-41a2-89e1-7bd3afd88b39">
          <name>My Server Scan</name>
        </task>
        <report id="342a7c1d-fda9-433f-86a3-1be1db034f72">
          ...
          <hosts>
            <count>1</count>
          </hosts>
          <closed_cves>
            <count>0</count>
          </closed_cves>
          <vulns>
            <count>3</count>
          </vulns>
          <os>
            <count>0</count>
          </os>
          <apps>
            <count>0</count>
          </apps>
          <ssl_certs>
            <count>0</count>
          </ssl_certs>
          ...
          <results start="1" max="10">
            <result id="d8ee2638-d5c5-4de4-8546-ad2bb30d80ba">
              <name>Hostname Determination Reporting</name>
              ...
            </result>
            <result id="734df0a5-aa0a-4588-a70c-9f09e16bd6f4">
              <name>OS Detection Consolidation and Reporting</name>
              ...
            </result>
          </results>
          ...
        </report>
      </report>
      ...
    </get_reports_response>
