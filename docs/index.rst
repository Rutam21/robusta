Welcome to Robusta
================================

Robusta is an open source platform for Kubernetes troubleshooting. It sits on top of your monitoring stack
(Prometheus, Elasticsearch, etc) and tells you why alerts occurred and how to fix them.

Robusta has three main parts, all open source:

1. An automations engine for Kubernetes
2. Builtin automations to enrich and fix common alerts
3. Manual troubleshooting tools for everything else

There are additional *optional* components:

4. An all-in-one bundle with Robusta, the Prometheus Operator, and default Kubernetes alerts [#f1]_
5. `A web UI to see all alerts, changes, and events in your cluster. <http://home.robusta.dev/ui?from=docs>`_ [#f2]_

Example Use Cases
~~~~~~~~~~~~~~~~~~

.. tab-set::

    .. tab-item:: Crashing pods

        .. admonition:: Monitor crashing pods and send their logs to Slack

            .. image:: /images/crash-report2.png
                :width: 700
                :align: center

    .. tab-item:: Event Correlation

        .. admonition:: Show application updates in Grafana to correlate them with error spikes

            .. image:: /images/grafana-deployment-enrichment.png
              :width: 400
              :align: center

    .. tab-item:: Remediate alerts

        .. admonition:: Temporarily increase the HPA maximum so you can go back to sleep

            .. image:: /images/alert_on_hpa_reached_limit1.png
                :width: 600
                :align: center

    .. tab-item:: Debug Pods

        .. admonition:: Attach the VSCode debugger to a runnning Python pod without tearing your hair out

            .. image:: /images/python-debugger.png
              :width: 600
              :align: center

            .. code-block:: bash

                 robusta playbooks trigger python_debugger name=podname namespace=default

            See :ref:`Python debugger` for more details

How it works
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Robusta automates everything that happens **after** you deploy your application.

It is somewhat like Zapier/IFTTT for devops, with an emphasis on prebuilt automations and not just "build your own".

For example, the following automation sends logs to Slack when an alert fires for crashing pods:

.. code-block:: yaml

    triggers:
      - on_prometheus_alert:
          alert_name: KubePodCrashLooping
    actions:
      - logs_enricher: {}
    sinks:
      - slack

Every automation has three parts:

.. grid:: 3

    .. grid-item-card:: Triggers
        :class-card: sd-bg-light sd-bg-text-light
        :link: catalog/triggers/index
        :link-type: doc

        When to run
        (on alerts, logs, changes, etc)

    .. grid-item-card::  Actions
        :class-card: sd-bg-light sd-bg-text-light
        :link: catalog/actions/index
        :link-type: doc

        What to do
        (over 50 builtin actions)

    .. grid-item-card::  Sinks
        :class-card: sd-bg-light sd-bg-text-light
        :link: catalog/sinks/index
        :link-type: doc

        Where to send the result
        (Slack, etc)

Automations run via webhook so if they fail it wont bring down your environment.

Writing your own automations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Many automations are included, but you can also write your own in Python.

.. dropdown:: View example action (Python)
    :color: light

    .. code-block:: python

        # this runs on Prometheus alerts you specify in the YAML
        @action
        def my_enricher(event: PrometheusKubernetesAlert):
            # we have full access to the pod on which the alert fired
            pod = event.get_pod()
            pod_name = pod.metadata.name
            pod_logs = pod.get_logs()
            pod_processes = pod.exec("ps aux")

            # this is how you send data to slack or other destinations
            event.add_enrichment([
                MarkdownBlock("*Oh no!* An alert occurred on " + pod_name)
                FileBlock("crashing-pod.log", pod_logs)
            ])

Next Steps
~~~~~~~~~~~~

:ref:`Ready to install Robusta? Get started! <installation>`

`Star us on Github to receive updates. <https://github.com/robusta-dev/robusta/>`_

.. rubric:: Footnotes

.. [#f1] These alerts should cause no noise on a healthy cluster. If they're noisy in your env, let us know and we'll fix it.

.. [#f2] This is the only component that isn't open source and it's completely optional. An on-prem version is in development too.

.. toctree::
   :hidden:

   Project Home <https://home.robusta.dev/?from=docs>

.. toctree::
   :maxdepth: 4
   :caption: About Robusta
   :hidden:

   Overview <self>
   architecture
   coverage
   community

.. toctree::
   :maxdepth: 4
   :caption: Tutorials
   :hidden:

   installation
   tutorials/configuring-automations
   tutorials/prometheus-enrichment
   tutorials/python-troubleshooting
   tutorials/java-troubleshooting

..
   tutorials/alert-correlation

.. toctree::
   :maxdepth: 4
   :caption: Reference
   :hidden:

   upgrade
   user-guide/configuration
   catalog/triggers/index
   catalog/actions/index
   catalog/sinks/index
   user-guide/robusta-cli
   user-guide/trigger-action-binding

.. toctree::
   :maxdepth: 4
   :caption: Developer Guide
   :hidden:

   developer-guide/actions/index
   developer-guide/platform/index
